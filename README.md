# Skills

[![skills.sh](https://skills.sh/b/vpicone/skills)](https://skills.sh/vpicone/skills)

Agent skills for Claude Code that help your codebase stay clean.

## Install

```bash
npx skills@latest add vpicone/skills
```

## Reference

- **[remove-pointless-comments](./skills/remove-pointless-comments/SKILL.md)** — Removes pointless, noisy, or redundant comments (the ones that restate WHAT the code does instead of WHY) while preserving docstrings, license headers, `TODO`/`FIXME` markers, and tooling directives. Safe by default: flags commented-out code and ambiguous cases instead of deleting them.
- **[linear-pr](./skills/linear-pr/SKILL.md)** — Ships the work in the current repo as three linked artifacts: a Linear ticket summarizing the changes, a git branch named with Linear's generated branch name, and a GitHub pull request that references the issue so Linear auto-links and advances it. Requires the Linear MCP and an authenticated `gh` CLI.
