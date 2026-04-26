This skill converts a PRD file into a multi-phase implementation plan using a tracer bullet (vertical slice) approach, then self-evaluates every phase transition for agent-handoff completeness and produces an improved final version. Use when the user wants to create a plan from a PRD, or wants to turn a feature spec into an actionable phased plan.

## Step 1 — Read the PRD

Read the PRD file the user points to (usually in `plans/`). Identify:
- The **functional pillars** (main feature areas) — each becomes a phase.
- The **system layers** present in this project (e.g., persistence, service/logic, API/wiring, UI, tests). These are the layers every phase must cross.
- External dependencies (data models, third-party services, framework conventions) that must be resolved before implementation begins.
- Any conditional decisions ("if X exists, else add it") — these are risk items.

## Step 2 — Write the Initial Plan (Vertical Slices)

Save as `plans/plan-<name>.md`.

**Phase structure — tracer bullet order:**
- **Phase 1 — Audit + Shell + First Tracer:** Codebase discovery, application shell, and ONE complete end-to-end slice for the simplest pillar. The slice must touch all system layers (e.g., one query → one service function → wired into a loader or controller → rendered in a UI component or output). This proves the full stack works, establishes all patterns, and produces something testable before any other phase begins.
- **Phases 2–N — One vertical slice per remaining pillar:** Each phase implements one complete functional pillar across all system layers. Never isolate one layer in its own phase (e.g., a "service layer phase" or a "UI phase") — persistence, service, wiring, presentation, and tests for each pillar belong together.
- **Final phase — QA/Polish:** Performance, accessibility, code health, and verification of all user stories from the PRD.

**Each phase (except QA) must contain all of the following:**
1. All-layer tasks: persistence (query, migration, index), service function with exported typed interface, wiring (loader/controller/route), presentation (UI component, CLI output, or API response), and tests.
2. **Goal**, **Tasks** (numbered), **Exit criteria** that names specific artifacts — not just "it works."
3. A **Feedback Opportunity** block at the end: 2–4 questions stakeholders can answer at this point, and which decisions are still open vs. locked in for the next phase.

## Step 3 — Evaluate Every Phase Transition

For each consecutive pair (Phase N → Phase N+1), ask:

1. **Vertical slice gap**: Does every non-audit, non-QA phase actually touch all system layers identified in Step 1? A phase that covers only one layer (e.g., only service functions, only UI) cannot be tested end-to-end. Expand or merge the phase until it is a complete vertical slice.

2. **Foundation gap**: Does Phase N+1 assume knowledge (e.g., data model field names, API contracts, framework conventions, third-party configuration) that Phase N never explicitly produced? If yes, add an audit/discovery task to Phase N.

3. **Contract gap**: If Phase N implements service functions or APIs, does it export the typed interfaces (types, schemas, API shapes) that Phase N+1 consumers depend on? If not, add explicit interface definitions to Phase N's exit criteria.

4. **Pattern gap**: If Phase N establishes an architectural or structural pattern (e.g., how a module is wired, how errors propagate, how configuration flows), does it document that pattern for Phase N+1 to follow? If not, add a documentation task.

5. **Hard gate gap**: Are there conditional tasks in Phase N ("add X if absent") that Phase N+1 silently depends on? Convert these to hard gates with explicit "do not mark complete until resolved" language.

6. **Reusability gap**: If a later phase needs to reuse a module from Phase N in a broader or different context, does Phase N build it without hard-coded scope assumptions? If not, add a reusability constraint.

7. **Test infrastructure gap**: If Phase N produces test fixtures or test data that a later phase needs (e.g., for performance testing), does it document the fixture shape and location?

## Step 4 — Write the Improved Plan

Save as `plans/plan-<name>-v2.md`. Apply all findings from Step 3:
- Add sub-phases (e.g., `1a — Audit`, `1b — First Tracer`) where discovery must precede implementation.
- Add an **Appendix: Audit Findings** section — a table for facts discovered in Phase 1 (field names, conventions, third-party versions, layer inventory). Subsequent phases reference it; they never re-derive these facts.
- Strengthen exit criteria to name the specific artifacts (typed interfaces, documented patterns, resolved dependencies) the next phase needs.
- Add a prerequisite note at the top of each phase listing what Phase N-1 must have delivered.

## Conventions

- Each phase must be independently implementable by an AI agent starting cold — assume no shared memory between phases.
- Never leave external dependency questions open past Phase 1. The audit sub-phase must resolve them.
- No isolated layer phases after Phase 1 — all layers for each pillar belong in the same phase.
- "Out of Scope" section must be preserved verbatim from the PRD.
- Save both the initial plan and the v2 plan — the v2 is the deliverable.