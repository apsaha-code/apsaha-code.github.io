+++
title = "Evaluations - a leadership angle!"
date = 2025-12-06
[taxonomies]
tags = ["evals", "leadership"]
+++

Evaluation is often framed as a technical challenge: better datasets, better metrics, more labels, more benchmarks. When  AI systems fail, the instinct is to look for gaps in the evaluation pipeline or shortcomings in the model.

From working on building foundation-model to Applied AI systems in real deployments, I’ve come to believe **this framing is incomplete**.

Many evaluation failures persist not because teams lack technical capability, but because evaluation is fundamentally a leadership problem. It sits at the intersection of uncertainty, incentives, judgment, product understanding and accountability — areas where technical improvements alone are rarely sufficient.

<!-- more -->

### Where evaluation gets hard

In production settings, evaluation rarely answers a clean question like “Is the model correct?” More often, it asks questions such as:

* Are observed gains meaningful, or artifacts of proxies?
* How much uncertainty are we implicitly accepting?

These are not questions with crisp technical answers. These require judgment, and that judgment cannot be fully delegated to metrics.

What I’ve experienced repeatedly is that teams can surface uncertainty, **but struggle with how to act on it**. When that happens, evaluation drifts toward reassurance rather than decision support: metrics are produced, dashboards look nice, yet underlying risks remain poorly examined.

### Metrics rarely fail in isolation

Most evaluation pathologies emerge under pressure:

➜  Pressure to ship  
➜  Pressure to demonstrate progress  
➜  Pressure to justify prior investment  

Under these conditions, even thoughtfully designed metrics can change role. They stop functioning as signals and start functioning as shields.

The incentives subtly reshape interpretation:

➜  Ambiguous signals are resolved optimistically  
➜  Proxy metrics are treated as outcomes  
➜  Uncertainty is reframed as “acceptable risk” without being fully articulated/understood.

Once this dynamic sets in, adding more data or more metrics tends to increase confidence faster than it increases understanding.

### Evaluation is not just a development function

At some point, evaluation is not just about measurement but also drives the choices:

* Do we ship?
* Do we delay?
* Do we stop entirely?

One pattern I’ve seen repeatedly is evaluation being treated as an extension of the development process. The same team that builds the system are asked to define, track, and interpret the evaluation metrics.

This is understandable. Development teams know the models best. They are closest to the implementation details. Evaluation, however, is not about optimization. It is about adjudication.

It requires someone with knowledge and authority to say:

* This improvement is not meaningful.
* This metric does not reflect the real objective.
* We are not ready to ship.

Those calls are fundamentally different from building or iterating.

When evaluation ownership sits entirely within the development team, subtle bias is almost inevitable. Not because of bad intent, but because incentives and accountability are aligned toward progress, not constraint.

### Who should own evaluation?

In organizations where evaluation works well, ownership is explicit and senior. The right owner is not the development team alone. It is typically a domain-accountable product leader -- someone who: 

* Has deep understanding of how the system is actually used
* Is senior enough to block or delay launches
* Is accountable for real-world outcomes, not just technical outputs
* Will personally absorb the consequences if the system fails

Most importantly, they must have **skin in the game**.

Evaluation ownership without decision authority is symbolic --> Decision authority without accountability is dangerous.

A healthy split of delivery often looks like this:

**Evaluation metrics owner (domain-accountable leader):**
* Defines what “good” actually means in the real world
* Decides which metrics matter
* Sets kill criteria and launch gates
* Owns the final ship / no-ship decision
* Is accountable for downstream consequences

**Model / AI system development team:**
* Implements the system
* Designs and runs technical evaluations
* Ensures reproducibility and statistical rigor
* Builds baselines and ablations
* Surfaces uncertainty transparently

The development team ensures evaluation is technically sound.
The evaluation owner ensures evaluation constrains action.

Both are essential — but they are not interchangeable.

That ownership clarity changes behavior in subtle but important ways:

* Metrics are tied to decisions, not just reporting
* Assumptions are surfaced earlier
* Kill criteria are defined before launch
* Tradeoffs are debated explicitly

Without clear — and correct — ownership, evaluation risks becoming an endless refinement exercise that never meaningfully constrains action.

### Evaluation also reflects values and goals, not just technical skills

What gets evaluated and what is ignored reveals what an organization actually values. 

If evaluation emphasizes short-term task metrics, long-horizon failures will be missed.
If user adaptation is ignored, feedback loops will be misunderstood.
If evaluation never blocks a launch, it is not functioning as evaluation.

These are leadership choices, whether made explicitly or by default.

### My learnings

When reviewing applied AI systems or initiaing work on building a new one, I pay less attention to whether evaluation artifacts exist and more attention to who owns them and how they are used.

The questions I ask:

* Who owns these evaluation metrics, and are they accountable for outcomes?
* Who is this evaluation meant to inform?
* What decision would change if these signals worsened?
* Where might we be overconfident?
* What uncertainty are we accepting implicitly?

As AI systems become more capable and more consequential, evaluation will continue to be discussed primarily as a technical challenge.

My experience suggests that many of the most costly failures are not caused by missing metrics or insufficient data. They are caused by unclear ownership when tradeoffs arise and reluctance to make uncomfortable decisions.

Evaluation works best when leaders treat it as a decision discipline — one that requires accountability, judgment, and the willingness to slow down to define metrics that actually constrain action.