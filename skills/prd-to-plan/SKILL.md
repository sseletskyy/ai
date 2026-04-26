---
name: prd-to-plan
description: Turn a PRD into a multi-phase implementation plan using tracer-bullet vertical slices, saved as a local Markdown file in ./plans/. Use when user wants to break down a PRD, create an implementation plan, plan phases from a PRD, or mentions "tracer bullets".
---

# PRD to Plan

Break a PRD into a phased implementation plan using vertical slices (tracer bullets). Output is a Markdown file in `./plans/`.

## Process

### 1. Confirm the PRD is in context

The PRD should already be in the conversation. If it isn't, ask the user to paste it or point you to the file.

### 2. Explore the codebase

If you have not already explored the codebase, do so to understand:

- Framework, routing convention, and loader/controller pattern
- Auth and role-check pattern (how is the current user obtained, how are roles enforced)
- Database engine and ORM (determines date-grouping syntax, sync vs. async queries)
- Exact table and column names for every table the PRD touches — read schema files, not docs
- Existing service layer conventions (file names, function signatures, return types)
- UI component and styling conventions
- Any existing third-party dependencies relevant to the feature (charting libs, validators, etc.)

### 3. Resolve every uncertainty before slicing

Before drafting phases, resolve all open questions from the PRD so no phase inherits an unanswered question:

- **Schema uncertainties**: if the PRD says "confirm whether table X exists," check right now. Record the finding as a fact, not a question.
- **Conditional decisions**: if the PRD says "add migration if absent," determine absent or present. Convert to a statement.
- **Third-party library**: if the PRD lists options (e.g., "Recharts vs Tremor"), confirm whether any is already installed. If not, note the chosen one.

Embed all confirmed facts in the **Architectural decisions** section of the plan. No phase should need to re-derive what was confirmed in this step.

> Any uncertainty that cannot be resolved from the codebase alone is a decision for the user — surface it before drafting phases.

### 4. Identify durable architectural decisions

From Steps 2 and 3, produce the Architectural decisions block that opens the plan. Include:

- Routes and URL patterns
- Auth and access guard approach
- Confirmed schema (actual table/column names, not logical names from the PRD)
- Key data models and their source tables
- Service layer location and conventions
- Third-party dependencies confirmed or chosen
- Performance constraints (e.g., "all aggregation server-side in loaders")
- Any constants or configuration values shared across phases (e.g., a fee rate, a threshold)
- **Loader pattern** (if applicable): describe how data flows from the server to the UI so all phases follow the same convention without re-discovering it

These are facts every phase can reference without reading any other phase.

### 5. Draft vertical slices

Break the PRD into **tracer bullet** phases. Each phase is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (persistence, service, wiring/loader, UI, tests)
- A completed slice is demoable or verifiable on its own — a real value appears on screen or a real API response is returned
- **Phase 1 must end with the simplest possible end-to-end tracer**: one query → one service function → wired into the loader → rendered in a UI component. This proves the full-stack pattern works and establishes conventions all later phases inherit.
- Prefer many thin slices over few thick ones — a phase implementable in one LLM session is the right size
- Never isolate one layer in its own phase (e.g., "service layer phase," "UI phase") — persistence, service, wiring, presentation, and tests for each pillar belong together in the same phase
- Do NOT include specific file names, function names, or TypeScript signatures — these are implementation details that will be discovered from the real code. DO include durable decisions: route paths, schema shapes, data model names, behavior descriptions
- If a phase would combine items of wildly different scope (e.g., a trivial widget + a substantial new route), split them
</vertical-slice-rules>

### 6. Quiz the user

Present the proposed breakdown as a numbered list. For each phase show:

- **Title**: short descriptive name
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Should any phases be merged or split further?

Iterate until the user approves the breakdown.

### 7. Self-evaluate phase transitions before writing

For each consecutive pair (Phase N → Phase N+1), check:

1. **Vertical slice check**: does every non-QA phase touch all layers (persistence, service, wiring, UI, tests)? A phase with no UI output is a horizontal slice — restructure it.
2. **Prerequisite check**: does Phase N+1 assume anything that Phase N never explicitly produces? If yes, add it to Phase N's acceptance criteria or note it in Phase N+1's Prerequisite field.
3. **Pattern propagation check**: if Phase N establishes a reusable pattern (loader shape, auth guard, component prop convention), will Phase N+1 know to extend it rather than re-create it? If not, add a "Pattern to carry forward" note to Phase N.
4. **Hard gate check**: if Phase N+1 silently depends on a conditional task in Phase N ("add migration if absent"), convert it to an explicit hard gate with a ⛔ marker in Phase N+1's acceptance criteria.
5. **Scope balance check**: are all phases of roughly comparable scope? A phase that is 5× larger than its neighbors should be split.

Fix any gaps found before writing the plan file.

### 8. Write the plan file

Create `./plans/` if it doesn't exist. Write the plan as a Markdown file named after the feature (e.g. `./plans/plan-user-onboarding.md`). Use the template below.

<plan-template>
# Plan: <Feature Name>

> Source PRD: <path or identifier>

## Architectural decisions

Confirmed facts from codebase exploration. Every phase references these — no phase re-derives them.

- **Framework / routing**: ...
- **Auth pattern**: ...
- **Routes**: ...
- **Database**: engine, ORM, date-grouping syntax
- **Confirmed schema**: actual table and column names for every table this feature touches
- **Service layer**: location, conventions
- **Loader pattern**: how data flows from server to UI (established in Phase 1)
- **Third-party deps**: confirmed installed or chosen
- **Performance**: e.g., "all aggregation server-side; no large datasets sent to client"
- **Shared constants**: any fee, threshold, or config value used across phases

---

## Phase 1: <Title — Foundation + First Tracer>

**User stories**: <list>

### What to build

<Narrative description of the end-to-end behavior. Include: the shell or scaffold, the one complete tracer slice (query → service → loader → UI), and what pattern this establishes for subsequent phases. Do not list layer-by-layer tasks — describe observable behavior.>

**Pattern to carry forward**: <Describe the pattern Phase 1 establishes that all subsequent phases must follow — e.g., the loader shape, the service function convention, the component prop pattern.>

### Acceptance criteria

- [ ] <Access guard behavior>
- [ ] <Shell / navigation behavior>
- [ ] <Tracer slice: data visible on screen from live source>
- [ ] <Filter or param behavior>
- [ ] Integration test: service function tested with empty and non-empty data
- [ ] <DB index added if applicable>

---

## Phase 2: <Title>

**User stories**: <list>

**Prerequisite**: <What Phase 1 must have delivered that this phase depends on. Be specific: name the loader, pattern, component, or schema element.>

### What to build

<Narrative. Reference the Pattern from Phase 1 where applicable — e.g., "extend the Phase 1 loader's Promise.all block." No layer-by-layer task list.>

### Acceptance criteria

- [ ] Behavioral criterion 1 — observable in the running app
- [ ] Behavioral criterion 2
- [ ] No regression in Phase 1 components
- [ ] Integration test: service function tested with empty and non-empty cases
- [ ] DB index added if this phase introduces a new table query

---

<!-- For phases that have a schema hard gate: -->

## Phase N: <Title>

**User stories**: <list>

**Prerequisite**: <What prior phases must have delivered.>

⛔ **HARD GATE — resolve before writing any code in this phase:**
<Describe the schema or dependency question that must be answered first. State what to check, what to do if the answer is yes, and what to do if the answer is no. Include the hard gate as the first acceptance criterion below.>

### What to build

<Narrative — written assuming the hard gate is resolved.>

### Acceptance criteria

- [ ] ⛔ HARD GATE: specific condition confirmed before any implementation begins
- [ ] Behavioral criterion 1
- [ ] Behavioral criterion 2
- [ ] Integration test: service function tested with empty and non-empty cases

---

<!-- Repeat Phase block for each phase -->

---

## Out of scope

<Copy verbatim from the PRD's out-of-scope section.>
</plan-template>

## What makes a good acceptance criterion

Acceptance criteria must be **behavioral** — observable states of the running application, not artifacts to be produced:

- GOOD: "Switching course selector updates the enrollment count card with data from the new course"
- GOOD: "Lessons below 40% completion display an amber badge"
- BAD: "Typed interfaces are exported from analyticsService.ts"
- BAD: "Service functions are implemented"

Every phase must include at least one integration test criterion: a service function tested with realistic data covering the empty case and at least one non-empty case.

## Scope guidance

A phase is the right size if a single focused LLM session (Sonnet 4.6) can implement it completely without context degradation. Signs a phase is too large:

- It contains service functions for more than one feature pillar
- It has no UI output (horizontal slice)
- It takes more than ~8 numbered tasks to describe
- Implementing it would require more than one round of "now continuing with..."

When in doubt, split.