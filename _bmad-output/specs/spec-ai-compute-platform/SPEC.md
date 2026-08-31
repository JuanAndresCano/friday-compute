---
id: SPEC-ai-compute-platform
title: "University AI Compute Management Platform — Formal Specification Contract"
status: final
created: 2026-08-31
updated: 2026-08-31
author: "Juan Andrés Cano Ramírez (A00399560)"
course: "AI Assisted Development Process — Spec Task 2, Deliverable B"
companions:
  - companion-files/glossary.md
  - companion-files/persona-archetypes.md
  - companion-files/diagrams/system-context.mmd
  - companion-files/diagrams/state-transitions.mmd
sources:
  - ../../../docs/spec-1-compute-platform.pdf
  - ../../../docs/project - university_ai_compute_management_platform_brief.pdf
  - ../../../design-artifacts/A-Product-Brief/project-brief.md
  - ../../../docs/Workshop 2 - Ruiz A00399562 - Peer Review of Cano Spec-1.docx
---

> **Canonical contract.** This SPEC plus the files under `companions:` are the complete,
> preservation-validated contract for what to build, test, and validate. It defines the
> **WHAT** — observable behaviour and its acceptance signals — never the **HOW**.
> Structural decisions (paradigm, boundaries, stack, state-mutation rules) live in
> `ARCHITECTURE-SPINE.md`. Source documents in frontmatter are traceability only.

# University AI Compute Management Platform — Formal Specification Contract

## Why

The university's shared local GPU fleet serves teaching, research, and coursework with no
automated coordination. Long training jobs starve students who need the same hardware for a
live lab; nothing enforces how much compute a student may spend against a course budget;
instructors cannot guarantee the machines their class is scheduled on; researchers cannot
run large jobs without risking someone's class. Today these are settled by email and
hallway negotiation, and every dispute escalates to Academic Administration. This is a
**mandate-and-pain** driver: the institution requires fair, policy-aligned, auditable use
of institutional hardware, and the people who depend on the fleet — administrators,
instructors, students, lab operators, privileged researchers — need the guarantees enforced
by the system rather than negotiated by hand. The value of solving it is measurable:
classes that start on time, denials that carry a reason, and utilization the institution
can account for.

## Capabilities

Identifiers `CAP-1..CAP-12` are stable and never reused. Each states an observable intent
(WHAT) and a binary or threshold acceptance signal. Capability-to-Spec-1 provenance is in
`peer-review-remediation.md`.

- **CAP-1 — Workload Admission Decision & Named-Rejection Taxonomy**
  - **intent:** A user can submit a workload (interactive inference, single-node training, or distributed training) against an academic context, stating model id and resource requirement (GPU count, minimum per-GPU VRAM, worker count), and receive exactly one admission outcome.
  - **success:** Every submission returns exactly one of `{EXECUTING, QUEUED, PENDING_ADMIN_REVIEW, REJECTED}`. Every `REJECTED` carries exactly one reason from the closed set `{unauthorized-model-for-context, exhausted-personal-quota, exhausted-context-quota, hardware-class-absent-from-fleet, restricted-window-violation}`. Every outcome — including rejections — includes a quota snapshot (remaining personal balance, remaining context balance). A `QUEUED` outcome additionally includes: (i) a **queue position** — the exact integer rank of this workload among workloads admitted-ahead of it on the same hardware class — and (ii) an **advisory estimated start time** whose computation basis is stated (projected release times of the workloads ahead on that hardware class). The position is binary-verifiable against actual dequeue order; the estimate is explicitly advisory, not a guarantee. Hardware that exists but is merely busy yields `QUEUED`, never `REJECTED`. Verified by exercising each of the five rejection triggers and the busy-hardware case and asserting the response shape — including position correctness against dequeue order — for every one. *(Remediation of Workshop-2 Clarity finding: Spec-1 left "estimated start time" computation and accuracy undefined; position is now exact and testable, and the estimate is scoped as advisory with a stated basis.)*

- **CAP-2 — Multi-Tier Quota & Token Metering**
  - **intent:** The system meters prompt tokens, completion tokens, and GPU-time for every workload and enforces personal and academic-context budgets pre-emptively.
  - **success:** At admission an estimated token cap is reserved against **both** the personal balance and the context balance. If reserving that cap would drive either the remaining personal balance or the remaining context balance below zero, the workload is **rejected before execution** with the corresponding CAP-1 reason (`exhausted-personal-quota` or `exhausted-context-quota`) — it is never admitted and then truncated to fit. Once admitted, generation is truncated when the reserved cap is reached. On completion the ledger reconciles reserved → actual in the same billing record: a workload that exceeds its estimate is truncated so post-run `actual ≤ cap`; a workload under its estimate releases the unused reservation back to both balances. Verified by running one insufficient-balance submission (asserted rejected pre-execution), one over-estimate workload, and one under-estimate workload, and asserting the ledger and both balances for each.

- **CAP-3 — Role-Based Access Governance & Academic-Context Scoping**
  - **intent:** Every actor carries a coarse role tier that gates which actions are structurally possible; every workload is scoped to an academic context that determines reachable models and budget pools.
  - **success:** The five role tiers `{student, instructor, researcher, operator, admin}` each resolve to a fixed action set; an action outside an actor's set is refused before any resource or quota check. A model absent from the context's allowed-model catalog produces the CAP-1 rejection `unauthorized-model-for-context`. Fine-grained per-user-per-model authorisation is out of scope (NG-7).

- **CAP-4 — Heterogeneous Hardware Placement & Schedulability**
  - **intent:** The system places a workload only on a node whose hardware satisfies the workload's stated GPU count and minimum per-GPU VRAM, across a fleet of mixed node types.
  - **success:** Given the fleet (many 24 GB single-GPU nodes + one node with 2× 48 GB GPUs), a workload requiring > 24 GB per GPU **or** > 1 co-located GPU is placeable only on the high-capacity node and `QUEUED` when that node is busy; a workload requiring a hardware class absent from the fleet is `REJECTED hardware-class-absent-from-fleet`. No placement path assumes uniform nodes. Verified against a fixed fleet inventory with one workload per case.

- **CAP-5 — Classroom Reservation Admission**
  - **intent:** An instructor can request exclusive hardware for a future time window and receive an admission outcome for that request.
  - **success:** Each request resolves to exactly one of `{GRANTED, SUBSTITUTE_OFFERED, PENDING_ADMIN_REVIEW}`: `GRANTED` when the exact requested hardware class is free for the whole window; `SUBSTITUTE_OFFERED` when it is not but an equivalent-or-better node (CAP-7) is free for the whole window; `PENDING_ADMIN_REVIEW` when neither. A bare rejection is never returned. This capability is **admission only**; window enforcement is CAP-6.

- **CAP-6 — Reservation Window Enforcement (relocate, then preempt)**
  - **intent:** When a granted reservation's window begins, the system makes the reserved hardware exclusively available to the reservation — relocating non-reserved workloads off it where possible, and preempting them only as a fallback.
  - **success:** At window start, for every non-reserved workload on the reserved hardware the system first runs CAP-10's relocation procedure (CAP-7 with `S` = the workload's remaining run duration). A successfully relocated workload continues elsewhere and is **not** preempted — no `PREEMPTED` state applies to it. Only if CAP-7 returns `NO_QUALIFYING_NODE` does the fallback apply: a termination signal, a fixed grace period to checkpoint, then forced termination with resource reclaim; **only this fallback path** records the workload with terminal state `PREEMPTED`, distinct from `FAILED`. `PREEMPTED` is therefore terminal and non-resumable by construction — it is only reached after relocation has already failed. Over a simulated academic cycle: zero reservations begin without exclusive hardware; zero preempted workloads are misrecorded as `FAILED`; every workload for which a qualifying node existed was relocated rather than preempted. Trigger (window start) and fallback failure mode (checkpoint-in-time) differ from CAP-5 — separately testable. *(Remediation of Workshop-2 Atomicity finding: Spec-1 bundled this with CAP-5.)*

- **CAP-7 — Equivalent-or-Better Node Resolution (shared mechanism)**
  - **intent:** Given a resource requirement and a time span, the system deterministically selects the smallest available node that is provably equivalent-or-better, or reports that none exists. This is a single shared mechanism invoked by CAP-5 (reservation substitution), CAP-6 (preemption-avoidance relocation) and CAP-10 (node-failure relocation) — it is not a reservation-only concept.
  - **success:** A candidate node `C` qualifies as equivalent-or-better for requirement `R` over span `S` **iff all** of: (a) `per_GPU_VRAM(C) ≥ R.min_vram`; (b) `GPU_count(C) ≥ R.gpu_count`; (c) `C`'s GPU architecture class is in `R`'s compatible set (same or newer compute-capability tier); (d) `C` is free for the entire span `S` — a reservation window for CAP-5, the workload's remaining run duration for CAP-6 and CAP-10. Among qualifiers, the selected node minimises per-GPU VRAM, then GPU count, then node id ascending (fully deterministic). If no node qualifies the mechanism returns `NO_QUALIFYING_NODE`, and the calling capability decides the consequence (CAP-5 → `PENDING_ADMIN_REVIEW`; CAP-6 → termination sequence; CAP-10 → front-of-queue). Verified by asserting that, for a fixed fleet and requirement, the selection is reproducible and satisfies (a)–(d), and that each caller maps `NO_QUALIFYING_NODE` to its defined fallback. *(Remediation of Workshop-2 open item: "equivalent-or-better" was deliberately undefined in Spec-1; it is now a testable shared predicate.)*

- **CAP-8 — Distributed / High-VRAM Training Lifecycle**
  - **intent:** A distributed or high-VRAM training job runs only within its designated window (unless an admin overrides) and progresses through an observable lifecycle.
  - **success:** A distributed/high-VRAM submission outside its allowed window is `REJECTED restricted-window-violation` unless an admin override is attached; an attached override is recorded per CAP-12. An admitted training job exposes a lifecycle state in `{REQUESTED, SCHEDULED, ACTIVE, PREEMPTED, COMPLETED, FAILED}`, and only the transitions in `companion-files/diagrams/state-transitions.mmd` are permitted; `PREEMPTED`, `COMPLETED`, and `FAILED` are terminal. A CAP-10 or CAP-6 relocation is **not** a lifecycle transition — a relocated job stays `ACTIVE` on the new node. Verified by: out-of-window job without override → rejected; with override → admitted and override present in the audit trail; an illegal transition (e.g. `COMPLETED → ACTIVE`) → rejected; a relocated job observed continuously `ACTIVE`.

- **CAP-9 — Node Health State & Status Visibility**
  - **intent:** Any authenticated user can query the live state of the fleet without infrastructure-level access.
  - **success:** A status query (optionally filtered by node id) returns, per node, exactly one state from `{available, reserved, busy, idle, failing}`. No credential beyond a valid role tier is required. Reported-state staleness is bounded by a tunable freshness SLO (C-6); the concrete value is set by the operator (OQ-5) and no default is invented here.

- **CAP-10 — Node-Failure Workload Relocation**
  - **intent:** When a node enters a `failing` state, the system relocates every workload on it to another node rather than failing the workload — because a workload is admitted against capacity, not against a named physical machine.
  - **success:** On transition to `failing`, for every workload on that node the system invokes CAP-7 (`R` = the workload's original resource requirement, `S` = its remaining run duration). If a qualifying node is found, the workload continues there and is **not** marked `FAILED` or `PREEMPTED` and keeps its queue standing. If CAP-7 returns `NO_QUALIFYING_NODE`, the workload enters the queue **at the front of the entire queue** — ahead of every other pending submission regardless of workload type. This front-of-queue procedure is the one referenced by CAP-6. Verified by injecting a `failing` transition and asserting: no workload remains scheduled on the failed node; relocated workloads carry no failure or preemption state; a workload with no qualifying node is dequeued before every other pending submission.

- **CAP-11 — Usage & Consumption Reporting**
  - **intent:** A user can retrieve token and GPU-time consumption for a student scope or an academic-context scope.
  - **success:** A usage query for a student id or a context id returns prompt-token, completion-token, and GPU-time totals for that scope, equal to the sum of the per-workload reconciled ledger entries (CAP-2) for that scope. Report staleness is bounded by a tunable reporting-freshness SLO (C-6, OQ-5); no default value is invented here.

- **CAP-12 — Audit Trail for Governance Actions**
  - **intent:** Every reservation change, quota override, and administrative bypass is recorded as a retrievable, immutable entry.
  - **success:** Each such action writes exactly one entry carrying `{actor, action, target, timestamp}`. Verified by performing each governed action type once and asserting exactly one matching retrievable entry per action, and that no update or delete path exists for entries.

## Constraints

Real boundaries that bend design. Identifiers `C-1..C-13` are stable anchors for ADRs,
tests, and downstream references.

- **C-1 — Heterogeneous fleet.** Many 24 GB single-GPU nodes plus one node with 2× 48 GB GPUs. Placement, reservation, and substitute logic must handle mixed hardware; uniformity must not be assumed anywhere.
- **C-2 — Kubernetes-mediated GPUs.** GPUs are exposed as schedulable resources through device plugins. Allocation granularity is a whole GPU (no fractional/MIG partitioning in iteration 1 — see A-8). The platform governs *through* this mechanism and inherits its limits.
- **C-3 — Kubeflow-mediated distributed training.** Multi-worker training is expressed through Kubeflow Trainer v2's single `TrainJob` custom resource; the platform's training lifecycle (CAP-8) must map onto what Trainer v2 observably reports. *(Spec-2 update: was "Kubeflow Training Operator's job types"; the v1 operator is on an end-of-life track as of 2026 — see `ARCHITECTURE-SPINE.md` AD-12.)*
- **C-4 — Pre-emptive enforcement.** Quota (CAP-2) and reservation (CAP-5/CAP-6) checks occur before a workload starts consuming resources, never as after-the-fact correction.
- **C-5 — On-premises network boundary.** No public-cloud compute or bursting, and no institutional data (prompts, completions, datasets) leaves the university network.
- **C-6 — Numeric thresholds are tunable policy defaults.** Grace periods, timeouts, TTLs, staleness bounds, and quota accuracy margins are policy defaults set by their owner, not fixed technical limits. Every such value in this SPEC is tagged `[ILLUSTRATIVE DEFAULT]`.
- **C-7 — Authorisation model is not fixed.** Whether access control is realised as RBAC, ABAC, or a hybrid is an architecture decision; downstream must not assume one. The SPEC fixes only the five coarse role tiers and context-level model catalogs.
- **C-8 — Every policy bypass is logged.** Any administrative override or bypass path must produce a CAP-12 entry. This is Academic Administration's traceability mandate, not a feature toggle.
- **C-9 — Token metering depends on the serving layer.** Prompt/completion token counts are sourced from the LLM-serving layer's counters; the platform does not re-derive them (see A-1).
- **C-10 — Academic priority is absolute.** A live classroom reservation outranks any background research or ad-hoc workload contending for the same hardware; there is no priority tier that can override an active reservation.
- **C-11 — Clock synchronisation is coarse-grained only.** Cross-node clocks are assumed consistent enough to order events and queue entries; no behaviour may depend on sub-second precision (see A-7).
- **C-12 — The assignable compute fleet lives in a physically uncontrolled space.** The workload-assignable machines sit in a student lab (under 40 machines) where hardware can be disconnected or powered off without warning. The cluster control-plane must not depend on that hardware for its own operation, and node loss must be a routine, handled event (CAP-9, CAP-10) rather than an exception. *(stakeholder-confirmed by the course professor)*
- **C-13 — Workloads and reservations bind to a requirement, not to a node.** A workload or reservation binds to a resource requirement (GPU count, minimum per-GPU VRAM, architecture class, duration/window), never to the identity of a specific physical node. Node identity is only *where a requirement is currently satisfied*, and is expected to change under relocation (CAP-10) and preemption-avoidance (CAP-6) without altering the workload's or reservation's own identity. The Architecture Spine's state-mutation rules must not persist a durable workload-to-node binding as source of truth. *(shared invariant behind CAP-4, CAP-5, CAP-6, CAP-7, CAP-10)*

## Non-Goals

Explicit exclusions. Identifiers `NG-1..NG-11`. Absence of a boundary here lets downstream
fill the vacuum, so each is stated.

- **NG-1 — No public-cloud compute or burst capacity.** The fleet is the local hardware only.
- **NG-2 — No external billing, fintech, or payment gateway.** Quotas are academic budgets, not money; there is no charge-back or invoicing integration.
- **NG-3 — No physical provisioning, hardware procurement, or driver/firmware configuration.** The platform consumes an already-provisioned cluster.
- **NG-4 — No end-user model-authoring environment.** No notebook service, IDE, or fine-tuning UI; "submitting a workload" and "checking status" describe behaviour, not screens.
- **NG-5 — No unmanaged direct SSH to raw GPU nodes.** All compute access is mediated by the platform.
- **NG-6 — No internal module/microservice decomposition, database schema, UI design, or deployment topology.** These emerge in planning and `ARCHITECTURE-SPINE.md`, not in this contract.
- **NG-7 — No fine-grained authorisation beyond context-level catalogs.** Per-user-per-model or per-dataset ACLs are out of scope.
- **NG-8 — No *additional* scheduling heuristic or bin-packing algorithm.** The equivalent-or-better predicate **and its deterministic tie-break** (CAP-7 — minimise per-GPU VRAM, then GPU count, then node id) are a fixed part of this contract, not deferred. What remains an architecture concern is any *further* optimisation layered on top (throughput-maximising bin-packing, anti-fragmentation, cost/energy heuristics, predictive placement).
- **NG-9 — No identity provider or SSO implementation.** The platform consumes an upstream identity; it does not authenticate credentials itself.
- **NG-10 — No academic-context lifecycle management.** Creation of courses, labs, groups, and their membership is upstream; the platform governs *usage within* a context.
- **NG-11 — No autoscaling or capacity-planning recommendations.** The platform reports utilisation (CAP-11); it does not advise on buying hardware.

## Success Signal

### Holistic operational-readiness criteria

The platform is operationally ready when, over a simulated academic cycle:

1. Every admitted class reservation receives exclusive hardware at its scheduled start with no interference from a running background job (CAP-6).
2. Every rejected workload returns its one specific named reason — never a generic failure (CAP-1).
3. Every workload needing merely-busy (not absent) hardware is queued, not rejected — in 100% of cases (CAP-1, CAP-4).
4. Every conflicting later reservation receives either an equivalent-or-better substitute offer or administrative review — never a bare rejection (CAP-5, CAP-7).
5. Every reservation change, quota override, and administrative bypass produces a matching retrievable log entry (CAP-12).
6. Pre-emptive token-quota enforcement keeps reconciled consumption within a bounded margin of the reserved cap under realistic concurrent load (CAP-2).

### KPI detail — provenance and replace-with formulas

Full detail lives here only; `project-brief.md § 5` carries the property-level summary and does not restate this table.

| KPI | Property (real target) | Number | Provenance | Replace-with |
|---|---|---|---|---|
| KPI-1 | Zero class reservations fail to receive exclusive hardware at scheduled start | window `[ILLUSTRATIVE DEFAULT: 14-day simulated academic cycle]` | Property → Spec-1 Success Signal. Window value → Spec-2 task *example* Success Signal | One real teaching term, once class-schedule data exists |
| KPI-2 | Pre-emptive quota enforcement holds reconciled consumption within a bounded margin of the reserved cap under realistic concurrent load | margin `[ILLUSTRATIVE DEFAULT: ±5%]`; concurrency `[ILLUSTRATIVE DEFAULT: 100 concurrent inference streams]` | Property → Spec-1 (pre-emptive enforcement + reconciliation). Both numbers → Spec-2 task *example* Success Signal; Spec-1 itself commits only to "moderate concurrent load" | concurrency := `Σ over nodes (sustainable concurrent sessions per node for the target model)`; margin := the metering-accuracy SLO agreed with Academic Administration once CAP-2 reconciliation error is measured |
| KPI-3 | 100% of rejected workloads return one specific named reason; zero generic failures | 100% (categorical) | Spec-1 Success Signal (verbatim) | — (not a tunable threshold) |
| KPI-4 | 100% of merely-busy-hardware workloads queued, never rejected | 100% (categorical) | Spec-1 Success Signal (verbatim; Workshop-2 rated Verifiability Strong) | — |
| KPI-5 | Usage reports available per course / research group / role | staleness bound — no default invented (CAP-11) | Project brief (usage-reporting goal); Spec-1 "Check usage" | Operator-agreed reporting-freshness SLO (OQ-5) |
| KPI-6 | 100% of conflicting later reservations get substitute offer or admin review — never bare rejection | 100% (categorical) | Spec-1 Success Signal (verbatim) | — |
| KPI-7 | Every governed action produces a matching retrievable audit entry | 100% (categorical) | Spec-1 Success Signal + Constraints (verbatim) | — |

Rubric-named metrics **"fair quota distribution"** and **"utilisation rate"** are carried by
KPI-2 / KPI-5 respectively; neither receives a fabricated numeric target because the fleet's
realistic peak-capacity baseline is unknown at spec time.

## Assumptions

Each is tagged `[Safe]` (backed by Spec-1, the brief, or the operational context) or
`[Risky]` (an inference that, if wrong, breaks a capability). Identifiers `A-1..A-12`.

- **A-1 `[Risky]` — The LLM-serving layer reports exact prompt and completion token counts.** *Impact:* if false, CAP-2 metering and enforcement have no reliable data and need an estimation fallback; KPI-2 becomes unverifiable. *(from Spec-1)*
- **A-2 `[Risky]` — Load is moderate; a full-cohort simultaneous surge is not a design target for this iteration.** *Impact:* CAP-1 admission latency and CAP-2 concurrent quota-decrement correctness are unvalidated under surge; would need separate treatment. *(from Spec-1; open question OQ-2)*
- **A-3 `[Risky]` — An accurate class-schedule / reservation source of truth is available to the platform.** *Impact:* CAP-5 and CAP-6 have nothing correct to enforce against if the schedule is wrong or stale. *(new; open question OQ-3)*
- **A-4 `[Risky]` — The target cluster runs Kubeflow Trainer v2 with device-plugin versions that support the `TrainJob` shapes CAP-8 assumes.** *Impact:* CAP-8 lifecycle states may not map cleanly onto what Trainer v2 reports. *(new; checked at architecture phase — Trainer v2.2 is the 2026 standard, single `TrainJob` CRD)*
- **A-5 `[Risky]` — Research training workloads can checkpoint within the grace period.** *Impact:* CAP-6 preemption becomes data loss rather than a clean pause if false.
- **A-6 `[Safe]` — GPU hardware is exposed as schedulable units by the orchestration layer.** *Impact:* foundational; without it the platform has no substrate regardless of spec quality. *(from Spec-1)*
- **A-7 `[Safe]` — Cross-node clock sync is reliable enough for consistent event and queue ordering.** *Impact:* audit-log ordering (CAP-12) and queue position (CAP-1) rely on it; no sub-second precision needed (C-11). *(from Spec-1)*
- **A-8 `[Safe]` — Whole-GPU allocation granularity for iteration 1 (no MIG / fractional GPUs).** *Impact:* simplifies CAP-4 placement math; revisit if MIG is introduced. *(new; consistent with C-2)*
- **A-9 `[Safe]` — Academic contexts and their membership already exist upstream.** *Impact:* CAP-3 scoping consumes them; the platform governs usage within a context, not context creation (NG-10). *(from Spec-1)*
- **A-10 `[Safe]` — A single institution-wide quota policy is sufficient for iteration 1.** *Impact:* per-course policy overrides are deferred; if instructors need them sooner, CAP-2 gains a policy-resolution step. *(new; open question OQ-1)*
- **A-11 `[Safe]` — The control-plane runs on independent server infrastructure, isolated from the student lab.** *Impact:* node-failure relocation (CAP-10) and health reporting (CAP-9) stay available when lab machines drop; the control-plane's own provisioning and availability are managed outside this platform (NG-3, C-12). *(new; confirmed by the course professor as project stakeholder)*
- **A-12 `[Risky]` — Node failures are infrequent enough that CAP-10 front-of-queue relocation does not starve fresh submissions.** *Impact:* under frequent failures the relocation queue repeatedly front-loads ahead of new submissions, delaying them indefinitely; this fairness risk is not resolved at this design stage. *(new; open question OQ-6)*

## Open Questions

Identifiers `OQ-1..OQ-6`. Resolved questions are retained (not deleted) as an audit trail
of what was asked and answered before the architecture phase.

- **OQ-1** — `RESOLVED`: iteration 1 uses a single institution-wide quota policy; per-course overrides are deferred. Decision lives in **A-10**. *(Original question: does iteration 1 need per-course quota-policy overrides?)*
- **OQ-2** — Is full-cohort simultaneous load a first-class design target for this iteration, or is "moderate concurrency" (A-2) acceptable?
- **OQ-3** — Who owns the source of truth for academic-context membership and the class schedule, and does the platform need a fallback authoring path if upstream is unavailable (A-3)?
- **OQ-4** — `RESOLVED`: CAP-7 uses smallest-sufficient-node ranking (minimise per-GPU VRAM, then GPU count, then node id). Decision lives in **CAP-7**'s tie-break. *(Original question: smallest-sufficient vs load-balancing ranking?)*
- **OQ-5** — Concrete values for the `[ILLUSTRATIVE DEFAULT]` thresholds (grace period, staleness bounds, quota margin) — who is the policy owner and what are the agreed numbers?
- **OQ-6** — How should the platform bound the fairness impact of CAP-10 front-of-queue relocation under frequent node failure (A-12) — e.g. a relocation-vs-fresh-submission ratio, ageing of waiting submissions, or a cap on consecutive front-insertions?
