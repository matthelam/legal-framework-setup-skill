# Legislation template

One file per confirmed unit, at `<unit>/CLAUDE.md`. Legislation is loaded
just-in-time when the agent works inside that directory, so everything in it
must be *about this unit* — cross-cutting rules belong in the Constitution.

Write only what you have evidence or interview backing for. A short, honest
Legislation beats a padded one: three real laws are worth more than ten
plausible-sounding ones, because the agent treats every law as binding.

Shape (delete these instructions; output starts at the `#` line):

---

# Legislation — {{unit path, e.g. services/api/}}

{{One paragraph: what this unit IS — its role in the system, its boundary,
who calls it and what it must never reach into. This is the sentence the
agent reads before touching anything here.}}

## {{Architecture / patterns — pick the heading that fits}}

{{The unit's internal structure, key modules, and the patterns changes must
follow. Tables work well for enumerable things (endpoints, scripts, ports,
adapters). Point to deeper docs rather than duplicating them.}}

## Laws for this directory

1. **{{Short law name.}}** {{The rule, stated as an invariant, with the WHY
   in one sentence — agents follow explained rules far better than bare
   commands.}}
2. ...

---

Patterns worth copying from mature installations:

- A table enumerating the unit's pieces (script/module → purpose → notes)
  doubles as the unit's map and keeps the file short.
- Laws that name their enforcement ("X is a violation — `lint` catches it",
  "the gate blocks Y") get followed; laws with no teeth drift.
- If the unit owns a dangerous capability (money, PII, deletion, external
  sends), restate the relevant supreme law here in unit-local terms and add
  the unit's own guard ("self-guards regardless of what any caller claims").
