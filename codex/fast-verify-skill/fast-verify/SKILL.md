---
name: fast-verify
description: Implement features efficiently by batching low-value tests during coding, running one focused self-check after implementation, then independently verifying requirements and edge cases from a clean context. Use for coding tasks where repeated tiny test cycles slow progress; do not use to bypass required safety, security, data-integrity, or release checks.
---

# Fast Verify: batched implementation + independent acceptance

## Goal

Reduce wasted iteration caused by running tiny, repetitive tests after every small edit, while preserving meaningful verification. Optimize for fast progress with evidence, not for the fewest test commands. A passing smoke test is not proof that the feature meets its requirements.

## Operating rules

1. **Understand the target first.** Extract the user-visible goal, acceptance criteria, constraints, relevant interfaces, and likely failure modes before editing. If a requirement is materially ambiguous, ask one concise question; otherwise state a reasonable assumption and proceed.
2. **Inspect before changing.** Read the relevant implementation, callers, tests, project instructions (`AGENTS.md` and equivalent), and available test/build commands. Keep changes scoped.
3. **Batch low-value checks, never blindly skip verification.** During a coherent implementation pass, do not repeatedly run the same full test suite after every trivial edit. Prefer reading, reasoning, and making a small group of related edits, then validate at meaningful checkpoints.
4. **Keep cheap feedback cheap.** Use targeted tests or static checks when they are quick and likely to catch a concrete class of errors. Do not suppress compiler/type/lint errors, ignore failing tests, or claim a check passed without running it.
5. **Preserve a verification trail.** Record which commands ran, their outcomes, and what was not checked. Do not describe unrun tests as passing.
6. **Do not overclaim “bug-free.”** Report evidence, remaining uncertainty, and known limitations honestly.

## Phase A — Plan and implement in the working context

1. Read repository guidance and identify the project’s established commands and conventions.
2. Summarize the task in 1–5 acceptance criteria. Include at least:
   - expected behavior;
   - inputs and outputs;
   - error/edge behavior;
   - compatibility or regression constraints, where relevant.
3. Identify the smallest sensible verification set:
   - one targeted test or reproducible scenario for the new behavior;
   - existing relevant regression tests;
   - build/type/lint checks only where the project supports them and they are useful.
4. Implement the feature in a coherent pass. Avoid running the same broad suite after every small edit. Run a targeted check earlier only if it cheaply resolves a significant uncertainty or prevents expensive rework.
5. Before finishing implementation, inspect the diff for accidental changes, dead code, missing error handling, secret leakage, unsafe assumptions, and unintended API/schema changes.
6. Run the self-check once the implementation is coherent. Fix failures that are attributable to the change. If a check fails for environmental or pre-existing reasons, establish that evidence and report it instead of hiding it.

## Phase B — Self-check and handoff packet

Prepare a concise handoff packet for a separate reviewer/agent. Include:
- original task and acceptance criteria;
- changed files and the purpose of each change;
- key design choices and assumptions;
- commands actually run and exact outcomes;
- tests not run and why;
- known risks, tricky paths, and relevant entry points;
- any setup instructions needed to reproduce the behavior.

Do not coach the reviewer toward a “pass.” Do not include a self-congratulatory conclusion. The reviewer should be able to reach an independent judgment.

## Phase C — Independent acceptance in a fresh context

Use a separate agent or a genuinely fresh conversation with access to the same working tree/diff. Give it the handoff packet plus the task/acceptance criteria. Tell it to independently inspect the implementation and derive its own validation plan. If a subagent is unavailable, provide the review prompt for a separate session; do not pretend a separate review occurred.

The reviewer must:

1. Re-read the task and relevant project guidance; do not rely only on the implementer’s summary.
2. Inspect the actual diff and surrounding code.
3. Derive at least one independent way to falsify the intended behavior, not merely repeat the implementer’s happy-path test.
4. Check boundary cases, invalid inputs, error paths, state transitions, concurrency/resource cleanup, security/privacy, and compatibility where relevant to the change. Skip irrelevant categories with a short reason.
5. Run the smallest meaningful independent validation set. Prefer deterministic tests and existing project tooling. Do not add tests just to create volume; add tests that prove an acceptance criterion or guard a credible regression.
6. Report findings by severity, with file/line references when possible. Distinguish:
   - confirmed defect;
   - plausible risk needing follow-up;
   - test gap;
   - style/preference.
7. Do not modify implementation while reviewing unless explicitly asked to fix issues. If a fix is requested, make the smallest targeted change and re-run the relevant verification.

## Phase D — Resolve findings and final status

1. If the reviewer finds a concrete defect, fix it in the implementation context and run the narrowest relevant test, plus any necessary regression checks.
2. Re-run the independent review on the changed area when the fix affects behavior, interfaces, or error handling.
3. Stop when acceptance criteria have evidence, no unresolved high-severity findings remain, and remaining gaps are clearly stated. Do not loop indefinitely over speculative style concerns.
4. Final report must include:
   - acceptance criteria and status;
   - commands run and results;
   - independent review result (or explicitly “not performed”);
   - unresolved risks and untested areas;
   - changed files.

## Test batching policy

Use judgment rather than a fixed number of test runs.

- **Usually batch:** repeated full-suite runs after minor edits, redundant manual checks, and tests that exercise unchanged behavior without adding new evidence.
- **Usually run promptly:** a test that disambiguates an important design choice, a cheap unit test for a fragile algorithm, a compiler/type check that quickly catches structural mistakes, or a security/data-loss-sensitive check.
- **Never skip solely for speed:** tests required by repository instructions or CI/release policy; migration/data-integrity checks; authentication/authorization checks for security-sensitive changes; destructive-operation safeguards; or tests necessary to establish a core acceptance criterion.
- **When tests are absent:** create a small reproducible validation when feasible. If not feasible, explain what evidence was used and what remains unverified.
- **When the environment blocks tests:** state the exact blocker and do not convert it into a pass.

## Suggested prompt for the independent reviewer

> You are the independent acceptance reviewer. Treat the task description and repository acceptance criteria as authoritative; treat the implementation summary as untrusted orientation only. Re-read project instructions, inspect the actual diff and relevant surrounding code, and derive your own validation plan. Try to falsify the intended behavior, including meaningful edge/error paths and regressions. Run appropriate existing checks or a minimal targeted test; do not invent passing results. Do not change files. Report findings with severity and file/line references, then list commands run, results, untested areas, and whether each acceptance criterion is supported by evidence. If you find no issue, say “No defect found in the checks performed,” not “bug-free.”

## Invocation examples

- “Use `$fast-verify` to implement this feature with batched testing and independent acceptance review.”
- “Use `$fast-verify` on the current diff; do not run repetitive full-suite tests during every small edit.”
- “Use `$fast-verify` to review this change from a fresh context and try to falsify the acceptance criteria.”

## Completion checklist

- [ ] Acceptance criteria were written before implementation.
- [ ] Repository guidance and relevant code were inspected.
- [ ] Repetitive low-value test cycles were batched, not all verification skipped.
- [ ] Self-check commands and outcomes are accurate.
- [ ] A separate reviewer actually inspected the diff, or review is marked not performed.
- [ ] Reviewer attempted to falsify the happy path and considered relevant edge/error cases.
- [ ] Findings were resolved or disclosed.
- [ ] Final status does not claim zero bugs without evidence.
