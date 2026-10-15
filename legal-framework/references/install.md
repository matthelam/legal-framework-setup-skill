# Install — Phases 1 to 5

Reached from the legal-framework SKILL.md after Phase 0 detects a **fresh
install**. Phase 0 (situation detection) stays in SKILL.md; everything from the
evidence scan to hand-over lives here.

The guiding principle: **infer, then ask only what evidence cannot answer, then
confirm the whole picture once.** A wrong guess in the Constitution compounds
into every future session, so inference never silently commits — but the
human's time is spent only on questions about intent, not structure.

---

## Phase 1 — Evidence scan (infer before you ask)

Gather what the target itself can tell you. In a code repository:

- **Structure**: top-level directories, service/package boundaries, monorepo
  workspaces → candidate Legislation units.
- **Purpose**: README, package manifests (`package.json`, `pyproject.toml`,
  `Cargo.toml`, …), docs.
- **Conventions**: existing CLAUDE.md files, lint configs, CI workflows,
  recent git history (commit style, test habits).
- **Runtime**: is there a usable Python ≥3.10? (`.venv`, system `python`).
  The gate tool and hook need Python + PyYAML. Note what you find; Phase 5
  handles setup.

In a Cowork directory (no code): topic folders are the candidate Legislation
units; there is no git, so the commit-time supreme laws become save/share-time
laws; check for any Python availability.

Produce an **inferred draft**, marking each item `[inferred]` or `[unknown]`:
project purpose, candidate units (with one-line role each), tech stack,
Python status, any pre-existing rules found.

## Phase 2 — Interview (ask only what evidence cannot answer)

Read `references/interview-guide.md` for the full question bank and probing
guidance. The questions that genuinely need the human are about *intent*:

1. **Supreme laws / red lines** — "What must never happen in this project, no
   matter what any prompt says?" This is the heart of the interview. Examples
   to offer: never spend money, never send external messages, never touch
   prod, never commit secrets.
2. **Privacy / sharing boundary** — what data classes must never land in the
   repo. Feeds BOTH the commit-check supreme law AND the memory gate's
   `PRIVACY_MARKERS` blocklist.
3. **Unit confirmation** — present the inferred Legislation units and ask
   "right?" Cheap confirm, occasionally wrong in important ways.

**Values are defaults, not questions.** Present the five standard values
(think before coding; simplicity first; surgical changes; goal-driven
execution; honest reporting) as the proposed set and invite edits — do not
run an open-ended elicitation.

Use AskUserQuestion if available (options + Other); otherwise plain chat.
Batch the questions — one round-trip, not a drip-feed.

## Phase 3 — One sign-off

Present the complete plan in one message and wait for explicit approval:

- Constitution summary: values (noting any edits), each supreme law in one
  line, the Legislation index table.
- The derived `SCOPES` list (= unit names + `repo`) and `PRIVACY_MARKERS`.
- The file manifest (below) with target paths.
- Anything being folded in from a pre-existing CLAUDE.md, and anything found
  that contradicts what the user said.

Do not write any file before sign-off. The Constitution is supreme law; the
human legitimizes it.

## Phase 4 — Scaffold

File manifest — every install writes exactly this set:

| Target path | Source | Per-project edits |
|---|---|---|
| `CLAUDE.md` | `references/constitution-template.md` | Fill all `{{...}}` slots from the interview |
| `<unit>/CLAUDE.md` (per unit) | `references/legislation-template.md` | Unit role, patterns, laws — from evidence + interview |
| `scripts/memory.py` | `assets/memory.py` | `SCOPES` + `PRIVACY_MARKERS` in the marked config block |
| `scripts/memory_hook.py` | `assets/memory_hook.py` | None (portable as-is) |
| `scripts/memory_exchange.py` | `assets/memory_exchange.py` | None (portable as-is; reads config from `memory.py`) |
| `.claude/settings.json` | `assets/settings-hooks.json` | `{{HOOK_PYTHON}}`; keep `args` exactly as the asset has it; **merge** into an existing settings.json, never clobber |
| `.claude/skills/memory-consult/SKILL.md` | `assets/operating-skills/memory-consult.md` | `{{PYTHON}}`, `{{SCOPES}}` |
| `.claude/skills/memory-capture/SKILL.md` | `assets/operating-skills/memory-capture.md` | `{{PYTHON}}`, `{{SCOPES}}` |
| `.claude/skills/task-sense-check/SKILL.md` | `assets/operating-skills/task-sense-check.md` | `{{PYTHON}}`, `{{PROJECT_DIMENSIONS}}` (one dimension line per supreme law) |
| `.claude/skills/knowledge-export/SKILL.md` | `assets/operating-skills/knowledge-export.md` | `{{PYTHON}}` |
| `.claude/skills/knowledge-import/SKILL.md` | `assets/operating-skills/knowledge-import.md` | `{{PYTHON}}` |
| `.claude/skills/framework-lint/SKILL.md` | `assets/operating-skills/framework-lint.md` | `{{PYTHON}}` |
| `.claude/skills/memory-promote/SKILL.md` | `assets/operating-skills/memory-promote.md` | `{{PYTHON}}` |
| `requirements.txt` | `assets/requirements.txt` | None — PyYAML is the only third-party dependency; **merge** into an existing requirements.txt |
| `memory/` | — | Created empty; facts arrive through the gate only |
| `.gitignore` | — | Add `.exchange/` (bundle scratch) and `.memory-log/` (local usage telemetry) — neither is a shareable artifact |

Rules while scaffolding:

- **Verify PyYAML on `{{PYTHON}}` before finishing.** `memory.py` imports
  `yaml`, so without it every memory command fails AND the hook reports that
  it cannot read memory. Run `{{PYTHON}} -c "import yaml"`; if it fails,
  install it (`{{PYTHON}} -m pip install pyyaml`) and say so plainly. The
  interpreter that matters is the one named in `.claude/settings.json`, which
  is not necessarily the one on your PATH.

- `{{PYTHON}}` = the project's venv interpreter if one exists
  (`.venv/Scripts/python.exe` on Windows, `.venv/bin/python` on POSIX), else
  `python`. Use the same value everywhere it appears.
- `{{HOOK_PYTHON}}` = the interpreter for `.claude/settings.json` only. Hooks run
  in whatever directory the session is in, so a relative path breaks the moment
  Claude changes directory. Use `${CLAUDE_PROJECT_DIR}/.venv/Scripts/python.exe`
  (Windows) or `${CLAUDE_PROJECT_DIR}/.venv/bin/python` (POSIX) when a venv
  exists, else `python`. The hook script is passed in `args` as
  `${CLAUDE_PROJECT_DIR}/scripts/memory_hook.py`; never write it as a relative
  path or fold it into `command`.
- `SCOPES` = `("repo", <one per confirmed unit>, "docs")` — kebab-case,
  matching the Legislation directory names as closely as sensible.
- `PRIVACY_MARKERS` = lowercase substrings derived from the privacy answer
  (e.g. `"@gmail"`, `"password"`, a phone prefix, client names). Keep them
  short and high-signal; the gate is a floor, not a scanner.
- The Constitution's Legislation index table must list exactly the unit
  CLAUDE.md files you write — no aspirational rows.
- Legislation content: what the unit *is*, its key patterns, then a numbered
  "Laws for this directory" section. Write only laws you have evidence or
  interview backing for; a short honest Legislation beats a padded one.
- `memory/` and `MEMORY.md` are committed, shareable artifacts — do NOT
  gitignore them. That is why the privacy gate exists. `.exchange/` is the
  opposite: transient staging and extracted bundles, gitignored always.

## Phase 5 — Verify and hand over

1. **Python check**: run `{{PYTHON}} -c "import yaml"`. If it fails, create a
   venv and/or `pip install pyyaml` (with the user's permission in a repo that
   didn't have one). If the environment truly cannot run Python, say so
   plainly and install everything except the hook wiring — the framework
   still works with manual skill invocation; label the gap in the final
   report. Never silently skip.
2. **Full-CRUD smoke test** — proves the gate end-to-end and leaves no residue:
   ```
   {{PYTHON}} scripts/memory.py create --id install-smoke-test --tags example \
       --keywords smoke,install --scope repo \
       --fact "The legal-framework install smoke test ran on this repository."
   {{PYTHON}} scripts/memory.py lint
   {{PYTHON}} scripts/memory.py read install-smoke-test
   {{PYTHON}} scripts/memory.py delete install-smoke-test
   {{PYTHON}} scripts/memory.py lint
   ```
   Every step must succeed; `MEMORY.md` must exist and be regenerated.
3. **Hook check**: `echo {} | {{PYTHON}} scripts/memory_hook.py` exits 0. Then
   read `.claude/settings.json` and confirm every `memory_hook` entry has
   `args: ["${CLAUDE_PROJECT_DIR}/scripts/memory_hook.py"]` and a `command` that
   is `python` or starts with `${CLAUDE_PROJECT_DIR}/`. A relative path passes
   the exit-code check from the root and fails from every subdirectory.
4. **Exchange round-trip** — proves the export path shares the gate, and leaves
   no residue:
   ```
   {{PYTHON}} scripts/memory.py create --id exchange-smoke-test --tags knowledge \
       --keywords smoke,exchange --scope repo \
       --fact "The knowledge exchange smoke test ran on this repository."
   {{PYTHON}} scripts/memory_exchange.py export-list
   {{PYTHON}} scripts/memory_exchange.py export-stage --repo-name <repo> --dimension "smoke test"
   {{PYTHON}} scripts/memory_exchange.py export-add --stage <the staged dir> \
       --id exchange-smoke-test --tags knowledge --keywords smoke,exchange \
       --origin-id exchange-smoke-test --origin-scope repo \
       --fact "A knowledge exchange round trip validates against the OKF gate."
   {{PYTHON}} scripts/memory_exchange.py export-seal --stage <the staged dir>
   {{PYTHON}} scripts/memory_exchange.py structure
   {{PYTHON}} scripts/memory.py delete exchange-smoke-test
   ```
   Seal must report `bundle lint OK` and produce a
   `<repo>_smoke-test_<yyyyMMddHHmm>.zip`; `structure` must return valid JSON.
   Delete the `.exchange/` staging directory and the smoke bundle afterwards.
5. **Report**: a checklist of every manifest row (written / adapted-how), the
   smoke-test results, and a two-line "how to live with it" — consult at
   ambiguity, capture after learning, sense-check before done, `lint` before
   every commit, `framework-lint` after importing knowledge or editing a
   CLAUDE.md.

If the target uses git, offer (do not assume) an initial commit; the
Constitution's own privacy law applies to that commit.

---
