# friday-compute

Spec-driven platform for governing access to local LLM inference and training across a shared university GPU fleet — permissions, quotas, reservations, and usage observability.

## Spec Task 2 — deliverable bundle

Extended High-Level Specification & Architecture Spine. Pre-implementation design only — no application code, database migrations, or runtime deployment config (task §6.3). All artifacts in English.

| # | Deliverable | Path |
|---|---|---|
| **A** | Product Brief — vision, proto-personas, value proposition, KPIs | [`design-artifacts/A-Product-Brief/project-brief.md`](design-artifacts/A-Product-Brief/project-brief.md) |
| **B** | SPEC — Why, Capabilities (CAP-1..12), Constraints (C-1..13), Non-Goals (NG-1..11), Success Signal, Assumptions (A-1..12), Open Questions (OQ-1..6) | [`_bmad-output/specs/spec-ai-compute-platform/SPEC.md`](_bmad-output/specs/spec-ai-compute-platform/SPEC.md) |
| **C** | Architecture Spine — Design Paradigm, System Boundaries, State-Mutation Invariants, Technology Stack, ADRs (AD-1..13), Deferred Items | [`_bmad-output/specs/spec-ai-compute-platform/ARCHITECTURE-SPINE.md`](_bmad-output/specs/spec-ai-compute-platform/ARCHITECTURE-SPINE.md) |
| **D** | Companion files — glossary, persona archetypes, C4 system-context diagram, workload/reservation state-transition diagram | [`_bmad-output/specs/spec-ai-compute-platform/companion-files/`](_bmad-output/specs/spec-ai-compute-platform/companion-files/) |
| | ↳ Glossary | [`companion-files/glossary.md`](_bmad-output/specs/spec-ai-compute-platform/companion-files/glossary.md) |
| | ↳ Persona archetypes | [`companion-files/persona-archetypes.md`](_bmad-output/specs/spec-ai-compute-platform/companion-files/persona-archetypes.md) |
| | ↳ System-context diagram (C4 L2) | [`companion-files/diagrams/system-context.mmd`](_bmad-output/specs/spec-ai-compute-platform/companion-files/diagrams/system-context.mmd) |
| | ↳ State-transitions diagram | [`companion-files/diagrams/state-transitions.mmd`](_bmad-output/specs/spec-ai-compute-platform/companion-files/diagrams/state-transitions.mmd) |
| **E** | Peer-Review Remediation & Traceability Matrix — 8-dimension mapping of Workshop 2 findings to Spec-2 actions | [`peer-review-remediation.md`](peer-review-remediation.md) |
| **F** | AI decision / prompt audit trail (append-only) | [`_bmad-output/specs/spec-ai-compute-platform/.memlog.md`](_bmad-output/specs/spec-ai-compute-platform/.memlog.md) |

### Source inputs (traceability only)

- `docs/spec-1-compute-platform.pdf` — Spec-1 (the specification being elevated)
- `docs/Workshop 2 - Ruiz A00399562 - Peer Review of Cano Spec-1.docx` — the peer review remediated in Deliverable E
- `docs/project - university_ai_compute_management_platform_brief.pdf` — original project brief

### Reading order

A → B → C → D, with E and F as the audit layer. `SPEC.md` is the canonical contract (WHAT); `ARCHITECTURE-SPINE.md` is the structural HOW; the `companions:` files are part of the contract.
