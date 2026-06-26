# skills

Agent skills for Claude Code, installable with the [`skills` CLI](https://github.com/vercel-labs/skills) (`npx skills`).

## Skills

- **[remove-pointless-comments](skills/remove-pointless-comments/SKILL.md)** —
  Removes pointless, noisy, or redundant code comments (the ones that restate
  WHAT the code does instead of WHY) while preserving docstrings, license
  headers, `TODO`/`FIXME` markers, and tooling directives. Safe by default:
  flags commented-out code and ambiguous cases instead of deleting them.

## Install

Install a skill globally (to `~/.claude/skills/`, available across all projects):

```sh
npx skills add <owner/repo> --skill remove-pointless-comments -g -a claude-code
```

Replace `<owner/repo>` with this repository's GitHub slug (e.g. `your-org/skills`).

Flags:

- `--skill remove-pointless-comments` — install just this skill from the repo.
- `-g` — install globally to `~/.claude/skills/` instead of the current project.
- `-a claude-code` — target the Claude Code agent.

Drop `-g` to install into the current project's `.claude/skills/` instead.
