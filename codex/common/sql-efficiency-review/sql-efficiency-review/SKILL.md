---
name: sql-efficiency-review
description: Audit and optimize SQL, schema design, ORM queries, indexes, and data-access paths in an existing codebase—especially when the schema was inferred from requirements rather than explicitly designed. Use for database performance reviews, query optimization, N+1 detection, index design, and independent-agent SQL audits.
---

# SQL Efficiency Review

## Mission

Find and fix material database inefficiencies in the actual project, especially when schema and SQL were generated directly from application requirements without an explicit data model. Review the whole data-access path—not only the visible SQL string—and prove that proposed changes preserve behavior.

The objective is not “add indexes everywhere” or “make SQL look clever.” Optimize measured or well-supported bottlenecks while preserving correctness, maintainability, security, and data integrity.

## Non-negotiable rules

1. **Inspect before proposing.** Identify database engine/version if possible, ORM/query builder, schema migrations, model definitions, data-access code, call sites, and existing indexes/constraints.
2. **Do not guess the schema.** Treat migrations and live schema metadata (if safely available) as stronger evidence than ORM model assumptions. If schema/data volume is unavailable, label conclusions as hypotheses.
3. **Never run risky SQL against production.** Do not execute destructive DDL/DML, expensive full-table scans, or `EXPLAIN ANALYZE` on production without explicit authorization and an approved safe plan. Prefer read-only inspection, staging, representative fixtures, or a restored snapshot.
4. **No blind index creation.** Every index proposal must state the query it supports, predicate/join/order columns, selectivity assumptions, write/storage cost, and whether an existing index already covers it.
5. **Preserve semantics.** Before rewriting a query, capture its intended results, ordering, null behavior, pagination, transaction/isolation assumptions, and authorization filters. Faster but semantically different SQL is a defect.
6. **Parameterize values.** Never introduce SQL injection by string-concatenating user-controlled values. Identifiers that cannot be parameterized must be allow-listed.
7. **Evidence over claims.** Do not claim a query is faster unless supported by a reproducible plan or benchmark. Distinguish measured improvement from reasoned expectation.
8. **Avoid scope creep.** Prefer small, reviewable changes. Do not redesign the whole schema unless the evidence shows a structural issue and the user authorizes the scope.

## Phase 1 — Inventory SQL and data paths

Search the repository for:
- Raw SQL (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, CTEs, subqueries);
- ORM query construction, eager/lazy relationships, serializers that trigger database access;
- repositories/DAO/service functions and their callers;
- model definitions, schema migrations, indexes, unique constraints, foreign keys;
- pagination, sorting, filtering, aggregation, batch jobs, scheduled tasks;
- transaction boundaries and repeated reads/writes;
- database configuration, connection pools, timeout and query logging settings;
- tests and fixtures related to database behavior.

Build a compact inventory of high-impact query paths. Prioritize:
1. queries called frequently or in loops;
2. list/detail endpoints with joins or relationship loading;
3. unbounded reads, exports, reports, dashboards, and background jobs;
4. filtering/sorting/pagination on large or growing tables;
5. duplicate queries and N+1 patterns;
6. write-heavy paths with expensive indexes or unnecessary updates;
7. lock contention, long transactions, and connection-pool pressure.

Do not dump every SQL statement into the final report. Group similar query shapes and identify the highest-risk paths first.

## Phase 2 — Infer the intended data model

When no explicit schema design exists, reconstruct it from evidence:
- business entities and lifecycle/state transitions;
- primary keys, foreign keys, uniqueness and cardinality;
- nullable versus required fields;
- ownership/tenant boundaries and authorization constraints;
- expected read/write ratios and likely row counts;
- common filters, joins, sort orders, pagination patterns;
- retention, archival, and deletion behavior.

Create a concise inferred schema map. Mark uncertain assumptions. Do not change table structure solely because an inferred design “looks cleaner.”

If a schema change is proposed, specify:
- current schema and problem;
- proposed schema/index/constraint;
- migration and rollback strategy;
- compatibility with old and new application versions;
- backfill, locking, downtime, and data-integrity risks;
- verification query or test.

## Phase 3 — Review query efficiency

For each important query shape, inspect applicable issues:

### Access and selectivity
- Is `SELECT *` retrieving unnecessary columns on a hot or wide-table path?
- Are filters selective and applied before expensive joins/aggregations where appropriate?
- Are expressions/functions/casts on filtered columns preventing useful index use?
- Are implicit type conversions or collation mismatches affecting plans?
- Are `OR`, wildcard searches, JSON fields, array fields, or computed predicates requiring a different strategy?
- Is the query fetching an unbounded number of rows?

### Joins and relationship loading
- Are join keys compatible in type and indexed where useful?
- Can one-to-many joins multiply rows unexpectedly?
- Does ORM serialization trigger N+1 queries?
- Would eager loading, a join, a batch query, or a two-step fetch be more appropriate?
- Are correlated subqueries repeated per row when a join/aggregation would be better, or is a rewrite actually likely to worsen the plan?

### Pagination and ordering
- Does pagination have deterministic ordering, including a tie-breaker?
- Is deep `OFFSET` pagination a demonstrated bottleneck where keyset/cursor pagination is suitable?
- Does the index align with common equality predicates followed by range/order columns?
- Are count queries or total-page calculations disproportionately expensive?

### Aggregation and duplication
- Can repeated identical reads be safely batched or cached with clear invalidation?
- Are repeated writes unnecessary?
- Are `DISTINCT`, grouping, sorting, or subqueries masking an avoidable join explosion?
- Is an aggregation being recomputed on every request when a maintained summary is justified by measured workload?

### Indexes and constraints
- Check existing indexes before recommending new ones.
- Consider composite index column order based on real predicates, selectivity, range conditions, and sort order; do not apply a universal “most selective first” rule without considering the engine and query shape.
- Check whether foreign keys, uniqueness requirements, and join columns have appropriate indexes for the specific database engine.
- Consider covering indexes only when they materially help and storage/write overhead is acceptable.
- Flag redundant, duplicate-prefix, unused, or excessively wide indexes only when usage evidence is available.
- Do not recommend indexes on every column.

### Writes, transactions, and concurrency
- Avoid row-by-row writes when a safe bulk operation is appropriate.
- Check unnecessary read-before-write patterns, repeated updates, and overly broad updates.
- Preserve transaction atomicity and isolation expectations.
- Look for long-running transactions, lock escalation/contention, deadlock patterns, and unsafe retry behavior.
- For concurrent workers, verify uniqueness/idempotency and race-condition behavior rather than relying on a prior existence check alone.

### Correctness and security
- Preserve tenant/user authorization predicates in every rewrite.
- Check NULL semantics, timezone/date boundaries, numeric precision, collation, duplicate rows, ordering guarantees, and pagination stability.
- Check injection risks and parameter binding.
- Ensure query optimizations do not leak data across tenants or bypass soft-delete/state filters.

## Phase 4 — Validate with database-specific evidence

Use the actual database engine and version where available. Do not transplant PostgreSQL advice to MySQL, SQLite, SQL Server, or another engine without checking differences.

Preferred evidence order:
1. existing production/staging query metrics or slow-query logs, with sensitive data removed;
2. actual execution plan on a safe representative environment;
3. representative benchmark using realistic row counts and data distribution;
4. deterministic unit/integration tests for result equivalence;
5. static reasoning, explicitly labelled as such.

Where supported, use `EXPLAIN` to inspect scan types, join order/algorithm, estimated rows, actual rows, sort/hash operations, and index usage. Use `EXPLAIN ANALYZE` only when executing the statement is safe and authorized; it can actually run the query and add overhead. Compare before/after plans and timings under comparable conditions. Do not rely on a single noisy timing sample.

If no database or realistic data is available:
- do not invent performance numbers;
- produce a prioritized hypothesis list;
- add or propose focused regression tests;
- state exactly what would need to be measured before shipping.

## Phase 5 — Optimize in priority order

Rank findings by expected impact, confidence, and risk:
- P0: correctness/security/data-loss issue or severe production risk;
- P1: likely major hot-path bottleneck or unbounded workload;
- P2: meaningful inefficiency with moderate impact;
- P3: low-impact cleanup or speculative improvement.

For each finding include:
- location (file, function, query shape; line numbers if available);
- evidence and likely workload impact;
- root cause;
- recommended change;
- risks and trade-offs;
- validation plan;
- confidence: high / medium / low.

Prefer the smallest effective intervention:
1. remove accidental repeated queries/N+1;
2. limit rows and columns appropriately;
3. correct join/filter/pagination shape;
4. add or adjust a justified index;
5. batch writes or shorten transaction scope;
6. consider caching, denormalization, partitioning, or schema redesign only when simpler options are insufficient and workload evidence supports them.

Do not implement speculative high-risk schema changes without approval. For high-impact changes, present the recommendation and migration plan before applying it.

## Phase 6 — Independent review using another agent or clean session

Prepare a review packet containing:
- user-visible requirements and result semantics;
- database engine/version and ORM;
- relevant schema/migrations/indexes;
- exact query or query shape and call path;
- observed evidence (plans, row counts, timings) and environment limits;
- proposed or implemented changes;
- commands/tests run and outcomes;
- known assumptions and unresolved risks.

Ask the independent reviewer to re-read the relevant code and schema, derive its own diagnosis, and try to disprove the optimization. It must specifically check:
- result equivalence and ordering/null/pagination semantics;
- index usefulness and write/storage costs;
- whether an existing index already serves the query;
- realistic data distribution and plan stability;
- N+1, row multiplication, locking, and concurrency;
- tenant/authorization predicates and injection risk;
- migration rollout, rollback, and mixed-version compatibility.

The reviewer must not merely approve the implementer’s explanation. It should report confirmed defects, likely risks, test gaps, and optional style concerns separately. Do not claim independent review occurred unless another agent/session actually performed it.

## Phase 7 — Final report

Use this format:

### Executive summary
- Highest-impact findings and overall risk.

### Findings
For each finding:
- Severity / confidence
- File and query path
- Current behavior and evidence
- Proposed or applied optimization
- Expected impact (measured, estimated, or unmeasured)
- Trade-offs and risks

### Validation
- Database engine/version and test environment
- Before/after execution plans or benchmarks, if available
- Result-equivalence tests
- Commands run and exact outcomes
- Checks not run and why

### Schema and index recommendations
- Existing versus proposed indexes/constraints
- Migration and rollback requirements
- Any proposal requiring approval

### Independent review
- Reviewer used, findings, and unresolved disagreements
- Explicitly “not performed” if no independent reviewer was available

### Remaining risks
Never conclude “SQL is optimal” or “bug-free.” Say what was inspected and what evidence supports the conclusion.

## Suggested invocation

“Use `$sql-efficiency-review` to audit this repository’s SQL and ORM access paths. The schema was not explicitly designed up front. Infer the data model from migrations/models/callers, prioritize high-impact query shapes, inspect existing indexes, and propose or implement only evidence-supported optimizations. Validate result equivalence and use a fresh independent agent/session to challenge the findings. Do not run risky queries against production or invent benchmark results.”

## Review-only mode

If the user asks only for an audit:
- do not modify files or schema;
- deliver prioritized findings and safe reproduction/measurement steps;
- separate confirmed issues from hypotheses.

## Optimization mode

If the user authorizes implementation:
- make minimal targeted changes;
- include regression tests for changed semantics;
- use migrations for schema changes;
- do not apply migrations to production without explicit approval;
- compare plans/benchmarks when safe and available;
- run independent review after the changes.
