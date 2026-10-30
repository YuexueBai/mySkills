---
name: code-reuse
description: Apply disciplined code reuse during feature development: evaluate mature libraries first, then reuse existing project code, extract shared abstractions only when multiple real use cases justify them, and keep one-off logic local.
---

# Code Reuse and Dependency Selection

## Goal
Implement features with the least unnecessary duplication and the least unnecessary abstraction. Reuse should improve correctness, consistency, maintainability, or delivery speed—not merely reduce line count.

## Required decision order
For each non-trivial feature or implementation task, follow this order and record the decision briefly when it materially affects architecture or dependencies:

1. **Mature existing library / framework capability**
   - Check the project's declared dependencies and framework capabilities first.
   - If needed, identify a mature, actively maintained, appropriately licensed library that fits the language, runtime, framework, and project constraints.
   - Prefer established ecosystem solutions over custom implementations for well-solved concerns (parsing, validation, serialization, retries, date/time, auth primitives, etc.).
   - Evaluate maintenance activity, release history, compatibility, security posture, license, transitive dependency cost, API stability, bundle/runtime impact, and whether the project already has an equivalent dependency.
   - Do not add a dependency solely because it is popular. Avoid introducing a large dependency for a trivial requirement.
   - Do not claim a library is mature or maintained without evidence. If external lookup is unavailable, state that verification is incomplete.

2. **Existing project implementation**
   - Search the project map (if present), module docs, dependencies, and targeted code search for equivalent or near-equivalent functionality.
   - Inspect candidate implementations and their callers/tests before reuse. Confirm semantics, error handling, side effects, performance, and ownership boundaries.
   - Extend or parameterize existing code only when the new behavior genuinely belongs to the same responsibility and does not break existing callers.
   - Avoid creating a parallel helper/service/component that duplicates an existing capability.

3. **Extract shared project code when justified**
   - Extract a helper, module, service, component, or shared abstraction when at least two real call sites need the same stable behavior, or when a clearly defined cross-cutting policy must be centralized.
   - Prefer evidence from actual call sites over speculative future reuse.
   - Keep the abstraction cohesive: same responsibility, same invariants, and reasonably aligned reasons to change.
   - Define a small, clear API. Keep domain-specific decisions at the appropriate domain boundary; do not turn unrelated workflows into a generic framework.
   - Update callers and tests together. Check that the abstraction does not introduce hidden coupling or excessive indirection.

4. **Keep one-off code local by default**
   - If there is only one real use case, implement it near the caller unless there is a concrete reason for a separate unit (e.g. substantial complexity, independent lifecycle, test seam, security boundary, or clear domain responsibility).
   - Do not create a `common`, `utils`, `helpers`, `base`, or generic framework wrapper for a single trivial use merely to anticipate reuse.
   - Do not force reuse between code paths that only look syntactically similar but have different business meaning or change independently.

## Practical workflow
1. Understand the feature's behavior, constraints, and acceptance criteria.
2. Search targeted project locations for existing dependencies and implementations. Use the project map if available; do not scan the entire repository indiscriminately.
3. Compare candidates against the required behavior, not just matching names.
4. Choose the smallest appropriate reuse level: library capability → existing project code → justified shared abstraction → local implementation.
5. Implement with minimal public surface area and preserve existing conventions.
6. Add or update tests at the appropriate level, including regression tests for reused behavior and compatibility of existing callers.
7. Run focused verification, then the broader checks required by the task's risk and project workflow.
8. Summarize what was reused, what was newly implemented, and why any new dependency or abstraction was justified.

## Anti-patterns to reject
- Reimplementing a solved, non-trivial problem without checking mature libraries.
- Adding a dependency without checking whether the project already provides the capability.
- Copy-pasting existing business logic and allowing two versions to drift.
- Extracting a one-call-site helper solely to make a function shorter.
- Premature generic frameworks, speculative extension points, excessive inheritance, or deeply layered wrappers.
- Merging code solely because it looks similar while semantics or change drivers differ.
- Expanding a shared utility's API with unrelated flags and special cases.
- Reusing code without understanding its side effects, security assumptions, performance, or error semantics.

## Decision note template
For meaningful reuse decisions, include a concise note:
- **Library checked:** existing dependency / candidate / none needed; reason.
- **Project code checked:** relevant path(s) and symbols; reuse or no suitable match.
- **Abstraction decision:** shared extraction or local implementation; number of real call sites and rationale.
- **Trade-offs:** dependency cost, compatibility, performance, or maintenance concerns.

## Completion criteria
- No obvious mature project/library capability was overlooked for a non-trivial solved problem.
- No obvious existing project implementation was duplicated.
- Shared abstractions have multiple real consumers or a concrete architectural justification.
- One-off logic remains local unless separation has a clear benefit.
- Reused behavior is covered by focused tests, and existing callers remain compatible.
- Claims about library quality, reuse, and verification are evidence-based; unknowns are explicit.
