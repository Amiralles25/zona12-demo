# Zona12 — AI Development Guidelines

## 1. General principle

Act as an autonomous software-engineering agent.

When the user requests a change, do not immediately start editing files. First understand the request, inspect the existing implementation, and determine the minimum safe set of changes required.

Do not ask the user to choose which AI tool to use. Decide autonomously whether Graphify, Codex, or neither is appropriate.

Prefer reusing existing code, components, hooks, utilities, API endpoints, database structures, and established project patterns over creating new implementations.

Avoid duplicate functionality and unnecessary abstractions.

Keep changes focused and do not modify unrelated files.

---

## 2. Graphify

This project has a knowledge graph at `graphify-out/` with god nodes, community structure, and cross-file relationships.

Use Graphify autonomously when understanding the existing architecture or relationships between files would materially improve the task.

### When to use Graphify

Use Graphify for:

* Changes involving multiple files or modules.
* New features that interact with existing functionality.
* Questions about architecture, dependencies, or relationships.
* Refactors.
* Authentication, authorization, checkout, orders, products, users, database, or other cross-cutting functionality.
* Situations where existing functionality may already solve part of the requested problem.
* Before introducing a new component, hook, API endpoint, service, or utility when equivalent functionality might already exist.

Do not use Graphify for trivial, isolated changes where the relevant file and implementation are already obvious.

### Graphify commands

When `graphify-out/graph.json` exists:

* For codebase questions, first run:
  `graphify query "<question>"`
* For relationships between specific elements:
  `graphify path "<A>" "<B>"`
* For focused concepts:
  `graphify explain "<concept>"`

These commands return scoped subgraphs and should be preferred over broad source browsing.

If `graphify-out/wiki/index.md` exists, use it for broad navigation instead of unnecessarily browsing the entire source tree.

Read `graphify-out/GRAPH_REPORT.md` only for broad architecture reviews or when `query`, `path`, or `explain` do not provide enough context.

After modifying code, run:

`graphify update .`

This keeps the knowledge graph current. Graphify updates are AST-only and do not require an API call.

---

## 3. Planning before implementation

Before modifying code, classify the task by complexity and risk.

### Low-risk tasks

Examples:

* Text changes.
* Small styling changes.
* Clearly isolated UI changes.
* Simple, localized bug fixes.

For these tasks, inspect the relevant code and implement directly without unnecessary planning or Codex review.

### Medium-risk tasks

Examples:

* Changes spanning several files.
* New frontend functionality.
* New API endpoints.
* Changes involving existing hooks, contexts, or shared components.
* Changes where existing functionality may be reusable.

For these tasks:

1. Understand the relevant architecture.
2. Use Graphify when useful.
3. Search for existing implementations and reusable code.
4. Create a concise implementation plan internally.
5. Implement the minimum required changes.

Use Codex review when there is meaningful architectural uncertainty or risk of introducing duplication/regressions.

### High-risk tasks

Examples:

* Authentication or authorization.
* Password recovery.
* Checkout or order processing.
* Database schema or migrations.
* Security-sensitive functionality.
* Changes affecting multiple backend modules.
* Large refactors.
* Changes crossing frontend, backend, and database boundaries.
* Changes where an incorrect implementation could cause significant regressions.

For these tasks:

1. Inspect the existing architecture.
2. Use Graphify to understand relevant relationships.
3. Identify existing functionality that can be reused.
4. Prepare a clear implementation plan.
5. Ask Codex to review the plan before implementation.
6. If Codex identifies issues, correct the plan before proceeding.
7. Implement only after the plan is sufficiently validated.
8. Review the resulting changes with Codex when appropriate.
9. Run relevant tests/checks.
10. Update Graphify after modifications.

The user should not need to explicitly request Codex review.

---

## 4. Codex collaboration

Codex is available through MCP.

Use Codex autonomously when an independent review would materially improve correctness, architecture, security, or maintainability.

Do not invoke Codex for trivial changes simply for the sake of using another model. Minimize unnecessary model calls and token usage.

### Plan review

For high-risk changes, use Codex in read-only mode to review the implementation plan.

The review should focus on:

* Correctness.
* Completeness.
* Architectural consistency.
* Existing functionality that should be reused.
* Duplicate implementations.
* Security issues.
* Edge cases.
* Unnecessary complexity.
* Potential regressions.

Ask Codex to respond concisely and return either:

`APPROVED`

or:

`CHANGES NEEDED`

followed by specific actionable issues.

Do not ask Codex to modify files during plan review.

### Implementation review

After significant implementation, use Codex to review the actual changes when appropriate.

The review should focus on:

* Correctness.
* Regressions.
* Duplicate code.
* Security.
* Consistency with existing project patterns.
* Missing validation/error handling.
* Unrelated modifications.
* Tests that should be added or updated.

Prefer concise review output.

Maintain the same Codex session/thread when continuing a review instead of starting unnecessary new sessions.

Limit review iterations. Do not repeatedly ask agents to review the same change without a concrete reason.

---

## 5. Token efficiency

Optimize for useful work rather than maximum agent activity.

Rules:

* Do not call Graphify when its information would not materially help.
* Do not call Codex for trivial changes.
* Do not repeatedly inspect the same files when the required context is already known.
* Prefer scoped Graphify queries over large reports.
* Prefer concise Codex responses.
* Do not request or reproduce unnecessary reasoning.
* Reuse existing Codex sessions when continuing a review.
* Avoid repeated planning/review cycles once the implementation is sufficiently validated.

The goal is:

`minimum necessary analysis → safe implementation → appropriate verification`

not:

`maximum number of tools/agents used`.

---

## 6. Existing code first

Before creating new functionality, look for existing implementations.

Check for:

* Existing components.
* Existing hooks.
* Existing contexts.
* Existing API endpoints.
* Existing database queries.
* Existing validation schemas.
* Existing authentication middleware.
* Existing utilities.
* Existing types/interfaces.
* Existing styling patterns.

If suitable functionality already exists, extend or reuse it instead of creating a duplicate.

Do not create a second implementation of functionality that already exists unless there is a clear architectural reason.

---

## 7. Scope control

Only modify files necessary to complete the requested task.

Do not:

* Perform unrelated refactors.
* Rename unrelated files.
* Reformat large sections unnecessarily.
* Change dependencies without justification.
* Modify configuration unrelated to the task.
* Rewrite working code merely for stylistic preference.

Preserve existing project conventions.

---

## 8. Verification

After implementation:

1. Review the changes.
2. Run relevant tests, type checks, linting, or build commands when available and appropriate.
3. Check for unintended modifications.
4. Run `graphify update .` after code changes.
5. Summarize what changed and any relevant verification results concisely.

For significant changes, use Codex for an independent review when appropriate.

---

## 9. Communication style

Be concise.

Do not expose internal chain-of-thought or hidden reasoning.

Before implementation, briefly communicate the intended approach when the task is non-trivial.

For simple tasks, avoid unnecessary explanations.

When a task is complete, report:

* What changed.
* Important files affected.
* Verification performed.
* Any remaining issue or limitation.

Do not overwhelm the user with internal tool activity unless it is relevant to the result.

---

## 10. Visual validation

Do NOT use the Chrome/browser skill for routine visual reviews of the frontend.

The user will perform visual/UI reviews manually in the browser. Do not automatically open Chrome, navigate the application, take screenshots, or perform browser-based visual inspection merely to check whether a design change looks good.

For frontend changes:

* Implement the requested design accurately based on the existing code and the visual references in `diseño/`.
* Use static analysis, TypeScript checks, linting, tests, and builds when relevant.
* Do not spend tokens or tool calls performing visual browser validation that the user can perform themselves.
* If a visual detail cannot be reliably verified from the code, mention it to the user rather than automatically opening the browser.
* Do not use browser automation simply to confirm that a visual change "looks good".

Browser/Chrome tools may still be used when they are genuinely necessary for functional testing, debugging browser-specific behavior, testing user interactions, or verifying behavior that cannot reasonably be checked through static analysis.

When browser testing would be useful but is not strictly necessary, prefer asking the user rather than automatically performing it.

The user is responsible for the final visual approval of the frontend.

--

## 11. Design system

The `diseño/` directory contains the visual reference for the Zona12 redesign.

When modifying frontend UI:
- Treat `diseño/` as the primary visual reference.
- Maintain a consistent design system across all pages.
- Reuse existing components before creating new ones.
- Avoid introducing arbitrary colors, spacing, shadows, or typography that conflict with the established visual language.
- Keep desktop and mobile layouts coherent.
- Do not modify backend functionality when the task is purely visual.

---

## 12. Package Manager

This project uses **pnpm exclusively**.

Rules:

* Do not use `npm` under any circumstances.
* Always use `pnpm` to install, remove, update, or execute packages and scripts.
* Never create, modify, or use `package-lock.json`.
* Always use `pnpm-lock.yaml` as the project's lockfile.
* If a tool, documentation, error message, or generated instruction recommends using `npm`, automatically use the equivalent `pnpm` command instead.
* Do not switch package managers unless the user explicitly instructs you to do so.
* This is a mandatory project rule and must be followed even when using `npm` may appear more convenient.
* Avoid introducing unnecessary dependencies or modifying dependency versions unless it is necessary for the task.
