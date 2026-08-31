# Persona Archetypes — University AI Compute Management Platform

Deep profiles for the five stakeholder archetypes named in `project-brief.md § 3`. Each
covers motivation, friction today, what the platform changes, **permission boundary** (what
the role tier structurally can and cannot do — SPEC CAP-3, C-7), and the **tensions** this
persona sits inside. Names are fictional; role tiers and content are load-bearing.

---

## 1. Academic Administrator — *"Dr. Elena Navarro"*  ·  role tier: `admin`

**Motivation.** Accountable to the institution for fair, policy-aligned use of expensive
shared hardware. Needs to defend allocation and spend in budget reviews, show that no
group is quietly starving another, and prove that every exception was authorised.

**Friction today.** No enforced quotas — spend is discovered after the fact, if at all. No
consolidated usage picture across courses and research groups. Every scheduling or
fairness dispute escalates to her because there is no system ruling to point at.

**What the platform changes.** Quota policy the system enforces pre-emptively (CAP-2);
utilisation and token / GPU-hour reports per course, group, and role (CAP-11); a complete,
immutable audit trail of every override and bypass (CAP-12). Disputes are settled by a
named rejection reason or a log entry, not by her inbox.

**Permission boundary.**
- **Can:** set quota policy; grant admin overrides for out-of-window training (CAP-8);
  resolve `PENDING_ADMIN_REVIEW` reservations (CAP-5); read all usage and audit data.
- **Cannot:** silently bypass the audit trail — every admin action she takes writes a
  CAP-12 entry (C-8); create or delete academic contexts or their membership (that is
  upstream — NG-10); reorder a queue by hand or override an *active* reservation (C-10).

**Tensions.** Fairness vs. throughput (a strict quota can leave GPUs idle while a blocked
student waits); governance vs. friction (every control she adds is a step a student or
instructor must clear); her authority vs. the audit mandate that binds it.

---

## 2. Course Instructor — *"Prof. Marco Díaz"*  ·  role tier: `instructor`

**Motivation.** Wants the GPUs his class needs to be free and working when class starts,
and wants his students to reach only the models the course has sanctioned. Cares about
predictability far more than raw capacity.

**Friction today.** Must email lab staff to "hold" machines, with no guarantee the hold
survives contact with a long research job. Students reach models the course never
approved. No view of how fast the class is burning its shared budget.

**What the platform changes.** Advance reservations with an enforced exclusive lockout
window (CAP-5, CAP-6) — a running research job is relocated or preempted with a grace
period, automatically, at window start. A per-course allowed-model catalog (CAP-3). Live
visibility of class quota consumption (CAP-11).

**Permission boundary.**
- **Can:** create reservations for hardware classes for future windows (CAP-5); see his
  courses' quota balances and usage (CAP-11); submit workloads like any user.
- **Cannot:** guarantee a *specific physical machine* — a reservation binds to a hardware
  requirement, and the platform may satisfy it with an equivalent-or-better node (C-13,
  CAP-7); reserve retroactively or override another instructor's earlier granted
  reservation (earlier wins — CAP-5); change the allowed-model catalog himself if the
  institution assigns that to `admin`.

**Tensions.** His class's certainty vs. a researcher's need for long uninterrupted runs;
reserving generously (safe) vs. reserving tightly (leaves capacity for others); wanting a
named machine vs. the platform's freedom to place him anywhere equivalent.

---

## 3. Student — *"Sara Molina"*  ·  role tier: `student`

**Motivation.** Wants to do coursework: reach an approved model, get a clear limit, and —
when a request is refused — get a reason she can act on instead of a spinner.

**Friction today.** "The GPU is busy" with no distinction between *busy* and *impossible*.
Silent failures. No idea how much of the course budget is left or whether she personally
is near her cap.

**What the platform changes.** Named rejection reasons (CAP-1); a queue position and an
advisory start estimate when hardware is merely busy (CAP-1); a live view of her personal
and course balance returned with every request (CAP-1); a fleet status view without needing
infrastructure access (CAP-9).

**Permission boundary.**
- **Can:** submit interactive inference and single-node training within her context's
  allowed models and both quota balances (CAP-1, CAP-2); check node status (CAP-9); check
  her own and her context's usage (CAP-11).
- **Cannot:** submit distributed or high-VRAM training outside its restricted window
  (CAP-8); create reservations (CAP-5); exceed either the personal or the context balance —
  the request is rejected before it runs (CAP-2); see other students' individual usage.

**Tensions.** Her immediate need vs. the course's shared budget; wanting to run *now* vs.
queueing behind classmates fairly; personal cap vs. context cap (either one exhausted
blocks her, even if the other has room).

---

## 4. Lab Operator / Infrastructure Engineer — *"Tomás Herrera"*  ·  role tier: `operator`

**Motivation.** Keep the fleet healthy and legible. Wants to know machine health,
occupancy, reservations, and failures without SSH-ing into nodes or maintaining a
spreadsheet.

**Friction today.** Manual tracking. Discovers a failing node when a user complains.
Preemptions and node evictions are invisible unless someone files a ticket.

**What the platform changes.** Live per-node health state — `available / reserved / busy /
idle / failing` (CAP-9); node failure handled automatically as relocation, with a
front-of-queue fallback, surfaced as events rather than hidden (CAP-10); a reservation
calendar he can read.

**Permission boundary.**
- **Can:** read full fleet health, occupancy, and the reservation calendar (CAP-9);
  observe relocation and preemption events (CAP-10, CAP-12).
- **Cannot:** override quotas or reservations (that is `admin`); place or move workloads by
  hand — placement and relocation are the platform's, driven by the equivalent-or-better
  predicate (CAP-4, CAP-7, CAP-8); provision, image, or configure the physical machines —
  that is the external infrastructure team's, outside this platform (NG-3, A-11).

**Tensions.** His mental model of "which job is on which box" vs. the platform's design that
a workload binds to a requirement, not a node (C-13); wanting to intervene during an
incident vs. letting reconciliation converge; routine node loss in an uncontrolled lab
(C-12) vs. the instinct to treat every disappearance as an alarm.

---

## 5. Privileged AI Researcher — *"Dr. Nadia Farouk"*  ·  role tier: `researcher`

**Motivation.** Run long-running and distributed training on the high-capacity node, under
controlled conditions, without colliding with a class or losing work to an ambiguous kill.

**Friction today.** Ad-hoc coordination to dodge class schedules. Jobs killed by a class
starting up, recorded no differently from a crash, so restart logic and reporting can't
tell the two apart.

**What the platform changes.** A sanctioned window for distributed / high-VRAM jobs, with
an admin-granted override path for exceptions (CAP-8); preemption-avoidance — the platform
tries to relocate her job to an equivalent-or-better node before preempting it (CAP-6);
preempted jobs recorded as `PREEMPTED`, distinct from `FAILED`, with a grace period to
checkpoint (CAP-6, CAP-8).

**Permission boundary.**
- **Can:** submit distributed and high-VRAM training within its restricted window (CAP-8);
  request an admin override to run outside it (CAP-8, with a CAP-12 audit entry);
  reserve hardware if the institution also grants her the instructor reservation right.
- **Cannot:** run outside the restricted window without an override (CAP-8); take priority
  over an active classroom reservation — academic priority is absolute (C-10); assume her
  job stays on one physical node — it may be relocated mid-run (C-13, CAP-6, CAP-10);
  exceed her research group's context quota (CAP-2).

**Tensions.** Long uninterrupted runs vs. the class-first rule that can preempt her; the
value of a checkpoint-friendly job (survives preemption cleanly) vs. the cost of writing
one; her group's budget vs. the appetite of large training jobs.

---

## Cross-persona tension map

| Tension | Between | Resolved / mediated by |
|---|---|---|
| Class certainty vs. research continuity | Instructor ↔ Researcher | Reservations + exclusive lockout, but relocation-first before preemption (CAP-5, CAP-6, AD-8) |
| Immediate access vs. shared budget | Student ↔ Administrator | Pre-emptive dual-balance quota + named denials (CAP-2, CAP-1) |
| Fair allocation vs. fleet utilisation | Administrator ↔ all users | Queue-vs-reject correctness; substitute offers instead of bare rejections (CAP-1, CAP-5, CAP-7) |
| Manual control vs. automated convergence | Operator ↔ the platform | Desired-state reconciliation; node loss as a routine event (AD-2, AD-9, C-12) |
| Named-machine expectation vs. requirement binding | Instructor / Operator / Researcher ↔ design | SPEC C-13 + equivalent-or-better predicate (CAP-7, AD-4) |
| Authority vs. traceability | Administrator ↔ the audit mandate | Every override and bypass writes an immutable CAP-12 entry (C-8, AD-10) |
