+++
title = "building pb-scale ml training data — a systems deep dive"
date = 2026-05-18
draft = true

[taxonomies]
tags = ["data", "ml-platform"]
+++

Building ML training datasets at petabyte scale is one of those problems that sounds like an engineering execution challenge until you are actually doing it. Then it becomes clear that it is a systems design problem, a data modeling problem, and an organizational problem layered on top of each other. The fact that your data lives across six different systems with incompatible schemas, different update cadences, and different ownership is not a special case. It is the default condition.

I want to walk through how I think about this end to end — the full pipeline architecture, the tool choices at each stage, and where the open-source ecosystem converges with or diverges from how Google approaches the same problem.

<!-- more -->

### the starting condition

Imagine a large-scale population genomics study. The entity you want to train on is a sequenced individual — a genomic sample with associated biological and experimental metadata. The information that defines that entity does not live in one place. It lives across:

- a sequencing platform database: raw read quality metrics, coverage depth, sequencing batch, instrument metadata
- a variant calling store: called SNPs and indels per sample, zygosity, quality scores, population frequency annotations
- a phenotype annotation system: observed traits, computed scores, annotated outcomes
- a gene expression database: RNA-seq counts per gene per sample, normalized expression matrices
- a population reference panel: ancestry estimates, relatedness matrices, principal component projections
- a sample metadata system: collection site, cohort membership, consent tier, QC flags

None of these systems were designed with ML training in mind. Each is optimized for its own access patterns. Your job is to produce, from all of this, a wide flat table: one row per genomic sample, with hundreds of features, ready for a model to read efficiently at scale.

The full pipeline to get there looks like this:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SOURCE SYSTEMS                                        │
│                                                                               │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │ Sequencing │  │  Variant   │  │ Phenotype  │  │ Expression │  ...       │
│  │  Platform  │  │   Store    │  │ Annotation │  │    DB      │            │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘            │
└────────┼───────────────┼───────────────┼───────────────┼────────────────────┘
         │               │               │               │
         ▼               ▼               ▼               ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  STAGE 1 · EXTRACTION & NORMALIZATION  (Map)                                 │
│  • read each source independently, in parallel                               │
│  • normalize entity keys (sample IDs across systems are not the same)        │
│  • standardize schemas to canonical Avro/proto/Arrow                         │
│  • emit (sample_id, source_shard) records, partitioned by sample_id hash    │
│                                                                               │
│  Google: Flume readers + proto schemas                                        │
│  Open:   Spark/Beam connectors, PyArrow for in-process reads, Avro/Parquet  │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │  Shuffle by sample_id
                                   ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  STAGE 2 · ENTITY JOINING  (Shuffle + Reduce)                                │
│  • co-group all source shards by sample_id                                   │
│  • resolve cross-system key conflicts (sample aliasing, batch corrections)   │
│  • flatten one-to-many (one sample → thousands of variants)                  │
│  • apply QC filters: remove samples that fail coverage or consent checks     │
│  • emit one intermediate record per sample                                   │
│                                                                               │
│  Google: Flume CoGroupByKey                                                  │
│  Open:   Spark sortMergeJoin or Beam CoGroupByKey, PyArrow join kernels      │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │  Map (embarrassingly parallel)
                                   ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  STAGE 3 · FEATURE COMPUTATION  (Map)                                        │
│  • compute derived features from the joined record                           │
│  • aggregate variant burden scores, expression summaries, PCA projections    │
│  • encode categoricals, handle missingness explicitly                        │
│  • validate feature ranges and flag anomalies per sample                     │
│                                                                               │
│  Google: Flume DoFns, internal feature libraries                             │
│  Open:   Spark UDFs, PyArrow compute functions, pandas on Arrow batches      │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │  Shuffle by training partition key
                                   ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  STAGE 4 · PARTITION & WRITE  (Reduce)                                       │
│  • hash-partition by sample_id (NOT sequential — prevents cohort leakage)    │
│  • stratified sampling to balance population subgroups                       │
│  • schema validation before write                                            │
│  • write to columnar training format, partitioned for efficient ML reads     │
│  • register snapshot in table catalog                                        │
│                                                                               │
│  Google: Capacitor (columnar) + Flume sinks + Spanner for catalog           │
│  Open:   Parquet + Apache Iceberg or Delta Lake + Hive Metastore/Glue       │
└──────────────────────────────────────────────────────────────────────────────┘
```

### stage 1: extraction across heterogeneous systems

The first stage is where most pipeline complexity accumulates. Each source system has its own connector, its own schema, and its own failure modes. The sequencing platform exports in FASTQ or BAM derivatives. The variant store has its own query API. The expression database might be row-oriented HDF5 files. None of them agree on what a sample ID looks like.

In a Google environment, Flume's pipeline model handles this cleanly: you write a reader for each source as a PCollection source, and the framework manages parallelism and fault tolerance. Outside Google, the same pattern applies — Spark's Dataset API with custom source connectors, or Beam pipelines with IO transforms.

The key output of this stage is not the data itself but the normalized key space. Every downstream stage depends on a canonical sample identifier that resolves across systems. Getting this wrong means silent join failures downstream — samples that appear to have no variants, or no phenotype, because the key didn't match. At PB scale, silent failures are expensive to discover.

### stage 2: the shuffle is where you pay

Stage 2 is the most expensive operation in the pipeline and the one where framework choice makes the most difference. Co-grouping by sample ID requires a full shuffle: every record for a given sample, from every source, must arrive at the same reducer. At petabyte scale, this shuffle can dwarf everything else in cost and time.

In Flume, `CoGroupByKey` handles this idiomatically — you pass N PCollections keyed by the same entity key and receive, per key, a tuple of iterables from each source. The framework optimizes the shuffle. Apache Beam exposes the same abstraction, runnable on Dataflow or Spark. In native Spark, `sortMergeJoin` or `cogroup` on DataFrames achieves the same result, with the optimizer making decisions about broadcast vs shuffle join thresholds.

The reduce step here also handles the one-to-many problem. One genomic sample may have millions of called variants. Representing those as a repeated field in the output record, or aggregating them into summary statistics, is a decision that determines the shape of every downstream feature. It cannot be deferred.

### stage 3: feature computation and PyArrow

Feature computation is embarrassingly parallel — each sample's joined record can be processed independently. This is where the in-memory layer matters most, and where PyArrow has become central to the open-source stack.

Apache Arrow defines a columnar in-memory format for tabular data. PyArrow is the Python library for working with it. When Spark executes a Python UDF, it serializes data to Arrow batches before passing them to Python — the `mapInArrow` and `applyInPandas` APIs make this explicit. Processing a batch of samples as an Arrow RecordBatch rather than row by row is typically an order of magnitude faster because it avoids Python interpreter overhead on every row and keeps data in contiguous memory.

For feature computation outside of Spark — say, in a distributed worker pool processing Parquet shards directly — PyArrow's dataset and compute APIs are the right tool. `pyarrow.dataset.dataset()` lazily references a partitioned Parquet directory without loading it into memory. `pyarrow.compute` provides vectorized operations (arithmetic, string ops, conditional expressions) that run on Arrow arrays. The pattern looks like:

```python
import pyarrow.dataset as ds
import pyarrow.compute as pc

dataset = ds.dataset("gs://bucket/samples/", format="parquet")
for batch in dataset.to_batches(columns=["variant_burden", "expression_pc1"]):
    result = pc.add(batch["variant_burden"], pc.multiply(batch["expression_pc1"], 0.5))
```

Google's equivalent is internal — a typed column processing library operating on Capacitor-format data with similar vectorized semantics. The Arrow model is a faithful open-source analog.

PyArrow also handles Parquet I/O directly, with control over row group size, compression codec (Snappy and Zstd are standard choices), and dictionary encoding for categorical columns. For writing training shards, `pyarrow.parquet.write_to_dataset()` handles partitioning natively.

### stage 4: writing for training reads

The final stage determines how efficiently a training job can read the dataset. A few decisions here have outsized impact.

Partition by a hash of the entity key, not by any semantic attribute of the entity. Partitioning by cohort or population group leaks cohort membership into the train/validation split if you later partition by the same key for evaluation. A hash partition on sample ID distributes evenly and preserves split integrity.

Row group size in Parquet (the unit of columnar data within a file) should be tuned for the read pattern. A training job reading all columns benefits from larger row groups (128–256 MB). A job that reads a narrow feature subset benefits from smaller row groups with predicate pushdown. For ML training, 128 MB row groups with Snappy compression is a defensible default.

Table formats matter here. Raw Parquet files are immutable. When upstream variant calls are corrected, or new samples are added, you need either a full rewrite or an incremental update mechanism. Apache Iceberg solves this with snapshot isolation and partition evolution — you can add new files to a table snapshot without rewriting existing ones, and point-in-time queries let training jobs read a consistent snapshot even as new data arrives. Delta Lake offers the same semantics with tighter Databricks integration. Google's internal systems handle this transparently; in an open-source stack, you have to choose and operate one of these table formats explicitly.

### where the stacks actually differ

The processing model is more similar than it appears. Flume and Beam are literally the same abstraction. Spark covers most of the same ground. The real divergence is in three areas.

Schema enforcement. Protobuf in Google's stack makes schema changes visible and reviewable — a field removal is a code change that goes through review. In a Parquet stack, column renames and type changes are silent unless you build schema validation as an explicit pipeline step. This is operational discipline, not a framework feature, and it requires deliberate investment.

The catalog layer. Google's internal metadata systems track lineage, schema history, and dataset snapshots with fine granularity. The open-source equivalents — Apache Atlas, AWS Glue Data Catalog, Hive Metastore — are functional but require more operational care. Iceberg's built-in metadata goes further than raw Parquet but still requires a catalog service to persist and query it.

The in-memory transport layer. Arrow Flight, a gRPC-based protocol for transferring Arrow data between processes, is beginning to replace custom data transfer protocols in open-source pipelines. Inside Google, this problem is solved by internal RPC infrastructure. Arrow Flight is the credible open-source answer and is worth understanding if you are building systems where multiple services exchange large columnar payloads.

### my learnings

When I review designs for this class of problem, I look at a few things. First, where the entity key normalization happens and whether it is treated as a first-class artifact or an implicit join. Silent key mismatches at this scale are expensive to find and expensive to fix. Second, whether the shuffle cost in stage 2 has been estimated — it is almost always the bottleneck, and it is almost always underestimated. Third, whether the table format choice accounts for incremental updates, because source data will change and a pipeline that requires full reprocessing on every upstream correction does not scale operationally.

The comparison between Google's stack and the open-source ecosystem is less about capability and more about what is explicit versus what is handled for you. The open-source stack can do everything that Flume, Capacitor, and internal metadata systems do. What you give up is the implicit infrastructure: you have to choose and operate each layer yourself, and the choices are load-bearing in ways that are not always obvious until you are deep into a production incident.
