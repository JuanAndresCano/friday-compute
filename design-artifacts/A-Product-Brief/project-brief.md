---
title: "Product Brief — University AI Compute Management Platform (friday-compute)"
status: final
created: 2026-08-31
updated: 2026-08-31
author: "Juan Andrés Cano Ramírez (A00399560)"
course: "AI Assisted Development Process — Spec Task 2, Deliverable A"
generated_via: "WDS wds-1-project-brief posture (bmad-product-brief), fast/draft path"
---

# Product Brief — University AI Compute Management Platform

> Codename: **friday-compute**. This brief is the strategic "Why" that anchors `SPEC.md`
> (the "What") and `ARCHITECTURE-SPINE.md` (the structural "How"). It does not define
> capabilities, contracts, or invariants — it defines the mandate they answer to.

## 1. Vision & Academic Purpose

The university runs a shared, heterogeneous local GPU fleet for teaching, experimentation,
and model training. Today that fleet is governed by hand: access is negotiated over email
and hallway conversations, there is no enforced ceiling on how much compute a student can
spend against a course budget, and a long research training job can occupy the exact
hardware a scheduled class needs an hour later. The result is unpredictable classes,
invisible spend, and disputes that only Academic Administration can settle.

**friday-compute governs access to local LLM inference and training across the shared GPU
fleet so that academic priorities, fairness, operational control, and observability are all
preserved at once — enforced by the system rather than negotiated by people.** It is a
control plane: it decides who may run what, where, and when; it guarantees reserved
classroom hardware; it meters token and GPU-time consumption against academic budgets; and
it makes the live state of the fleet visible to everyone who depends on it.

The platform is institutional infrastructure, not a product for sale. Its success is
measured in classes that start on time, denials that come with a reason, and utilization
reports an administrator can defend in a budget meeting.

## 2. Core Concept

A request to use compute — an interactive inference session, a single-node training run, or
a distributed training job — enters through one governed path. The platform resolves it
against three things: **who is asking** (role tier and academic-context membership), **what
they are asking for** (model, hardware class, worker count), and **what is available**
(fleet state, quota balances, active and upcoming reservations). It returns one of a small,
named set of outcomes — running, queued with a position, pending administrative review, or a
specific rejection reason — never a generic failure.

Instructors reserve hardware for class windows in advance. When a reserved window opens, the
platform clears unreserved work off that hardware with a grace period to checkpoint, so the
class gets exclusive access without anyone paging an operator. Every reservation change,
quota override, and administrative bypass is written to an audit trail.

## 3. Proto-Personas

Names below are fictional placeholders; only the **role tier** and the goal/friction
content are load-bearing, and the tiers match Spec-1's coarse role model
(student, instructor, researcher, operator, admin).

| Persona | Role tier | Primary goal | Core friction today | What the platform gives them |
|---|---|---|---|---|
| **Dr. Elena Navarro — Academic Administrator** | admin | Fair, policy-aligned use of institutional hardware; defensible utilization and spend reporting | No enforced quotas; no consolidated usage data; every conflict escalates to her | Quota policy that the system enforces; utilization and token/GPU-hour reports per course, group, and role; a complete audit trail of overrides |
| **Prof. Marco Díaz — Course Instructor** | instructor | The GPUs my class needs are free when class starts; my students can reach only the models the course approves | Must email lab staff to "hold" machines; no guarantee they stay held; students hit models the course did not sanction | Advance reservations with enforced exclusive lockout; per-course model catalog; visibility into class quota consumption |
| **Sara Molina — Student** | student | Transparent access to approved models, a clear usage limit, and an understandable reason when a request is refused | Silent failures and "the GPU is busy"; no idea how much of the course budget is left | Named rejection reasons, queue position and estimated start time, a live view of personal and course quota balance |
| **Tomás Herrera — Lab Operator / Infrastructure Engineer** | operator | Know machine health, occupancy, reservations, and failures without SSH-ing into nodes | Manual tracking in spreadsheets; discovers failing nodes when a user complains | Live per-node health state (available, reserved, busy, idle, failing); reservation calendar; preemption events surfaced, not hidden |
| **Dr. Nadia Farouk — Privileged AI Researcher** | researcher | Run long-running and distributed training jobs on the high-capacity node under controlled conditions | Ad-hoc coordination to avoid colliding with classes; jobs killed with no distinction from a crash | A sanctioned window for distributed / high-VRAM jobs; admin-granted overrides; preempted jobs recorded as *preempted*, not *failed* |

Detailed motivations, friction points, and permission boundaries for each persona live in
`_bmad-output/specs/spec-ai-compute-platform/companion-files/persona-archetypes.md`.

## 4. Value Proposition & Differentiation

**Centralized governance replaces manual coordination and ad-hoc desktop access.** The value
is not a nicer interface over the cluster — it is that scheduling promises, spending limits,
and access rules become *enforceable guarantees* instead of social conventions.

- **vs. manual coordination (email + spreadsheets):** removes the human bottleneck and the
  ambiguity. A reservation is honored by the scheduler, not by whoever remembered the email.
- **vs. ad-hoc direct access to GPU desktops:** ends the "first to SSH wins" allocation
  model; every workload is admitted against quota and fleet state.
- **vs. a generic cluster scheduler (raw Kubernetes / Slurm):** adds the academic layer a
  bare scheduler has no concept of — course and research-group budgets, token-level
  metering, instructor reservations that preempt research, role-tiered permissions, and
  denial reasons written for a student rather than an SRE.
- **vs. a public-cloud AI platform:** stays on-premises and on the institution's own
  heterogeneous hardware, with no external billing dependency and no data leaving the
  network.

## 5. High-Level Success Metrics

Property-level summary only. The binary acceptance tests, provenance of each number, and
the replace-with formulas for tagged defaults live in `SPEC.md § Success Signal` — this
table is not restated there.

| # | Property | Provisional number |
|---|---|---|
| KPI-1 | Zero class reservations fail to receive exclusive hardware at their scheduled start | window `[ILLUSTRATIVE DEFAULT]` — see SPEC |
| KPI-2 | Pre-emptive token-quota enforcement holds actual consumption within a bounded margin of the reserved cap under realistic concurrent load | margin & concurrency `[ILLUSTRATIVE DEFAULT]` — see SPEC |
| KPI-3 | 100% of rejected workloads return one specific named reason; zero generic failures | 100% (categorical) |
| KPI-4 | 100% of workloads needing merely-busy (not absent) hardware are queued, never rejected | 100% (categorical) |
| KPI-5 | Usage reports (token consumption, GPU-hours) available per course / research group / role | staleness bound set in SPEC |
| KPI-6 | 100% of conflicting later reservations receive an equivalent-or-better substitute offer or administrative review — never a bare rejection | 100% (categorical) |
| KPI-7 | Every reservation change, quota override, and administrative bypass produces a matching, retrievable log entry | 100% (categorical) |

Rubric-named metrics **"fair quota distribution"** and **"utilization rate"** are carried by
KPI-2/KPI-5 respectively; neither is given a fabricated numeric target here because the
fleet's realistic peak-capacity baseline is unknown at spec time.

## 6. High-Level Constraints

Institutional and physical boundaries that shape every downstream decision. Technical
constraints are enumerated normatively in `SPEC.md § Constraints`.

- **Heterogeneous fleet.** Many single-GPU nodes at 24 GB VRAM plus one high-capacity node
  with 2× 48 GB VRAM. Placement logic cannot assume uniform hardware.
- **On-premises only.** No public-cloud compute, no external billing or payment gateway,
  no institutional data leaving the university network.
- **Kubernetes-mediated hardware.** GPUs are exposed as schedulable resources via device
  plugins; distributed training is mediated by the Kubeflow Training Operator. The platform
  governs *through* these mechanisms and inherits their limitations.
- **Pre-emptive enforcement.** Quota and reservation checks happen before a workload starts,
  not after it has already consumed resources.
- **Academic priority is non-negotiable.** A live classroom reservation outranks any
  background research or ad-hoc workload on the same hardware.
- **Every bypass is logged.** No silent administrative override path exists.
- **Design-phase boundary.** This deliverable stops at specification and architecture; no
  application code, schema migrations, or runtime deployment configuration (Spec Task 2 §6).

## 7. Scope Boundary (summary)

**In:** governance of access, reservations, quotas, token/GPU-time metering, fleet health
visibility, preemption rules, and the audit trail for the shared local GPU fleet.

**Out:** public-cloud billing or fintech/payment gateways, custom GPU hardware, end-user
model-authoring IDEs, unmanaged direct SSH to raw GPU nodes, physical provisioning and
driver configuration, and the final internal module split / database schema / UI design /
deployment topology (these emerge in planning, not here). The normative exclusion list is
`SPEC.md § Non-Goals`.

## 8. Open Questions (strategic)

Carried into the SPEC and Architecture phases for resolution.

- Is a single institution-wide quota policy sufficient for iteration 1, or must the platform
  support per-course policy overrides from the start?
- Does full-cohort simultaneous load (every student in a course launching at once) need to
  be a first-class design target, or is moderate concurrency an acceptable iteration-1
  assumption? (Spec-1 tagged this **Risky**.)
- Who owns the source of truth for academic-context membership and the class schedule — is
  it always upstream, or does the platform need a fallback authoring path?
