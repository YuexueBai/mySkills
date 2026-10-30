---
name: project-map
description: Build and maintain a compact, evidence-based project navigation map. Use it to locate modules, entry points, call flows, data models, tests, and commands without repeatedly scanning the whole repository.
---

# Project Map

## Purpose
Maintain a compact navigation index so coding agents can locate relevant code before exploring broadly. The map is not a substitute for source inspection, tests, IDE indexes, or runtime evidence.

## Rules
- Confirm the actual repository/workspace root and available tools. Do not assume the agent can access IDE-only project structure or language-server indexes.
- Read applicable `AGENTS.md`, README, manifests, and existing architecture documentation first.
- Start with a shallow directory/manifests overview. Exclude `.git`, dependencies, build output, caches, generated/vendor code, and large assets.
- Do not recursively read every source file. Use route registration, imports, dependency injection, migrations, manifests, and targeted symbol searches to identify architecture.
- Record only high-value navigation facts. Never copy large source excerpts or secrets, tokens, `.env` values, private keys, or sensitive production data.
- Every important claim must point to a real path and, where useful, a symbol/function/class/table. Mark uncertain relationships as inferred.
- Source code, migrations, tests, and runtime evidence outrank a stale map.
- Keep the map compact. For a small project, use one file; do not create empty documents just to follow a template.

## Suggested layout
Prefer `docs/ai/` when it fits existing project conventions:

```text
docs/ai/
  PROJECT_MAP.md   # entry point, navigation index, last verified date
  MODULES.md       # module responsibilities and dependencies
  FLOWS.md         # important request/data/job flows
  DATA_MODEL.md    # tables, relationships, migrations, query entry points
  COMMANDS.md      # verified run/test/build/lint/migration commands
```

Create only the documents the repository needs. The entry page must link to the other map files.

## Initial map creation

### 1. Establish workspace
- Confirm repository root, top-level structure, branch/worktree if visible.
- Read project instructions, README, package/build manifests, and existing architecture docs.
- Check ignore rules and identify generated/vendor directories.
- State which tools are actually available; never claim IDE indexing was used if only shell search was available.

### 2. Identify navigation anchors
Record relevant items when present:
- application startup and lifecycle;
- API routes/controllers and endpoint handlers;
- service/domain/business logic;
- repositories/DAO/ORM models and migrations;
- background jobs, queues, schedulers, event handlers;
- authentication, authorization, tenancy boundaries;
- external service clients and integrations;
- configuration loading (paths and variable names only, never secret values);
- frontend routes/pages/state/API clients;
- tests and fixtures;
- build, test, lint, type-check, migration, and local-run commands;
- cross-cutting logging, caching, transactions, and error handling.

For each important module include:
- path;
- responsibility in one or two sentences;
- key symbols or entry functions;
- verified upstream callers/downstream dependencies;
- related tests;
- confidence: verified, inferred, or unknown.

### 3. Map important flows
Document high-value flows only, using concrete paths and symbols. Examples:

`HTTP route -> controller -> service -> repository/ORM -> table`

`scheduler -> job -> batch processor -> external API`

Do not infer a call graph from directory names alone. Mark unverified edges as inferred.

### 4. Verify
- Check every referenced path exists.
- Confirm key symbols with targeted search or language tooling.
- Verify commands from manifests/docs; run only safe, relevant commands.
- Add last-verified date and a short list of unverified areas.
- Do not claim a complete architecture map after only a shallow scan.

## Task navigation protocol
For each new coding task:
1. Confirm workspace root and read applicable instructions.
2. Read `PROJECT_MAP.md` and follow its links.
3. Match the task to likely modules, flows, data model, and tests.
4. Inspect those files and direct dependencies; verify map claims against current source.
5. Before broad exploration, identify the likely 3–8 relevant files/symbols and the specific unknown that requires more search.
6. Expand search only if mapped paths are missing, source contradicts the map, relevant code is not found, tests fail unexpectedly, or the task crosses unmapped boundaries.
7. Update the map if the task reveals a meaningful structural fact or map defect.
8. Do not regenerate the whole map for an isolated change.

This is a search heuristic, not a hard limit. Security-sensitive changes and cross-cutting refactors may require broader inspection.

## Incremental maintenance
Update affected sections when a change:
- adds/removes/renames a module, route, major service, repository, job, or integration;
- changes important data flows or dependency direction;
- adds/removes tables, key relationships, migrations, or major query entry points;
- changes canonical build/test/migration commands;
- changes auth, authorization, tenant boundaries, or transaction ownership.

Usually do not update for local variable renames, formatting, or isolated fixes that do not affect navigation.

After updating:
- verify paths and symbols;
- update last-verified date;
- mention the map update in the change summary.
If uncertain, mark the fact inferred or leave it out.

## Fresh-session handoff
When handing work to another agent/session, provide:
- repository root and applicable instructions;
- map path and relevant sections;
- relevant source paths and symbols;
- important flow/data-model references;
- verified commands;
- known map gaps or stale areas.

The receiving agent must still inspect actual source and diff. The map reduces discovery cost; it does not replace review.

## Suggested prompts

### Create or refresh
“Use `$project-map` to create or refresh a compact project navigation map. Inspect project instructions and manifests, identify real entry points, modules, important flows, schema/migrations, tests, and verified commands. Do not read every file. Link claims to actual paths and symbols, mark uncertainty, avoid secrets, and verify referenced paths.”

### Navigate a task
“Use `$project-map` before implementing this task. Consult the existing map, identify the smallest relevant set of files and symbols, verify them against source, and expand the search only when evidence requires it.”

### Incremental update
“Use `$project-map` to update the map for this diff only. Inspect structural impact, update affected module/flow/data-model/command entries, verify paths and symbols, and avoid a full repository rescan.”

## Completion report
Report map files created/updated, entry points and flows identified, what was inspected versus inferred, commands verified, known gaps, and how future tasks should use the map.
