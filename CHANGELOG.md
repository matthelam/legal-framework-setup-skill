# Changelog

Summarised from the migration ledger in `legal-framework/references/audit-and-upgrade.md`, which
has the full detail and the upgrade steps for each version.

## 1.4.1

Hooks are called by absolute path. `settings-hooks.json` moves to exec form with
`${CLAUDE_PROJECT_DIR}/scripts/memory_hook.py`, so hooks no longer fail (exit 2, blocking
`UserPromptSubmit` and `PreToolUse`) once Claude changes into a subdirectory. `framework-lint`
gains a hook-path check. Backward-compatible.

## 1.4.0

The corpus records its own use: `search` and `read` log to a gitignored `.memory-log/`, and new
`usage`, `mark`, `reflect` and `promote` commands report how memory is being used and which
themes should become Legislation. New `memory-promote` skill. Hooks gain `PreToolUse` (blocks
hand-edits to `memory/` and `MEMORY.md`) and `Stop`. Backward-compatible.

## 1.3.0

Facts gain provenance and a lifecycle: required `provenance` and `checked`, plus `source`,
`resolves` and `rule`. `lint` gains cross-file checks, a new `audit` command reports open cases
and stale claims, and `MEMORY.md` pushes precedent first. Not backward-compatible: ships with a
backfill step for existing facts.

## 1.2.0

Knowledge-exchange layer: Common Law becomes portable between repositories through
`scripts/memory_exchange.py` and the `knowledge-export`, `knowledge-import` and `framework-lint`
skills. Backward-compatible.

## 1.1.0

Taxonomy extended to six tags with `knowledge`, for net-new domain facts. Backward-compatible.
