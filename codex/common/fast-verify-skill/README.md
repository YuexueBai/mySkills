# fast-verify

Codex skill for batched implementation checks followed by independent acceptance review.

## Install for one repository
Copy the `fast-verify` folder into:
`.agents/skills/fast-verify/`

## Install for your user account
Copy the folder into:
`~/.agents/skills/fast-verify/`

Then restart Codex if the skill does not appear. Invoke with `$fast-verify` or ask Codex to use the skill by name.

The skill intentionally batches repetitive tests; it does not authorize skipping required tests or claiming unrun checks passed.
