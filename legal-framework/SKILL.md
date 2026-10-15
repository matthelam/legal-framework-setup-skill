---
name: legal-framework
description: "Install, audit, upgrade, or repair the Legal Framework for agent context — a Constitution (root CLAUDE.md: values and supreme laws), Legislation (per-directory CLAUDE.md), and Common Law (an OKF repository-memory system with provenance, usage reflection, and promotion into Legislation) — in any repository or Cowork directory. Use for \"set up the legal framework\", \"add a constitution\", \"create legislation\", \"install common law / repository memory\", \"scaffold my CLAUDE.md structure\", \"set up agent governance for this repo\", \"port the framework from another repo\", or a new project needing the same agent-context pattern — even if only one tier is named. Also for auditing an install (\"audit my legal framework\") and UPGRADING one (\"upgrade my legal framework\", \"bring this repo up to date\", \"I've installed a newer version of the skill\") — upgrades preserve all project content and memory facts. Installs the knowledge-exchange layer, so Common Law can be exported to and imported from other repositories."
---

# The Legal Framework — installer, auditor, and upgrader

**Framework version: 1.4.1.** Every installation is stamped with the version it
was written by, in `scripts/memory.py` as `FRAMEWORK_VERSION`. That stamp is the
only thing Upgrade mode needs to work out what an existing install is missing.

This skill installs the Legal Framework for agent context
(https://blog.matthelam.com/blog/a-legal-framework-for-agent-context) into a
target repository or Cowork directory, or audits/repairs an existing
installation. The framework has three tiers plus an operating loop:

| Tier | Lives in | Role | Portability |
|---|---|---|---|
| **Constitution** | root `CLAUDE.md` | Values + supreme laws; loaded every session; final fallback | Skeleton is fixed; supreme laws come from the human |
| **Legislation** | per-directory `CLAUDE.md` | Architecture, patterns, and laws for one unit; loaded just-in-time | Shape is fixed; content inferred + confirmed |
| **Common Law** | `memory/` + generated `MEMORY.md` | Accumulated experience as atomic OKF facts (One Key Fact per file) | Machinery is identical everywhere; only scopes, privacy markers and staleness horizon are per-project |

The **operating loop** is installed alongside: a reflective hook and four skills
— `memory-consult` (before acting), `memory-capture` (after learning),
`task-sense-check` (before saying "done"), and `memory-promote` (when experience
has hardened into a rule). Together: **consult → work → capture → sense-check**,
with **promote** as the periodic pass that moves settled experience up a tier.

The **knowledge-exchange layer** (v1.2.0) makes the Common Law portable between
repositories: `knowledge-export`, `knowledge-import`, `framework-lint`. Both
directions run through `scripts/memory_exchange.py`, which imports `memory.py`
rather than reimplementing it — one OKF gate, shared by local capture, export
staging, and import. **Exports are generalised, never copied verbatim**, and **a
fact that conflicts with the destination's law is never imported**, with the
human the only one who can clear a conflict.

---

## The two kinds of fact

Common Law records *Issues, Learnings, and Resolutions*, mapped onto the fixed
six-tag taxonomy — Issues → `issue`; Resolutions → `fix`; Learnings →
`preference`, `example`, `verification`, `knowledge`. Explain it with the
triple; enforce it with the six tags.

Cutting across the taxonomy is a distinction that decides **how a fact reaches a
decision**:

**A CASE** is an issue and, when found, the resolution that closes it. Retrieved
by **search**, from a symptom, mid-investigation. The `fix` names the `issue` it
closes via `resolves:`; an issue no fix names is an **open case**. Open is
*derived* from that link and stored nowhere else, so the state has one owner and
cannot drift.

**A PRECEDENT** is a fine-tuned reading of a rule in a CLAUDE.md, naming it via
`rule:`. Retrieved **deterministically**, and *pushed*, never pulled: `MEMORY.md`
indexes precedent first, and the SessionStart hook emits every
Constitution-scoped reading. Because:

> **an interpretation of a rule that has to be searched for has already failed.**
> By the time anyone thinks to look one up, they have read the rule and formed
> their own. The settled reading has to arrive first or it does not arrive.

A precedent that does not cite the rule it construes is a floating opinion.

## Provenance — every fact declares how it is known

Each fact carries a **required** `provenance` tier and `checked` date.

| Tier | Means | Licenses |
|---|---|---|
| `observed` | Run, and the result seen | Reliance; outranks a contradicting document for immediate work |
| `documented` | The vendor's own material says so, untested | Repeating what is *claimed* — never asserting the thing behaves that way |
| `contractual` | Authoritative statement made outside the documentation | Reliance, **only** with `source:` naming who said it and when |
| `unverified` | Inference, or a house rule asserting nothing about external behaviour | A working assumption, declared as one |

**The tier follows where the AUTHORITY sits, never the subject matter.** A
workaround a vendor engineer recommends that the documentation does not sanction
is `contractual`, never `documented`. An unattributed `contractual` claim is
`unverified` wearing a better label, and the gate rejects it, placeholders
included.

`checked` is when the claim was last confirmed true, **not** when the file was
edited — rewording never refreshes it. `observed` and `documented` age past
`STALE_DAYS`; `unverified` never does, because it was never fresh.

**Why fields and not prose.** Authors following the discipline write the tier
into the body anyway. Frontmatter is what makes it *filterable, lintable and
ageable* — the difference between a corpus you can ask "what have I never
verified?" and one where the question has no answer.

## The corpus records its own use (v1.4.0)

`search` and `read` append to a gitignored `.memory-log/queries.jsonl`, and
`reflect` reads it back. This exists because **without it, every claim about
whether memory is working is unfalsifiable** — including the claim that the
consult nudge is being followed.

Outcome is **derived from behaviour, not self-reported**. A `read` after a search
means the fact was worth opening; search shows only the one-line fact, so opening
one is a real signal that cannot be forgotten the way a self-report can. `mark`
exists for the two signals behaviour cannot show (`dead_end`, `corrected`) and is
never depended on.

Scoring is time-decayed and corroborated: signals halve every 30 days, and a fact
needs two distinct positive signals before it counts as relied on. Decay rather
than expiry, because a fact consulted twice last year is not dead, merely quiet.

The bucket nothing else can give you is **never surfaced** — facts no search has
ever returned. Two very different causes, and telling them apart is the human's
job: dead weight, or keywords that do not match how anyone actually asks. The
second is a *retrieval* defect wearing the costume of a content defect.

**Tell the human plainly that `reflect` says nothing useful for a fortnight.** It
reads a log that starts empty. An empty report is the correct output, not a fault.

## Promotion — the staircase into Legislation

Common Law outgrows Legislation in every install that gets used. The same lesson
gets learned nine times and never written into the rules, so every session
re-derives it. `memory.py promote` finds themes that clear two gates:
**frequency** (learned enough times to be a pattern, not an anecdote) and
**absence** (the governing CLAUDE.md barely mentions it), weighted by **demand**
from `reflect`. A rule is what you would otherwise look up again and again, so
frequency without use, or use without frequency, is not enough.

Three rules, and the installer must never relax them:

1. **`promote` proposes; the human writes.** An agent that edits the rules it is
   governed by has stopped being governed by them. Same wall as law 5 — drafting
   is allowed, committing is not.
2. **Promoting never deletes the promoted facts.** The rule states the position;
   the facts remain its evidence. Delete them and the rule can never be
   re-checked, which is the whole purpose of the provenance tiers.
3. **The weakest tier in the cluster bounds what the rule may assert.** A rule
   drawn from a set containing one `unverified` fact cannot be worded as
   established behaviour. Word it as the practice to follow, not as a claim.

**Synthesis never lands in `memory/`.** A lesson merging facts across four tiers
and four dates has no honest provenance — the weakest tier makes it worthless,
the strongest is the rounding-up law 2 forbids. Atomicity is what makes each fact
independently verifiable, ageable and correctable. Legislation is where general
positions live; Common Law is where checkable atoms live. Keep them apart.

## Amending the vocabularies is a version change, not a local edit

The six tags, four provenance tiers, and the field vocabulary are closed and
shared by the gate, the capture skill, and the consult skill's routing. Changing
one in a single installation silently breaks retrieval parity with every other. A
new category is added to all of them at once, given a consult route (a tag or
tier nothing queries is dead storage), and shipped as a version bump with a
migration line — never introduced per-project.

---

## Phase 0 — Detect the situation, then load the right reference

Read the target's root `CLAUDE.md` (if any).

- If it declares the Legal Framework → an **existing install**. Read
  `FRAMEWORK_VERSION` from `scripts/memory.py`:
  - **older, or absent** → **upgrade mode** (absent means pre-1.1.0);
  - **equal** → **audit mode**;
  - **newer** → stop; never downgrade an install written by a later version.
- Otherwise → **fresh install**. If a root `CLAUDE.md` exists with other content,
  plan to *fold it in*, not overwrite.

Then read exactly the reference you need — they are mutually exclusive, and
reading the wrong one anchors you on the wrong workflow:

| Situation | Read |
|---|---|
| Fresh install | `references/install.md` |
| Existing install, same version | `references/audit-and-upgrade.md` |
| Existing install, older version | `references/audit-and-upgrade.md` (+ migration ledger) |

Supporting references, loaded when the phase that needs them says so:
`references/interview-guide.md`, `references/constitution-template.md`,
`references/legislation-template.md`.

**Upgrading to 1.3.0 requires a rewrite step** — `provenance` and `checked` are
required, so every pre-existing fact fails the gate until backfilled. Back up
`memory/` first, present the derived tiers for sign-off, then write. Two rules
govern the backfill:

1. A fact citing the documentation in order to **disagree** with it is
   `observed`, never `documented`. A plain keyword match inverts exactly those
   facts, which is the error the tiers exist to prevent.
2. **Never derive a tier from a tag.** `verification` does not mean `observed`.

**1.4.0 is backward-compatible** — no schema change, so nothing needs rewriting.

**1.4.1 rewires the hooks** — `.claude/settings.json` must call the hook by an
absolute path (`${CLAUDE_PROJECT_DIR}/scripts/memory_hook.py`, exec form), never
`scripts/memory_hook.py`. A relative path resolves against the session's current
directory, so every hook fails once Claude changes into a subdirectory, and a
failed `UserPromptSubmit` or `PreToolUse` hook blocks the action.

**The guiding principle of every mode:** infer, then ask only what evidence
cannot answer, then confirm the whole picture once. A wrong guess in the
Constitution compounds into every future session, so inference never silently
commits — but the human's time is spent only on questions about intent.

## Things this skill never does

- Scaffold before the Phase 3 sign-off.
- Overwrite a pre-existing CLAUDE.md or settings.json (fold in / merge).
- Invent supreme laws the human didn't state, or tags and tiers outside the
  vocabularies — those widen only by a version bump applied everywhere.
- Overwrite project content during an upgrade: `SCOPES`, `PRIVACY_MARKERS`,
  `STALE_DAYS`, supreme laws, Legislation, and `memory/` facts survive every
  migration, including tiers, dates, and `resolves`/`rule` links.
- Downgrade an installation stamped with a newer version than the skill.
- Hand-edit `memory/` or `MEMORY.md` — even during install and migration, facts
  go through `memory.py`. From 1.4.0 the `PreToolUse` hook enforces this.
- Assign a provenance tier the evidence does not support, or seed `resolves` /
  `rule` links on the human's behalf without asking. Offer; never assume.
- Let `promote` write to a CLAUDE.md, or delete the facts it promotes.
- Write synthesised lessons into `memory/`. Synthesis goes to Legislation.
- Make precedent searchable *instead of* pushed. Adding a query path is fine;
  removing its delivery is what breaks it.
- Move knowledge between repositories by copying files. `provenance`, `source`
  and `checked` travel with a fact; `resolves` and `rule` never do, because both
  name things that exist only in the source repository.
- Put personal/secret content anywhere in the scaffolded files.

