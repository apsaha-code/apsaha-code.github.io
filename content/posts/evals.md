+++
title = "evaluations - a leadership angle!"
date = 2025-12-06

[taxonomies]
tags = ["evals", "leadership"]
+++

Evaluation is often framed as a technical challenge: better datasets, better metrics, more labels, more benchmarks. When AI systems fail, the instinct is to look for gaps in the evaluation pipeline or shortcomings in the model.

From working on building foundation-model to Applied AI systems in real deployments, I've come to believe **this framing is incomplete**.

Many evaluation failures persist not because teams lack technical capability, but because evaluation is fundamentally a leadership problem. It sits at the intersection of uncertainty, incentives, judgment, product understanding and accountability — areas where technical improvements alone are rarely sufficient.

<!-- more -->

### where evaluation gets hard

In production settings, evaluation rarely answers a clean question like "Is the model correct?" More often, it is asking whether observed gains are meaningful or just artifacts of proxies, and how much uncertainty the team is implicitly accepting. These are not questions with crisp technical answers. They require judgment, and that judgment cannot be fully delegated to metrics.

What I've experienced repeatedly is that teams can surface uncertainty, **but struggle with how to act on it**. When that happens, evaluation drifts toward reassurance rather than decision support: metrics are produced, dashboards look nice, yet underlying risks remain poorly examined.

### metrics rarely fail in isolation

Most evaluation pathologies emerge under pressure — pressure to ship, pressure to demonstrate progress, pressure to justify prior investment. Under these conditions, even thoughtfully designed metrics can change role. They stop functioning as signals and start functioning as shields.

The incentives subtly reshape interpretation. Ambiguous signals get resolved optimistically. Proxy metrics get treated as outcomes. Uncertainty gets reframed as "acceptable risk" without being fully articulated or understood. Once this dynamic sets in, adding more data or more metrics tends to increase confidence faster than it increases understanding.

### evaluation is not just a development function

At some point, evaluation is not just about measurement — it drives the actual choices: do we ship, do we delay, do we stop entirely? One pattern I've seen repeatedly is evaluation being treated as an extension of the development process, where the same team that builds the system is also asked to define, track, and interpret the evaluation metrics.

This is understandable. Development teams know the models best and are closest to the implementation details. But evaluation is not about optimization. It is about adjudication. It requires someone with the knowledge and authority to say that an improvement is not meaningful, that a metric does not reflect the real objective, that the system is not ready to ship. Those calls are fundamentally different from building or iterating.

When evaluation ownership sits entirely within the development team, subtle bias is almost inevitable — not because of bad intent, but because incentives and accountability are aligned toward progress, not constraint.

### who should own evaluation?

In organizations where evaluation works well, ownership is explicit and senior. The right owner is not the development team alone. It is typically a domain-accountable product leader: someone who has deep understanding of how the system is actually used, is senior enough to block or delay launches, is accountable for real-world outcomes rather than just technical outputs, and will personally absorb the consequences if the system fails. Most importantly, they must have **skin in the game**.

Evaluation ownership without decision authority is symbolic. Decision authority without accountability is dangerous.

A healthy split looks like this: the evaluation metrics owner — the domain-accountable leader — defines what "good" actually means in the real world, decides which metrics matter, sets kill criteria and launch gates, owns the final ship/no-ship decision, and is accountable for downstream consequences. The model development team, in turn, implements the system, designs and runs technical evaluations, ensures reproducibility and statistical rigor, builds baselines and ablations, and surfaces uncertainty transparently.

The development team ensures evaluation is technically sound. The evaluation owner ensures evaluation constrains action. Both are essential — but they are not interchangeable.

That ownership clarity changes behavior in subtle but important ways. Metrics get tied to decisions rather than just reporting. Assumptions get surfaced earlier. Kill criteria get defined before launch. Tradeoffs get debated explicitly. Without clear — and correct — ownership, evaluation risks becoming an endless refinement exercise that never meaningfully constrains action.

### evaluation reflects values, not just technical skill

What gets evaluated and what is ignored reveals what an organization actually values. If evaluation emphasizes short-term task metrics, long-horizon failures will be missed. If user adaptation is ignored, feedback loops will be misunderstood. If evaluation never blocks a launch, it is not functioning as evaluation. These are leadership choices, whether made explicitly or by default.

### my learnings

When reviewing Applied AI systems or initiating work on building a new one, I pay less attention to whether evaluation artifacts exist and more attention to who owns them and how they are used. The questions I return to are: who owns these metrics and are they accountable for outcomes, what decision would actually change if these signals worsened, where might we be overconfident, and what uncertainty are we accepting implicitly.

As AI systems become more capable and more consequential, evaluation will continue to be discussed primarily as a technical challenge. My experience suggests that many of the costly failures are not caused by missing metrics or insufficient data — they are caused by unclear ownership when tradeoffs arise and a reluctance to make uncomfortable decisions.

Evaluation works best when leaders treat it as a decision discipline: one that requires accountability, judgment, and the willingness to slow down to define metrics that actually constrain action.
