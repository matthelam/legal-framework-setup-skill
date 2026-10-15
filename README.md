# Legal Framework

A Claude Code skill that installs, audits and upgrades the **Legal Framework** for agent context
in any repository or Cowork directory:

| Tier | Lives in | Role |
|---|---|---|
| **Constitution** | root `CLAUDE.md` | Values and supreme laws; loaded every session |
| **Legislation** | per-directory `CLAUDE.md` | Architecture, patterns and laws for one unit; loaded just-in-time |
| **Common Law** | `memory/` + generated `MEMORY.md` | Accumulated experience as atomic facts (One Key Fact per file) |

The ideas behind it, and how it was built:

- [A Legal Framework for Agent Context](https://blog.matthelam.com/blog/a-legal-framework-for-agent-context)
- [Agent Harness: Implementing the Legal Framework](https://blog.matthelam.com/blog/implementing-the-legal-framework)

## Install

1. Download `legal-framework-vX.Y.Z.zip` from the [latest release](https://github.com/matthelam/legal-framework-setup-skill/releases/latest).
2. Unzip it so the `legal-framework/` folder sits in your skills directory:
   - for every project: `~/.claude/skills/legal-framework/`
   - for one project: `<repo>/.claude/skills/legal-framework/`
3. Start a new Claude Code session and ask it to "set up the legal framework".

The skill needs Python 3.10 or later with PyYAML in the target repository.

## Upgrading an existing install

Install the newer release over the old one, then ask Claude to "upgrade my legal framework".
Every installation is stamped with the version that wrote it (`FRAMEWORK_VERSION` in
`scripts/memory.py`), and the skill's migration ledger
(`legal-framework/references/audit-and-upgrade.md`) takes an install from any older version to
the current one without touching its project content or memory facts.

## Versions

See [CHANGELOG.md](CHANGELOG.md). Releases are built by GitHub Actions from a `vX.Y.Z` tag, and
the build refuses a tag that doesn't match the version stamped in the skill.
