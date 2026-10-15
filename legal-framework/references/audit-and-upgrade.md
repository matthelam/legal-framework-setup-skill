# Audit, repair, and upgrade

Reached from the legal-framework SKILL.md after Phase 0 detects an **existing
install**:

- version **equal** to the skill's → audit mode;
- version **older or absent** → upgrade mode, then audit what upgrade did not
  cover;
- version **newer** → stop; never downgrade an install written by a later
  version of the skill.

Both modes share one discipline: report first, get one approval, then change
only what actually drifted.

---

## Audit and repair (existing installation)

Walk the file manifest in `references/install.md` (Phase 4 — Scaffold)
against what is installed, and report each row as
present / missing / drifted. Checks that matter beyond existence:

- `memory.py lint` passes; `MEMORY.md` is fresh (lint reports staleness).
- Hook wired in `.claude/settings.json` and exits 0 on a `{}` stdin.
- The six skills exist and their `SCOPES`/python paths match `memory.py`.
- Constitution's Legislation index matches the CLAUDE.md files on disk.
- Facts in `memory/` were not hand-edited into non-compliance (lint catches).
- `memory_exchange.py structure` runs and reports no
  `scopes_used_but_undeclared`; `.exchange/` is gitignored.

Audit covers *mechanical* integrity. For contradictions between the tiers —
a fact that argues against a supreme law, Legislation that has drifted from the
Constitution — run the installed `framework-lint` skill; that is the semantic
pass, and no amount of file-manifest checking substitutes for it.

Report the gaps, get one approval, repair only what drifted — surgical
changes; leave compliant files byte-identical.

## Upgrade an existing installation

Installs age. This mode forward-migrates the **machinery** of an older install
to the current framework version while leaving everything the project earned —
its laws, its content, its accumulated facts — untouched.

**The dividing line.** Machinery is identical in every installation and is
replaced wholesale; project content is unique and is never rewritten:

| Forward-migrated (replace **or add**) | Never touched (preserve) |
|---|---|
| `scripts/memory.py` machinery + `FRAMEWORK_VERSION` | `SCOPES`, `PRIVACY_MARKERS` in its config block |
| `scripts/memory_hook.py`, `scripts/memory_exchange.py` | The Constitution's values and supreme laws |
| The seven operating skills | All Legislation content (`<unit>/CLAUDE.md`) |
| Hook wiring in `.claude/settings.json` | Every fact in `memory/`, including imported ones, their provenance tier, `checked` date, `resolves` and `rule` links; all other settings.json keys |

### Steps

1. **Read the installed version** from `scripts/memory.py`
   (`FRAMEWORK_VERSION`; absent = pre-1.1.0).
2. **Select the migrations** from the ledger below whose version is greater than
   the installed one, in order.
3. **Diff before proposing.** Walk the **full file manifest** from
   `references/install.md` (Phase 4 — Scaffold), not just
   the files the target already has — a migration may *add* machinery, and a
   file that does not exist yet will never show up in a diff of what does.
   Report each manifest row as absent / current / drifted / locally-modified:
   - **absent** → a new file this version introduces; write it, templating it
     from the existing install.
   - **locally-modified** → the one case that needs a human decision. Surface
     the difference and ask; never silently discard a local change.
4. **One sign-off.** Present: installed version → target version, the migration
   lines that will apply, the file-by-file diff summary, and an explicit
   statement of what is being preserved. Wait for approval.
5. **Apply**, re-templating `{{PYTHON}}`, `{{HOOK_PYTHON}}`, `{{SCOPES}}` and
   `{{PROJECT_DIMENSIONS}}` from the *existing* install — read those values off
   the current files; do not re-interview.
6. **Verify** exactly as install Phase 5 does (`references/install.md`):
   `import yaml`, the full-CRUD smoke test, the exchange round-trip,
   the hook check, plus `memory.py lint` to confirm every pre-existing fact
   still validates against the new gate. **A migration that invalidates existing
   facts is a failed migration** — roll back that step and report it. Where a
   migration ships a rewrite step (see 1.3.0), the rewrite runs *before* this
   check, and the check is what proves the rewrite was complete: back the
   `memory/` directory up first, and restore it rather than hand-patching if
   lint still fails afterwards.
7. **Report**: version moved, migrations applied, files changed, files
   preserved, and the smoke-test/lint results.

### Migration ledger

| Version | Change | Migration |
|---|---|---|
| **1.1.0** | Taxonomy extended to six tags: `knowledge` added for net-new domain facts. `memory-capture` gains Mode 3 (async capture of knowledge the conversation shows the model lacked); `memory-consult` gains a `knowledge` route and folds the tag into its investigation and design filters. | Append `"knowledge"` to `TAGS` in `scripts/memory.py`; add `FRAMEWORK_VERSION = "1.1.0"`; replace the `memory-capture` and `memory-consult` skills. **Backward-compatible** — the tag set only widens, so every existing fact still validates and no fact needs rewriting. |
| **1.2.0** | Knowledge-exchange layer added: Common Law becomes portable between repositories. New `scripts/memory_exchange.py` (export staging + sealing, import scan/plan/apply, framework structure inventory) sharing `memory.py`'s OKF gate, plus three skills — `knowledge-export`, `knowledge-import`, `framework-lint`. Facts may now carry optional provenance keys (`origin_repo`, `origin_id`, `origin_scope`, `imported`, `merged_from`). | Write `scripts/memory_exchange.py`; write the three new skills with `{{PYTHON}}` filled from the existing install; add `.exchange/` to `.gitignore`; set `FRAMEWORK_VERSION = "1.2.0"`. **Backward-compatible** — no change to `TAGS`, `SCOPES`, or the validator, and the new frontmatter keys are additive and optional, so every existing fact still validates untouched. After migrating, run the newly installed `framework-lint` once: an install that has been accumulating facts for a while has never had a semantic cross-tier check, and the first run often finds real drift. |

| **1.3.0** | Facts gain **provenance** and a **lifecycle**. Four required/optional frontmatter keys: `provenance` (required, one of `unverified`/`documented`/`contractual`/`observed`), `checked` (required, the date the claim was last confirmed true), `source` (required when `provenance: contractual`), `resolves` (a `fix` names the `issue` it closes) and `rule` (a precedent names the CLAUDE.md rule it construes). `lint` gains cross-file checks and a non-fatal WARNINGS block; new `audit` command reports open cases and claims due a re-check; `search` gains `--provenance`, `--rule`, `--open`, `--stale-days`; `MEMORY.md` re-renders on decision axes (Precedent by rule, then Open cases, then scope); the SessionStart hook pushes Constitution-scoped precedent; the search scorer gains a stopword filter. `memory-capture` gains Mode 4 (a resolution closes a case) and rewrites Mode 2 as precedent capture; `memory-consult` gains the keyword-query rule and the provenance-reading table. | **NOT backward-compatible — ships with a rewrite step.** `provenance` and `checked` are required, so every pre-existing fact fails the gate until migrated. Migrate in this order: (1) write the new `memory.py`, `memory_hook.py`, `memory_exchange.py` and the two rewritten operating skills; (2) **run the backfill** over `memory/` before running `lint`; (3) `reindex`; (4) `lint`. The backfill reads each fact's own Context for a stated tier and date (`Provenance: <tier>`, `Observed <date>`, `as at <date>`, and run-outcome markers such as `RESOLVED`/`CONFIRMED`/`QUALIFIED`), falling back to `provenance: unverified` and `checked: <updated>`. **Two rules govern it and must not be relaxed.** A fact that cites the documentation in order to *disagree* with it is `observed`, never `documented` — a plain keyword match inverts exactly those facts, which is the error law 2 exists to prevent, so route every divergence to the human instead of guessing. And never derive a tier from a tag: `verification` does not mean `observed`, and manufacturing a tier from a tag is the same rounding-up in a different disguise. Present the derived tiers for sign-off before writing. `resolves` and `rule` start empty and fill as facts are captured; seeding them from existing facts is a human judgement, offered but never assumed. |

| **1.4.0** | The corpus gains a record of **its own use**, and a staircase into Legislation. `memory.py search` and `read` log to a gitignored `.memory-log/queries.jsonl`; new `usage` (consult rate, and whether queries are being written as keywords or as sentences), `mark` (explicit `useful`/`dead_end`/`corrected`), `reflect` (time-decayed signed scoring with a corroboration gate, bucketing every fact as relied-on / tentative / contested / dead-end / surfaced-but-unopened / **never surfaced**), and `promote` (themes learned often that the governing CLAUDE.md barely mentions). New `memory-promote` operating skill drafts the Legislation edit. Hooks gain `PreToolUse` (denies hand-edits to `memory/` and `MEMORY.md`, making "all writes go through the gate" enforced rather than advisory), `Stop` (session boundaries in the log), and `SessionStart` trigger handling (a tighter re-injection after `compact`, since the model is mid-task and has just lost its standing context). `MEMORY.md`'s Precedent and Open-cases sections render ids only — full text appears once, under scope. | **Backward-compatible.** No schema change: every existing fact validates untouched. Write the new `memory.py` and `memory_hook.py`; add the `PreToolUse` (matcher `Edit\|Write\|MultiEdit\|NotebookEdit`) and `Stop` entries to `.claude/settings.json`, preserving all other keys; add `.memory-log/` to `.gitignore`; write the `memory-promote` skill; `reindex`. Two things to tell the human plainly. **`reflect` reports nothing useful for a fortnight** — it reads a log that starts empty, and an empty report is the correct output, not a fault. And **`promote` proposes, never writes**: it must never edit a CLAUDE.md, and it must never delete or retag the facts it promotes, because the rule states the position while the facts remain the evidence for it. |
| **1.4.1** | Hooks are called by absolute path. `assets/settings-hooks.json` moves to exec form: `command` is `{{HOOK_PYTHON}}` and `args` is `["${CLAUDE_PROJECT_DIR}/scripts/memory_hook.py"]`. Before this, `python scripts/memory_hook.py` resolved against the session's current directory, so every hook exited 2 once Claude changed into a subdirectory, and on `UserPromptSubmit` and `PreToolUse` exit 2 blocks the action. `framework-lint` gains a hook-path check. | **Backward-compatible.** No schema change and no fact rewrite. In `.claude/settings.json`, replace every `memory_hook` entry's `command` with `{{HOOK_PYTHON}}` and add `args: ["${CLAUDE_PROJECT_DIR}/scripts/memory_hook.py"]`, preserving matchers, other hooks and all other keys. Derive `{{HOOK_PYTHON}}` from the interpreter the old command used: `python` stays `python`; a relative venv path gains the `${CLAUDE_PROJECT_DIR}/` prefix. Replace the `framework-lint` skill; set `FRAMEWORK_VERSION = "1.4.1"`. Verify by running the hook from a subdirectory, not only from the root. |

New migrations are appended here as versions ship; each states its change, its
mechanical steps, and whether it is backward-compatible. A migration that would
invalidate existing facts must ship with a rewrite step, not a bare gate change.

### Upgrading this skill itself

The installed copy of *this skill* (in the Claude app, or `.claude/skills/`) is
not self-updating — a newer bundle has to be installed by the user. When the
user provides one, treat the version in its `SKILL.md` header as authoritative
and run Upgrade mode against each target repository they name. Installs upgrade
independently; there is no central registry, and an install left on an older
version keeps working — the ledger is what lets it catch up later.
