# Glossary — University AI Compute Management Platform

Canonical domain terms for `SPEC.md` and `ARCHITECTURE-SPINE.md`. A term defined here means
the same thing in every artifact and downstream story. Where a term maps to a specific
capability, constraint, or architecture decision, the identifier is given.

## Actors & scoping

| Term | Definition |
|---|---|
| **Role tier** | One of `{student, instructor, researcher, operator, admin}`. A coarse classification that decides which *actions* an actor may structurally perform — not which models or courses they can reach. Fixed set (SPEC CAP-3, C-7). |
| **Academic context** | A course, lab, or research group. Carries a token budget pool, a GPU-hour budget pool, and an allowed-model catalog. Membership and lifecycle are owned upstream (SPEC NG-10, A-9); the platform governs *usage within* a context. |
| **Allowed-model catalog** | The set of model ids a given academic context may run. A request for a model outside it is rejected `unauthorized-model-for-context` (SPEC CAP-1, CAP-3). |
| **Actor** | Any authenticated principal (a person or a service) carrying a role tier and, for workload actions, an academic-context membership. |

## Workloads & admission

| Term | Definition |
|---|---|
| **Workload** | A single unit of compute demand: interactive inference, single-node training, or distributed training. Submitted against an academic context with a resource requirement. |
| **Resource requirement** | The tuple a workload or reservation binds to: GPU count, minimum per-GPU VRAM, architecture class, worker count, and duration or window. Per SPEC C-13 a workload binds to this, **never** to a physical node. |
| **Admission outcome** | Exactly one of `{EXECUTING, QUEUED, PENDING_ADMIN_REVIEW, REJECTED}` returned for every submission (SPEC CAP-1). |
| **Named rejection reason** | One value from the closed set `{unauthorized-model-for-context, exhausted-personal-quota, exhausted-context-quota, hardware-class-absent-from-fleet, restricted-window-violation}`. A `REJECTED` outcome always carries exactly one; a generic failure is never returned (SPEC CAP-1). |
| **Quota snapshot** | The `{personal_remaining, context_remaining}` balance pair returned with *every* admission outcome, including rejections (SPEC CAP-1). |
| **Queue position** | The exact integer rank of a `QUEUED` workload among workloads admitted ahead of it on the same hardware class. Binary-verifiable against actual dequeue order (SPEC CAP-1). |
| **Estimated start time** | An explicitly *advisory* projection returned with a `QUEUED` outcome, computed from the projected release times of the workloads ahead on that hardware class. Not a guarantee (SPEC CAP-1). |
| **Lifecycle state** | The authoritative status of an admitted workload: `{REQUESTED, SCHEDULED, ACTIVE, PREEMPTED, COMPLETED, FAILED}`. `PREEMPTED`, `COMPLETED`, `FAILED` are terminal. Governed by an explicit transition table (SPEC CAP-8); see `diagrams/state-transitions.mmd`. |

## Quota & metering

| Term | Definition |
|---|---|
| **Token bucket quota** | A budget pool (personal or academic-context) measured in tokens, drawn down pre-emptively at admission and settled at reconciliation (SPEC CAP-2). "Bucket" = a balance that is decremented and can be refilled by policy; it is not a rate limiter. |
| **Personal balance** | A per-user token budget. Debited in the same transaction as the context balance (ARCH AD-5). |
| **Context balance** | A per-academic-context token budget. Both balances must permit a reservation or the workload is rejected before execution (SPEC CAP-2). |
| **Estimated cap / reservation** | The projected token ceiling reserved against both balances at admission. Generation is truncated locally if it is reached (SPEC CAP-2, ARCH AD-13). |
| **Reconciliation** | The asynchronous settlement of a completed workload's *reserved* estimate against its *actual* token consumption, releasing any unused reservation back to both balances. Triggered by a `workload.tokens.finalized` event (ARCH AD-13). |
| **Ledger entry** | One immutable per-workload record of reserved and reconciled token and GPU-time consumption. Usage reports (SPEC CAP-11) are sums of ledger entries for a scope. |
| **GPU-time** | Wall-clock time a workload held one or more GPUs, accrued per workload and reported alongside token consumption (SPEC CAP-11). |
| **Optimistic-balance window** | The interval between a workload completing and its reconcile event landing, during which a balance still reflects the estimate rather than actuals (ARCH AD-13). |

## Reservations & exclusivity

| Term | Definition |
|---|---|
| **Reservation** | An instructor's claim on hardware of a given class for a future time window, for exclusive class use. Resolves to `{GRANTED, SUBSTITUTE_OFFERED, PENDING_ADMIN_REVIEW}` (SPEC CAP-5). |
| **Exclusive lockout window** | The time window of a `GRANTED` reservation during which the reserved hardware is made exclusively available to it; non-reserved workloads are relocated or preempted off it (SPEC CAP-6). |
| **Hardware class** | A set of interchangeable allocatable units with equivalent capability — e.g. the 24 GB single-GPU class (many units) and the 2× 48 GB class (SPEC C-1). |
| **`unit_count`** | The number of interchangeable allocatable units in a hardware class. Bounds how many reservations for that class may overlap in time (ARCH AD-6). |
| **Capacity slot** | An abstract index `(class_id, slot_no ∈ 1..unit_count)` a granted reservation holds for its window. A capacity accounting device, **not** a physical node — the node is chosen at window start (ARCH AD-6, AD-7). |
| **Equivalent-or-better** | A node `C` satisfies a requirement `R` over span `S` iff: per-GPU VRAM(C) ≥ R.min_vram; GPU count(C) ≥ R.gpu_count; C's architecture class is in R's compatible set; and C is free for all of `S`. Among qualifiers the smallest sufficient node wins (min VRAM, then GPU count, then node id). A pure deterministic predicate shared by CAP-5, CAP-6, CAP-10 (SPEC CAP-7, ARCH AD-7). |
| **Substitute offer** | An equivalent-or-better node returned when the exact requested reservation hardware is unavailable (SPEC CAP-5, CAP-7). |
| **`PENDING_ADMIN_REVIEW`** | The escalation outcome when neither the requested hardware nor any equivalent-or-better substitute is available, or when a policy decision needs a human (SPEC CAP-1, CAP-5). |

## Placement, relocation & preemption

| Term | Definition |
|---|---|
| **Placement** | Assigning a workload to a concrete node that satisfies its resource requirement (SPEC CAP-4). The binding is advisory and expected to change (SPEC C-13). |
| **`current_placement_ref`** | The nullable, last-observed node a workload is running on. Never a key, never read to make a governance decision (ARCH AD-4). |
| **Relocation** | Moving a running workload from one node to an equivalent-or-better node **without** changing its lifecycle state or queue standing. Invoked on node failure (CAP-10) and at reservation window start (CAP-6). Not a lifecycle transition (SPEC CAP-8, ARCH AD-8). |
| **Preemption-avoidance** | The relocation attempt CAP-6 makes *before* preempting a non-reserved workload at window start (SPEC CAP-6, ARCH AD-8). |
| **Preemption** | The fallback when relocation finds no qualifying node at window start: termination signal → grace period to checkpoint → forced termination → terminal state `PREEMPTED` (SPEC CAP-6). |
| **`PREEMPTED` vs `FAILED`** | `PREEMPTED` = terminated by the platform to honour a reservation or after an unrecoverable relocation failure, recorded distinctly so usage reports can tell it from a crash. `FAILED` = ended by an error. `PREEMPTED` is terminal and non-resumable (SPEC CAP-6, CAP-8). |
| **Grace period** | The fixed, tunable interval a workload is given to checkpoint after a termination signal before forced termination (SPEC CAP-6, C-6). |
| **Front-of-queue re-entry** | The CAP-10 fallback when a failed node's workload cannot be relocated: it re-enters the queue ahead of every other pending submission of any type (SPEC CAP-10). |
| **Preemptible priority tier** | *Not a platform concept.* Every non-reserved workload is equally subordinate to an active reservation; there is no priority tier that can override one (SPEC C-10). Listed here to mark it explicitly out of the model. |

## Fleet, telemetry & health

| Term | Definition |
|---|---|
| **Fleet** | The set of GPU machines assignable to workloads: many 24 GB single-GPU nodes plus one 2× 48 GB node, in a physically uncontrolled student lab of under 40 machines (SPEC C-1, C-12). |
| **Node health state** | Exactly one of `{available, reserved, busy, idle, failing}` per node, computed by the Telemetry Ingestion Engine and queryable by any authenticated user (SPEC CAP-9). |
| **VRAM allocation unit** | The whole-GPU granularity at which the platform allocates GPU memory in iteration 1 — no MIG partitioning or time-slicing (SPEC A-8, C-2). One unit = one physical GPU's VRAM. |
| **Device plugin** | The Kubernetes mechanism (NVIDIA GPU Operator / `k8s-device-plugin`) that exposes GPUs as schedulable resources. The platform governs *through* it (SPEC C-2). |
| **Device plugin exporter / DCGM exporter** | The component that publishes per-GPU metrics (utilisation, memory, health) consumed on the telemetry plane to derive node health state (ARCH AD-9). |
| **Control plane** | The five platform binaries plus PostgreSQL, the event bus, and the metrics store, running on isolated server infrastructure (SPEC A-11) — never schedulable onto a lab node (ARCH Deployment). |
| **Telemetry plane** | The asynchronous path carrying raw node/GPU/serving-layer signals over the event bus into the Telemetry Ingestion Engine. Never on the synchronous governance path (ARCH AD-9). |
| **Governance plane** | The synchronous request path: admission, quota, scoping, placement decisions, audit. Reads the transactional store, never blocks on telemetry (ARCH AD-3, AD-9). |

## Training

| Term | Definition |
|---|---|
| **Distributed training** | A multi-worker training job spanning several nodes, expressed as a Kubeflow Trainer v2 `TrainJob` (SPEC C-3, CAP-8, ARCH AD-12). |
| **`TrainJob`** | The single custom resource of Kubeflow Trainer v2 that replaces the deprecated framework-specific v1 CRDs (`PyTorchJob` / `TFJob` / `MPIJob`) (ARCH AD-12). |
| **Restricted window** | The designated time window outside which a distributed or high-VRAM training job is rejected `restricted-window-violation` unless an admin override is attached (SPEC CAP-8). |
| **Admin override** | An explicit, audited authorisation attached to a submission to run it outside its restricted window (SPEC CAP-8, CAP-12). |

## Governance records

| Term | Definition |
|---|---|
| **Audit entry** | One immutable `{actor, action, target, timestamp}` record written for every reservation change, quota override, and administrative bypass. Append-only; no update or delete path (SPEC CAP-12, C-8, ARCH AD-10). |
| **Transactional outbox** | The pattern by which an audit entry and its domain event are written in the same transaction as the mutation, then relayed to the event bus at-least-once (ARCH AD-10). |
| **Source of truth** | The PostgreSQL governance store. Cluster state is an observation, never authoritative for a decision (ARCH AD-3). |
| **`[ILLUSTRATIVE DEFAULT]`** | A tag on any numeric threshold in `SPEC.md` that is a placeholder pending a real policy decision (grace period, staleness bounds, quota margin) — not a fixed technical limit (SPEC C-6, OQ-5). |
