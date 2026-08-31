# Peer-Review Remediation & Traceability Matrix

**Spec-2 Deliverable E** · University AI Compute Management Platform (`friday-compute`)
**Author:** Juan Andrés Cano Ramírez (A00399560)
**Source review:** *Workshop 2 — Peer Evaluation of a High-Level Specification*, reviewer Juan Esteban Ruiz (A00399562), evaluating **Spec-1 — University AI Compute Management Platform** (Cano).
**Spec-2 artifacts referenced:** `_bmad-output/specs/spec-ai-compute-platform/SPEC.md` (B), `_bmad-output/specs/spec-ai-compute-platform/ARCHITECTURE-SPINE.md` (C), companion files under `_bmad-output/specs/spec-ai-compute-platform/companion-files/` (D).

This document traces every one of the eight review dimensions from the finding in Workshop 2, through its root cause in Spec-1, to the concrete action taken in Spec-2 with the exact section or identifier that carries the fix. The two **Weak** and two **Medium** dimensions are remediated. Of the four **Strong** dimensions, three are **preserved** (Scope control, Consistency, Implementation independence) and one is **strengthened** (Verifiability — extended from the Success Signal into a per-capability acceptance clause that Spec-1 did not have); each claim is justified below by comparison to the corresponding Spec-1 mechanism, not merely asserted.

**Quote fidelity.** In every dimension entry below, text shown in quotation marks as **Quoted (Workshop-2 …)** is verbatim from Ruiz's `.docx` (verdict table "Evidence (quote)" column unless noted). Ruiz's rationale and suggested rewrites are summarised, never quoted, and are labelled **Reviewer's rationale / suggested fix (paraphrased)**.

---

## Verdict summary

| Dimension | Workshop-2 rating | Spec-2 disposition | Primary anchor in Spec-2 |
|---|---|---|---|
| Clarity | Medium | Remediated — advisory estimate scoped, exact position added | `SPEC.md` CAP-1 |
| Scope control | Strong | Preserved + tightened (NG-1..NG-11; CAP-7 vs NG-8 boundary) | `SPEC.md` Non-Goals, CAP-7, NG-8 |
| Verifiability | Strong | Strengthened — Spec-1 verifiability lived only in the Success Signal + Contract output shapes; Spec-2 extends it to a per-capability binary/threshold success clause + "Verified by…" procedure on all 12 CAPs (Spec-1's 7 capabilities had none) | `SPEC.md` Capabilities, Success Signal |
| Atomicity | Weak | Remediated — reservation action split into CAP-5 + CAP-6 | `SPEC.md` CAP-5, CAP-6 |
| Consistency | Strong | Preserved — Spec-1 already had (and was rated Strong for) explicit cross-document term discipline; Spec-2 gives that discipline dedicated artifacts (glossary, conventions table, closed enums) so it holds across a larger set, but the spec is not more internally consistent than Spec-1 was | `companion-files/glossary.md`, `ARCHITECTURE-SPINE.md` Consistency Conventions |
| Implementation independence | Strong | Preserved — HOW physically moved to the Spine; no tech in any success clause | `SPEC.md` banner + frontmatter; `ARCHITECTURE-SPINE.md` |
| Traceability | Weak | Remediated — CAP/C/NG/A/OQ/AD identifier scheme, cross-refs, provenance table | `SPEC.md` all sections; provenance table below |
| Decomposition readiness | Medium | Remediated — one trigger + one success clause per CAP; shared CAP-7 mechanism; capability→component map | `SPEC.md` CAP-5/6/7; `ARCHITECTURE-SPINE.md` Capability → Architecture Map |

---

## Traceability matrix (Workshop 2 → Spec 2)

The required four-column mapping. The verbatim quote behind each finding, the full Spec-1
comparison, and the *why* for each disposition are in the **dimension-by-dimension**
section below; each row here is the compressed handle.

| Dimension | Issue / Finding from Workshop 2 | Root Cause in Spec 1 | Action Taken & Specific Section / ID in Spec 2 |
|---|---|---|---|
| **Clarity** | *Medium.* `"if queued, a position and an estimated start time"` (Contract, Submit workload) — how the estimate is computed and what accuracy is expected is not defined. | The Contract named an output field without binding its semantics: no computation basis, no accuracy claim, no promise-vs-hint statement. | `SPEC.md` **CAP-1** success clause: a `QUEUED` outcome now returns an exact integer **queue position** (binary-verifiable against dequeue order) **plus** an explicitly *advisory* estimated start time whose computation basis is stated in the contract; no accuracy figure fabricated. Terms pinned in `companion-files/glossary.md` — "Queue position", "Estimated start time". CAP-1 carries a remediation note. |
| **Scope Control** | *Strong.* `"no internal algorithm for computing hardware equivalence or scheduling heuristics (deliberately deferred, see Contract)"` (Non-Goals). | Not a defect. Spec-1 deferred hardware-equivalence *and* scheduling heuristics as one block — the subject of Ruiz's Open Question. | Preserved and tightened. `SPEC.md` **CAP-7** promotes the equivalent-or-better predicate + deterministic tie-break into the contract; **NG-8** reworded to defer only *further* optimisation (bin-packing, anti-fragmentation, cost/energy, predictive placement). Non-Goals expanded to **NG-1..NG-11**; `ARCHITECTURE-SPINE.md` **Explicit Deferred Items** adds 10 more with reasons. No capability widened. |
| **Verifiability** | *Strong.* `"a workload needing merely-busy (not absent) hardware queues instead of rejecting, in 100% of cases"` (Success Signal) — a full-coverage, quantified, binary guarantee. | Not a defect. The binary "100% of cases" style is the strength; Spec-1's seven capabilities carried no acceptance test of their own. | Strengthened. `SPEC.md` **every CAP-1..CAP-12** carries a binary/threshold `success:` clause **and** an explicit "Verified by…" procedure — verifiable surface Spec-1 did not have. Spec-1's categorical guarantees survive as **KPI-3 / KPI-4 / KPI-6 / KPI-7**. Numbers not traceable to a real source are tagged `[ILLUSTRATIVE DEFAULT]` (**C-6**, **OQ-5**) with replace-with formulas, not asserted as targets. |
| **Atomicity** | *Weak.* `"Create reservation" bundles admission logic (grant / substitute / escalate) with window-start preemption logic (termination signal, grace period, forced termination) in one action.` | One Contract action modelled two behaviours with distinct triggers (user request vs. clock reaching window start) and distinct failure modes (no hardware vs. failure to checkpoint in time), conceived as one lifecycle story. | `SPEC.md` split, exactly per Ruiz's suggested rewrite: **CAP-5 — Classroom Reservation Admission** (admission only; `{GRANTED, SUBSTITUTE_OFFERED, PENDING_ADMIN_REVIEW}`) and **CAP-6 — Reservation Window Enforcement** (window start; relocate-first via CAP-7/CAP-10, preempt only on `NO_QUALIFYING_NODE`; terminal `PREEMPTED ≠ FAILED`, non-resumable by construction). Each has its own success clause and verification. CAP-6 carries a remediation note; the two lifecycles are drawn separately in `companion-files/diagrams/state-transitions.mmd`. |
| **Consistency** | *Strong.* Author's justifications note: `"Checked, after the final pass, that the token-quota timing rule now appears identically in both the Contract and this justifications document."` | Not a defect. Spec-1 already practised single-definition term discipline and was rated Strong for it. | Preserved across a larger artifact set, with dedicated support it previously lacked: `companion-files/glossary.md` (canonical term source, ~50 terms cross-linked to CAP/C/AD); `ARCHITECTURE-SPINE.md` **Consistency Conventions** (id format, RFC3339 timestamps, event names, CAP-1 response envelope fixed once); closed enums defined once — the named-rejection set used *identically* in **CAP-1** and **CAP-2**, the lifecycle-state set in **CAP-8**, the reservation-outcome set in **CAP-5**. |
| **Implementation Independence** | *Strong.* `"the fields, output shapes, and named error conditions above are binding; any transport shown (REST, gRPC, a queue) is illustrative only, not a stack commitment."` (Contract framing note) | Not a defect. Naming the binding-vs-illustrative split out loud is the strength. | Preserved and made physical. All HOW moved to `ARCHITECTURE-SPINE.md`; `SPEC.md` banner + frontmatter declare the WHAT/HOW boundary and mark sources "traceability only". No product name appears in any CAP `success:` clause or in the Success Signal. Kubernetes / Kubeflow appear only in **C-2 / C-3** as *inherited external constraints*; every concrete stack name is quarantined in the Spine's **Technology Stack Decisions** table + ADRs. Authorisation model left open in **C-7**. |
| **Traceability** | *Weak.* `The 7 Capabilities are an unlabeled bullet list; they do not map 1:1 to the Contract's 4 named action blocks (Submit workload, Create reservation, Check node status, Check usage).` | Capabilities authored as descriptive prose; no identifier scheme existed, so nothing downstream could cite a capability except by paraphrase. | Full identifier scheme applied and propagated: **CAP-1..CAP-12**, **C-1..C-13**, **NG-1..NG-11**, **A-1..A-12**, **OQ-1..OQ-6** in `SPEC.md`; **AD-1..AD-13** in `ARCHITECTURE-SPINE.md`. Each CAP `success:` clause cross-references its `C-n` / `NG-n` / sibling `CAP-n`. `ARCHITECTURE-SPINE.md` **Capability → Architecture Map** (every CAP-n → component + governing AD) and every AD names the CAP-n it **Binds**. The **Capability → Spec-1 provenance map** in this document gives the 1:1 mapping, including the CAP-5 / CAP-6 split of "Create reservation". |
| **Decomposition Readiness** | *Medium.* `"Submit workload" decomposes cleanly into one story; "Create reservation" would need to be split first (same root cause as the Atomicity finding).` | The bundled reservation action could not become implementation stories without first being decomposed — a composite capability with two triggers and two failure modes. | The **CAP-5 / CAP-6** split makes each half independently story-able (single trigger, single success clause each). **CAP-7** extracted as one shared mechanism with three callers, bound to a single implementation in `ARCHITECTURE-SPINE.md` **AD-7 / AD-8**. Every CAP now exposes a single trigger + single `success:` clause; closed enumerated outcome sets bound each story's cases; the Spine's **Capability → Architecture Map** assigns every CAP-n to an owning component. |

---

## Response to Ruiz's Open Question (Partner Meeting)

Ruiz's review posed a direct question outside the verdict table: *"does this comparison
[equivalent-or-better] need a testable definition before Spec-2, given that the
reservation-conflict Capability depends on it, or is it safe to leave deferred until the
architecture phase?"*

**Answer: a testable definition, now, not deferred.** CAP-7 fixes the predicate as a
four-part deterministic condition with a fixed tie-break (see SPEC.md CAP-7). Rationale
(recorded in `.memlog.md`, entry 10): leaving it undefined would re-import the same
Clarity/Decomposition-readiness gap Ruiz already flagged elsewhere, since the
reservation-conflict capability cannot be independently story-able while its core
predicate is unspecified.

**On Ruiz's "Notes for the Pair Meeting":** the in-person pair discussion his review
anticipated did not take place before this submission. Remediation proceeded directly
from the written verdict table and quoted evidence in his `.docx`, without a live
follow-up conversation. Recorded here for transparency rather than left silent.

---

## Dimension-by-dimension traceability matrix

### 1. Clarity — *Medium → remediated*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | `"if queued, a position and an estimated start time" (Contract, Submit workload) — how the estimate is computed, and what accuracy is expected, is not defined.` |
| **Root cause in Spec-1** | The Contract named an output field but never bound its semantics: no computation basis, no accuracy claim, no statement of whether the value was a promise or a hint. An underspecified output field in an otherwise binding contract. |
| **Action taken in Spec-2** | `SPEC.md` **CAP-1** now splits the `QUEUED` payload into two distinct things: **(i) queue position** — the exact integer rank among workloads admitted ahead on the same hardware class, *binary-verifiable against actual dequeue order*; **(ii) estimated start time** — explicitly labelled **advisory**, with its computation basis stated in the contract ("projected release times of the workloads ahead on that hardware class") and **no accuracy guarantee claimed**. The CAP-1 verification procedure asserts position correctness against dequeue order. Terms are pinned in `companion-files/glossary.md` ("Queue position", "Estimated start time"). A remediation note in CAP-1 records the change. No `[ILLUSTRATIVE DEFAULT]` accuracy number was invented — the honest move is to scope the field as advisory, not to fabricate a tolerance. |

### 2. Scope control — *Strong → preserved (and tightened)*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | Rating **Strong**. Evidence: `"no internal algorithm for computing hardware equivalence or scheduling heuristics (deliberately deferred, see Contract)" (Non-Goals).` |
| **Root cause in Spec-1** | Not a defect. Spec-1 deferred hardware-equivalence *and* scheduling heuristics as one block — which the Workshop-2 Open Question flagged as the thing that might need to change before Spec-2. |
| **Action taken in Spec-2** | Scope discipline is kept and made finer-grained. **CAP-7** promotes the *equivalent-or-better predicate and its deterministic tie-break* into the contract (they were the part blocking decomposition), while **NG-8** is reworded to still defer only *additional* optimisation on top — bin-packing, anti-fragmentation, cost/energy heuristics, predictive placement. The Non-Goals section grew from a single deferral bullet to **NG-1..NG-11**, each naming a vacuum that downstream would otherwise fill (public cloud, billing, provisioning, authoring UI, SSH, module decomposition, fine-grained authz, IdP, context lifecycle, autoscaling advice). `ARCHITECTURE-SPINE.md` **Explicit Deferred Items** lists 10 more with the reason each is out. Net effect: the scope boundary is narrower and more explicit than Spec-1's, never wider. |

### 3. Verifiability — *Strong → strengthened*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | Rating **Strong**. Evidence: `"a workload needing merely-busy (not absent) hardware queues instead of rejecting, in 100% of cases" (Success Signal).` Ruiz's rationale (paraphrased): a full-coverage, quantified guarantee rather than a qualitative one, turning the criterion into a binary pass/fail test. |
| **Root cause in Spec-1** | Not a defect. The "100% of cases" binary style is the strength to carry forward. |
| **Did Spec-1 have an equivalent mechanism?** | **Partially.** Spec-1's verifiability lived in exactly two regions: the **Success Signal** section (which already used full-coverage quantified phrasing — the line Ruiz quoted) and the **Contract's four action blocks** (which fixed "fields, output shapes, and named error conditions" as binding). Binary acceptance phrasing existed there. It did **not** exist for the capabilities: Spec-1's seven capabilities were prose bullets (Ruiz: "an unlabeled bullet list") with no acceptance test attached to any of them. |
| **Action taken in Spec-2 — why this is *strengthened*, not just preserved** | The binary style is retained *and extended into a region that previously had none*. **Every** capability CAP-1..CAP-12 now carries an explicit `success:` clause phrased as a binary or threshold assertion **with a stated verification procedure** ("Verified by…"). This is a genuine increase in verifiable surface area — 12 capability-level acceptance tests that did not exist in Spec-1 — not a restyling of existing text. The categorical Spec-1 guarantees also survive verbatim as **KPI-3, KPI-4, KPI-6, KPI-7** (100%, categorical), and `SPEC.md` adds six holistic **operational-readiness criteria** and a **KPI detail table**. Numbers not traceable to Spec-1, the brief, or a real datum are tagged `[ILLUSTRATIVE DEFAULT]` with a replace-with formula rather than presented as verifiable targets — protecting the verifiable claims from unearned precision (the anti-pattern corrected in Spec-1's own thresholds). The merely-busy-vs-absent guarantee itself is now CAP-1 + CAP-4 and KPI-4. |

### 4. Atomicity — *Weak → remediated*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | `"Create reservation" bundles admission logic (grant / substitute / escalate) with window-start preemption logic (termination signal, grace period, forced termination) in one action.` |
| **Reviewer's rationale / suggested fix (paraphrased)** | The preemption behaviour has a different trigger (the reservation window starting) and a different failure mode (a job that fails to checkpoint in time) than the admission behaviour in the same action; bundling them makes it easy to lose preemption as an independently testable behaviour during story breakdown. Ruiz's suggested rewrite: split into two Contract actions — "Create reservation" (admission: grant / substitute / escalate to admin review) and "Enforce reservation window" (preemption: termination signal, grace period, forced termination, preempted status) — each with its own success criterion. |
| **Root cause in Spec-1** | One Contract action modelled two behaviours with distinct triggers (a user request vs. a clock reaching the window start) and distinct failure modes (no hardware available vs. a job that fails to checkpoint in time), because the reservation was conceived as a single lifecycle story. |
| **Action taken in Spec-2** | Exactly the suggested split, in `SPEC.md`: **CAP-5 — Classroom Reservation Admission** (admission only; resolves to `{GRANTED, SUBSTITUTE_OFFERED, PENDING_ADMIN_REVIEW}`; explicitly states "window enforcement is CAP-6") and **CAP-6 — Reservation Window Enforcement (relocate, then preempt)** (triggered at window start; relocate-first via CAP-7/CAP-10, and only the `NO_QUALIFYING_NODE` fallback runs termination-signal → grace → forced-termination and records terminal state `PREEMPTED ≠ FAILED`). Each has its own `success:` clause and its own verification. CAP-6 carries a remediation note. The two lifecycles are drawn separately in `companion-files/diagrams/state-transitions.mmd`. A consequence the split made expressible: `PREEMPTED` is now *terminal and non-resumable by construction*, because it is only reached after relocation already failed — closing an open question Spec-1 could not answer while the behaviours were fused. |

### 5. Consistency — *Strong → preserved*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | Rating **Strong**. Evidence: `Justifications: "Checked, after the final pass, that the token-quota timing rule now appears identically in both the Contract and this justifications document."` |
| **Root cause in Spec-1** | Not a defect. The discipline is the strength to carry forward as the document set grows to B + C + four companion files. |
| **Did Spec-1 have an equivalent mechanism?** | **Yes — explicitly.** The Strong rating was awarded *precisely for* demonstrated cross-document term discipline: the evidence Ruiz cited is the author's own justifications note recording a final-pass check that a rule "appears identically" across two documents. Spec-1 already practised single-definition consistency and was credited for it. |
| **Action taken in Spec-2 — why this is *preserved*, not *strengthened*** | Spec-2 gives that discipline **dedicated artifacts** it previously lacked: `companion-files/glossary.md` as a **canonical term source** (~50 terms, each cross-linked to its CAP/C/AD); an `ARCHITECTURE-SPINE.md` **Consistency Conventions** table fixing id format (UUIDv7 opaque strings, per Spec-1's "no internal structure"), timestamp format (RFC3339 UTC, second precision — C-11), event naming (`<entity>.<past-tense-verb>`), and the CAP-1 response envelope once; and closed enums defined once — the **named-rejection set** (five values, used identically in CAP-1 and CAP-2), the workload **lifecycle-state set** (CAP-8), the **reservation-outcome set** (CAP-5) — referenced by name everywhere else including the state diagram. These artifacts **formalize and scale** a property Spec-1 already had and was rated Strong for; they let consistency hold across a B + C + 4-companion set, but the specification is **not more internally consistent than Spec-1 was**. Hence the claim is *preserved (under a larger artifact set)*, not *strengthened*. |

### 6. Implementation independence — *Strong → preserved*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | Rating **Strong**. Evidence: `"any transport shown (REST, gRPC, a queue) is illustrative only, not a stack commitment" (Contract framing note).` Finding #2 quotes the fuller sentence: `"the fields, output shapes, and named error conditions above are binding; any transport shown (REST, gRPC, a queue) is illustrative only, not a stack commitment."` |
| **Root cause in Spec-1** | Not a defect. Naming the binding/illustrative split out loud is the strength to carry forward. |
| **Action taken in Spec-2** | The split is now **physical**, not just stated. All HOW — paradigm, component boundaries, state-mutation rules, technology choices — lives in a separate document, `ARCHITECTURE-SPINE.md`. `SPEC.md`'s banner and frontmatter declare the WHAT/HOW boundary and that source documents are "traceability only". **No technology name appears in any CAP `success:` clause or in the Success Signal.** Kubernetes and Kubeflow appear only in **C-2 / C-3** and are framed as *external constraints the platform inherits and governs through*, not as chosen stack — and even there C-3 explicitly labels the version detail a Spec-2 correction pointing at an ADR. Every concrete stack decision with a product name (Go, PostgreSQL, NATS, controller-runtime, GPU Operator) is quarantined in the Spine's **Technology Stack Decisions** table and its ADRs, each with justification and rejected alternatives. C-7 keeps the authorisation model (RBAC/ABAC/hybrid) explicitly open. |

### 7. Traceability — *Weak → remediated*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | `The 7 Capabilities are an unlabeled bullet list; they do not map 1:1 to the Contract's 4 named action blocks (Submit workload, Create reservation, Check node status, Check usage).` |
| **Reviewer's rationale / suggested fix (paraphrased)** | Without stable identifiers, a later document (architecture spine, test design, an audit) has no formal handle to reference a specific capability — "this test covers Capability X" becomes an informal prose pointer that degrades as the document set grows. Ruiz's suggested rewrite: number each capability (CAP-1 through CAP-7) and reference those IDs from the corresponding Contract action, and from Non-Goals or Constraints where relevant. |
| **Root cause in Spec-1** | Capabilities were authored as descriptive prose. No identifier scheme existed, so nothing downstream could cite a capability except by paraphrase. |
| **Action taken in Spec-2** | A full identifier scheme, applied and propagated: **CAP-1..CAP-12** (stable, never reused), **C-1..C-13**, **NG-1..NG-11**, **A-1..A-12**, **OQ-1..OQ-6** in `SPEC.md`; **AD-1..AD-13** in `ARCHITECTURE-SPINE.md`. Each CAP `success:` clause cross-references the `C-n` / `NG-n` / sibling `CAP-n` it depends on. `ARCHITECTURE-SPINE.md` carries a **Capability → Architecture Map** (every CAP-n → owning component + governing AD) and every AD names the CAP-n it **Binds**. The companion glossary, persona archetypes, and both diagrams reference CAP/C/AD IDs inline. Resolved open questions (OQ-1, OQ-4) are kept in the list with a `RESOLVED` marker and a pointer to the decision of record, as an audit trail rather than a deletion. The **capability-to-Spec-1 provenance table** below gives the 1:1 map the review asked for, including where one Spec-1 action became two capabilities. |

### 8. Decomposition readiness — *Medium → remediated*

| | |
|---|---|
| **Quoted (Workshop-2 verdict table)** | `"Submit workload" decomposes cleanly into one story; "Create reservation" would need to be split first (same root cause as the Atomicity finding).` |
| **Root cause in Spec-1** | The bundled reservation action could not be turned into implementation stories without first being decomposed — a composite capability with two triggers and two failure modes. |
| **Action taken in Spec-2** | The **CAP-5 / CAP-6 split** (see Atomicity) makes each half independently story-able: one trigger, one success clause, one verification each. Beyond the split: **CAP-7** is extracted as a *single shared mechanism* — one deterministic function with three callers (CAP-5, CAP-6, CAP-10) — so relocation/substitution logic is decomposed once instead of re-specified three times, and `ARCHITECTURE-SPINE.md` **AD-7 / AD-8** bind it to a single implementation. Every CAP now exposes a **single trigger and a single `success:` clause** — the decomposition-readiness test. Closed enumerated sets (admission outcomes, rejection reasons, lifecycle states, reservation outcomes, node health states) give each story a bounded set of cases to cover. The Spine's **Capability → Architecture Map** assigns every CAP-n to an owning component, so story allocation has a target before planning starts. |

---

## Capability → Spec-1 provenance map

The 1:1 (and 1:2, where a bundled action was split) map from Spec-1 to Spec-2 capabilities. `SPEC.md` points here for this table.

| Spec-2 capability | Origin in Spec-1 | Nature of the change |
|---|---|---|
| **CAP-1** Workload Admission & Named-Rejection Taxonomy | "Submit workload" action + capability bullet *"returns running, queued with position, pending review, or a specific named rejection reason"* | Kept; `QUEUED` payload clarified (exact position + advisory estimate — Clarity fix); rejection set fixed as a closed five-value enum. |
| **CAP-2** Multi-Tier Quota & Token Metering | Token-quota timing rule (Contract + justifications); multi-tier quota capability bullet | Kept; over-quota now *rejected pre-execution* with the matching CAP-1 reason rather than admitted-then-truncated (stakeholder gap fix). |
| **CAP-3** Role-Based Access & Academic-Context Scoping | Coarse role model + context-level allowed-model catalog | Kept; five role tiers named as a fixed set; fine-grained authz pushed to NG-7, model chosen left open in C-7. |
| **CAP-4** Heterogeneous Hardware Placement & Schedulability | Heterogeneous-placement capability bullet + mixed-fleet constraint | Kept; placement predicate made explicit (GPU count + min per-GPU VRAM); binds to a requirement not a node (C-13). |
| **CAP-5** Classroom Reservation Admission | "Create reservation" action — **admission half** | **Split** from Spec-1's bundled action (Atomicity fix); admission only, three named outcomes. |
| **CAP-6** Reservation Window Enforcement | "Create reservation" action — **window-enforcement half** | **Split** from the same action (Atomicity fix); relocate-first, preempt only as fallback, `PREEMPTED ≠ FAILED` and terminal. |
| **CAP-7** Equivalent-or-Better Node Resolution | The "equivalent-or-better" comparison — *deliberately deferred* in Spec-1's Contract and Non-Goals | **Promoted from deferral to defined capability** (Workshop-2 Open Question); now a four-part iff predicate + deterministic tie-break, shared by CAP-5/6/10. |
| **CAP-8** Distributed / High-VRAM Training Lifecycle | Distributed / high-VRAM training + restricted window + preempted status | Kept; lifecycle state set enumerated and its legal transitions bound to the state diagram; relocation defined as *not* a transition. |
| **CAP-9** Node Health State & Status Visibility | "Check node status" action + node-state list | Kept; five health states fixed; staleness bounded by a tunable SLO (no default invented — OQ-5). |
| **CAP-10** Node-Failure Workload Relocation | Node-health-driven handling of workloads on a bad node | **Reframed** from eviction/re-queue to *relocation-first* (stakeholder redirect); binds to capacity not a named machine; front-of-queue only when no qualifying node (fairness risk logged — A-12, OQ-6). |
| **CAP-11** Usage & Consumption Reporting | "Check usage" action | Kept; totals defined as the sum of reconciled per-workload ledger entries; staleness bounded by a tunable SLO (no default — OQ-5). |
| **CAP-12** Audit Trail for Governance Actions | Audit-trail constraint + capability bullet | Kept; entry shape fixed as `{actor, action, target, timestamp}`; append-only, no update/delete path; every C-8 bypass writes one. |

---

## Preservation check on the Strong dimensions

The rubric asks that remediation not regress what Workshop 2 rated **Strong**. Explicit confirmation, with the per-dimension "strengthened vs preserved" call resolved by comparison to Spec-1:

- **Scope control — preserved (tightened).** No capability was widened; Non-Goals went from one deferral to eleven explicit exclusions plus ten deferred-item rows in the Spine. CAP-7's promotion is offset by NG-8 tightening to defer only *further* optimisation. Spec-1 already had a firm scope boundary — the change is finer granularity within it, not a stronger property.
- **Verifiability — strengthened.** Spec-1's verifiability existed only in the Success Signal and the Contract's output shapes; its seven capabilities carried no acceptance test. Spec-2 adds a binary/threshold `success:` clause and a "Verified by…" procedure to all twelve capabilities — new verifiable surface, not a restyling. Spec-1's "100% of cases" guarantees survive as KPI-3/4/6/7; unearned numbers are tagged, not asserted.
- **Consistency — preserved.** Spec-1 was rated Strong *for* demonstrated cross-document term discipline (its own final-pass consistency check). Spec-2 gives that discipline dedicated artifacts (glossary, conventions table, closed enums) so it holds across a larger set — a formalization of an existing property, not a more-consistent specification.
- **Implementation independence — preserved.** HOW is physically separated into `ARCHITECTURE-SPINE.md`; no product name appears in any success clause or success signal; the binding/illustrative note is retained at the top of `SPEC.md`. Spec-1 already named the binding/illustrative split — Spec-2 enforces it structurally rather than by note alone, but the property is the same.

## Open items acknowledged (not defects — forward-looking)

- **A-12 / OQ-6** — frequent node failure could let CAP-10 front-of-queue relocation starve fresh submissions; fairness bound not resolved at this design stage.
- **OQ-2** — full-cohort concurrent surge is out of scope this iteration (A-2); flagged for a future decision.
- **OQ-3** — source-of-truth ownership for context membership and class schedule, and whether a fallback authoring path is needed (A-3).
- **OQ-5** — concrete values and policy owner for all `[ILLUSTRATIVE DEFAULT]` thresholds.
