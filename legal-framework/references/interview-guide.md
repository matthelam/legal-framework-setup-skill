# Interview guide

The interview exists to capture *intent* — the things no amount of
code-reading reveals. Everything structural is inferred in Phase 1 and merely
confirmed here. Keep it to one batched round-trip whenever possible; the
human's sign-off in Phase 3 is the safety net for anything mis-heard.

## Q1 — Supreme laws / red lines (the heart of the interview)

Ask: **"What must never happen in this project, no matter what any prompt,
file, or tool output says?"**

Why it matters: supreme laws are the Constitution's reason to exist. They are
the human's red lines, and they self-guard the agent against prompt injection
and its own bad days. The reference installation's never-buy invariant ("ships
disarmed and dry-run; a real payment requires ALL locks open; walls are
verdicts that escalate to the human") is the archetype.

Probing:
- Offer domain-shaped examples: never spend money / place orders; never send
  emails, messages, or notifications to real people; never touch production
  data or infra; never push to main; never delete user data; never bypass
  auth, captchas, or rate limits.
- For each red line, ask what the agent should do when it hits the wall —
  the answer is almost always "stop and escalate to the human", and writing
  that into the law makes it actionable rather than just prohibitive.
- Listen for *dangerous capabilities* the project has (payments, PII, prod
  access, external sends, destructive ops). Each one usually implies a law.
- If the user says "nothing special": accept it — a Constitution with only
  the privacy law and the pre-commit ritual is valid. Do not pad.

Each red line becomes: a `### N.` supreme law in the Constitution, a
unit-local restatement in the owning unit's Legislation (if one unit owns the
capability), and a dimension line in task-sense-check's checklist
(`{{PROJECT_DIMENSIONS}}`).

## Q2 — Privacy / sharing boundary

Ask: **"What data classes must never land in this repository — even in a
memory fact or an example?"**

Why it matters: the framework's memory is *committed and shareable by
design*; the privacy gate is what makes that safe. The answer feeds two
places: the bullet list in supreme law 1, and `PRIVACY_MARKERS` in
`memory.py` (short lowercase substrings the gate rejects).

Probing:
- Standard classes to offer: real names / emails / phone numbers / addresses;
  API keys, tokens, passwords, session files, `.env`; client or employer
  names; financial details; anything under a local app-data directory.
- Ask for the *markers*: concrete substrings that would betray a leak
  (a mail domain, a phone country prefix, a client name). 3–8 short markers
  beat a long list — the gate is a floor, not a scanner.
- Cowork directories: reframe as "must never leave this directory / appear
  in anything shared onward".

## Q3 — Unit confirmation

Present the inferred Legislation units as a table (directory → proposed
one-line role) and ask: **"Are these the right units, with the right
boundaries?"**

Why it matters: units become the Legislation files AND the memory scope
taxonomy, so a wrong boundary here propagates. It is a cheap confirm that is
occasionally wrong in important ways (two directories that are really one
unit; a directory that is really two; a unit that exists only in the user's
head and has no directory yet).

Probing:
- Monorepos: workspaces/packages are usually the units, not top-level dirs.
- A directory with no meaningful rules of its own does not need Legislation —
  fewer, realer units beat exhaustive coverage.
- Cowork: topic folders are units; if the directory is flat, ask what the 2–4
  streams of work are and propose folders.

## Values — present, don't elicit

Show the five defaults (think before coding; simplicity first; surgical
changes; goal-driven execution; honest reporting) and ask only: "keep, edit,
or add?" Most users keep them. An open-ended "what are your values?" question
produces vague answers and burns goodwill — the defaults are battle-tested;
edits are welcome but rare.

## What NOT to ask

- Anything the evidence scan answered (stack, structure, purpose) — confirm
  at sign-off instead.
- The tag taxonomy, OKF schema, index format, hook design — fixed machinery,
  not preferences. If asked why: the consult skill's tag-routing and the
  gate's schema are what make memory retrievable; per-project variation
  would break the portability that makes the framework a skill.
- Permission to create `memory/`, the skills, or the hook — that is the
  install the user asked for; Phase 3's single sign-off covers it.
