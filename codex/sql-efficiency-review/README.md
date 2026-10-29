# sql-efficiency-review

Codex skill for auditing SQL/ORM data-access paths, inferring an undocumented schema, prioritizing database performance issues, and using independent review.

## Install for one repository
Copy `sql-efficiency-review/` into:
`.agents/skills/sql-efficiency-review/`

## Install globally
Copy `sql-efficiency-review/` into:
`~/.agents/skills/sql-efficiency-review/`

Invoke in Codex with `$sql-efficiency-review`.

This skill does not blindly add indexes or claim speedups without evidence. It prohibits risky production validation without explicit authorization.
