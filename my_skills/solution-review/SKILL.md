---
name: solution-review
description: Sanity-check a solution or design decision for reasonableness in its project's context, and analyze the code's quality — two axes, Fit (is this the right solution for this project?) and Quality (is it well built?). Use when the user asks to evaluate, sanity-check, or critique an approach, decision, or applied solution («проверь разумность решения», «оцени качество»), wants a second opinion on a design, or asks whether X is the right way to do it here. For reviewing changes since a commit/branch against standards and spec, use code-review instead.
---

Two-axis judgement of a solution the user points at — something built, partially built, or described as a plan:

- **Fit** — is this the *right* solution for *this* project, given its architecture, constraints, and precedent?
- **Quality** — is the solution *well made*: correct, consistent with the codebase, safe?

Code-review checks a diff against documented standards and a spec; grilling interrogates a plan's author. This skill judges the solution itself on its merits. A solution can pass one axis and fail the other — cleanly-built code solving the problem the wrong way, or the right decision implemented roughly — so report the axes separately and rank findings within each, never across.

## Process

### 1. Pin the subject

Identify what is under review and in what state:

- **Applied** — code exists; the user names files/paths or a feature. Read the code.
- **Proposed** — a design or decision described in conversation or a doc, code not written yet. Fit runs alone; Quality reports NOT APPLICABLE.
- **Ambiguous** — "проверь это решение" with no pointer: ask where it lives before anything else.

Completion criterion: state in one sentence (a) the problem the solution claims to solve and (b) its shape — the files/modules involved, or the plan's outline. If you cannot, ask the user rather than guess.

### 2. Load the project context

Fit is judged against the project, so load the project before the code:

- Instruction files: `AGENTS.md` / `CLAUDE.md` at the repo root and in the touched layer.
- The README / spec / ADR covering the area.
- **Precedent**: search the codebase for how this class of problem is already solved — the closest existing analogue, or the fact that none exists.

Completion criterion: you can name the established pattern for this problem in this repo (or state that none exists), with file references, and list the documented constraints that touch the area.

### 3. Run both axes

If the subject spans several files or modules, dispatch Fit and Quality as parallel sub-agents so each reads with clean eyes (same mechanics as code-review — paste the rubric into the sub-agent prompt, it has no other access to it). For a focused subject, run both passes yourself — as separate passes, Fit first.

Match every rubric item against the subject: each axis's verdict must account for all of its items.

#### Fit rubric — reasonableness

Each item reads *what to check* → *how it fails*:

- **Problem fit** — restate the problem from the issue/request/spec (or inferred from the code). → Fails when the solution solves a neighboring problem, or the problem itself was never stated.
- **Precedent** — how the repo already solves this class of thing. → Fails when it bypasses an established pattern, or adds a second way to do what the repo already does without retiring the first.
- **Right layer** — the layer the project's architecture assigns to this responsibility (batch vs request-path, gateway vs direct storage, schema vs app code). → Cross-layer reach-arounds are Fit findings even when the code is clean.
- **Documented constraints** — walk every constraint/gotcha the instruction files record for this area (migration policy, allow-lists, environment specifics, shared-infra budgets). → Each violated constraint is a blocker.
- **Complexity budget** — the simplest solution the project's patterns allow. → Fails in both directions: machinery with no present caller (over-engineering), and shortcuts where the surrounding code handles variability (under-engineering).
- **Alternatives** — name the realistic alternatives, including "use what's already there" and "do nothing". → The decision is reasonable only if it beats them under this project's constraints; state which alternative you would pick and why.
- **Hidden costs** — maintenance, new dependencies, schema/config churn, security surface, cost on shared/production infra. → Flag costs the decision forces that its author likely didn't price in.

#### Quality rubric — craft

- **Correctness** — solves the stated problem on the happy path *and* the failure paths the surrounding code already handles (timeouts, malformed input, empty results, concurrency). → Missing failure paths and dead logic fail here.
- **Consistency** — naming, error style, module boundaries match the neighboring code. → A foreign idiom dragged in is a finding even when it works.
- **Cohesion** — one logical change lands in few modules, each edited for one reason. → Scattered edits for one purpose, or one module edited for several, fail here.
- **Depth** — interfaces hide their complexity behind one contract. → Shallow pass-through wrappers and interfaces leaking internal steps fail here.
- **Safety** — trace the failure modes: external call fails or returns garbage, input is hostile, same data arrives twice. Check the security basics relevant to the touched surface (injection, SSRF, auth, resource exhaustion). → Enumerate the failure modes the code handles alongside the ones it misses, so the verdict is exhaustive.
- **Tests** — the coverage the repo's test convention expects for this layer. → Missing expected coverage is a finding; where the layer has no tests by convention, note it and move on.
- **Hot-path performance** — only where the code runs per-request / per-row / per-batch. → N+1 queries, unbounded fetches, sync work on a request path fail here. Cold paths: skip.

### 4. Report

Write the report in the user's language. Every finding carries: file:line (or plan section), rubric item, severity, what fails, what to do instead.

Severity ladder:

- **Blocker** — must change before this ships: violated constraint, wrong layer, incorrect behavior.
- **Concern** — a competent reviewer would push back; should change.
- **Nit** — would improve; fine to defer.

Structure:

## Fit
Findings, then the **alternatives line**: the chosen solution vs the realistic alternatives, and why it wins or loses.

## Quality
Findings, or NOT APPLICABLE for a pre-code proposal.

End with a one-line verdict per axis — Fit: SOUND / QUESTIONABLE / UNSOUND; Quality: SOLID / ROUGH / FLAWED — plus totals and the worst finding *within each axis*. Report the worst finding per axis rather than a single cross-axis winner: one axis masking the other is exactly what the separation prevents.
