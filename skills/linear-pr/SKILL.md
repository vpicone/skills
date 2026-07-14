---
name: linear-pr
description: >-
  Ship the work in the current repo as a Linear ticket, a git branch named by
  Linear, and a GitHub pull request — in one flow. Use when asked to "open a PR
  for this", "make a Linear ticket and PR", "file a ticket and branch for these
  changes", "ship this work", or otherwise turn local (committed or uncommitted)
  changes into a Linear issue + branch + GitHub PR that are linked together.
---

# Linear → Branch → PR

Turn the work sitting in the current repo into three linked artifacts: a Linear
issue, a git branch named with Linear's generated branch name, and a GitHub PR
that references the issue so Linear auto-links and advances it.

Requires the Linear MCP tools (`mcp__claude_ai_Linear__*`) and an authenticated
`gh` CLI. If either is missing, stop and tell the user what to set up.

## Core rule

The Linear issue is created **first** so its branch name and identifier drive
everything downstream: the git branch is named exactly `branchName` from the
issue, and the PR body carries the issue identifier so Linear links and moves
the ticket automatically. Never invent a branch name — always fetch it from the
created issue.

Creating a Linear issue and a GitHub PR are outward-facing and awkward to undo.
**Show the drafted ticket (title + description), branch name, and PR
title/body, and get a quick confirmation before creating anything external.**
Everything before that (inspecting git, drafting text) needs no confirmation.

## Operating procedure

### 1. Inspect the work

Figure out *what changed* and *where it lives* before writing anything:

```bash
git branch --show-current
git status --porcelain
# base branch:
git symbolic-ref --quiet --short refs/remotes/origin/HEAD 2>/dev/null | sed 's@^origin/@@' \
  || gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

Then read the actual changes to summarize them:

- Uncommitted work: `git diff HEAD` (and `git diff --staged`).
- Local commits not on the base: `git log --oneline <base>..HEAD` and
  `git diff <base>...HEAD`.

Classify the situation — it decides step 4:

- **A. Uncommitted changes on the base branch** (e.g. on `main`, dirty tree).
- **B. Committed locally on the base branch, not pushed** (ahead of `origin/<base>`).
- **C. Already on a feature branch** (committed and/or dirty).

If there is genuinely nothing to ship (clean tree, no commits ahead of the
base), stop and say so — do not create an empty ticket.

### 2. Draft the ticket

From the diff/commits, write a **concise, specific** issue:

- **Title**: imperative and scoped (e.g. "Cache team lookups in linear-pr flow"),
  not "Various changes".
- **Description** (Markdown, literal newlines — do not escape): a one-line
  summary, then a short "What changed" bullet list grounded in the actual diff,
  and any follow-ups/risks you noticed. Don't pad it.

### 3. Resolve the Linear team

- If the user named a team, use it.
- Else `mcp__claude_ai_Linear__list_teams`. One team → use it. Multiple → ask
  which, and offer to remember the choice in memory for this repo.

### 4. Create the issue, get the branch name  ← confirmation checkpoint

Show the user the drafted title, description, resolved team, and the branch
name + PR title you intend to use, then on confirmation:

1. `mcp__claude_ai_Linear__save_issue` with `title`, `team`, `description`, and
   `assignee: "me"`. Capture the returned identifier (e.g. `HUME-123`).
2. `mcp__claude_ai_Linear__get_issue` with that identifier and read
   `branchName` (e.g. `vince/hume-123-cache-team-lookups`). This is the exact
   branch name to use — do not modify it.

### 5. Create the branch and carry the work over

Use the case from step 1:

**A — uncommitted on base:** create the branch (this brings the uncommitted
changes along), then commit.

```bash
git checkout -b <branchName>
git add -A && git commit -m "<title>

<optional body>"
```

**B — committed on base, unpushed:** move the commits onto the new branch and
restore the base ref to the remote, without touching the working tree.

```bash
git checkout -b <branchName>          # new branch now holds the local commits
git branch -f <base> origin/<base>    # rewind base to remote (base is not checked out, safe)
# then commit any remaining uncommitted work as in case A
```

**C — already on a feature branch:** if its name isn't the Linear branch name,
rename it (`git branch -m <branchName>`); commit anything uncommitted. If the
branch is already pushed under the old name, note that to the user rather than
silently rewriting remote refs.

Never run `git reset --hard` or force-move a branch that is currently checked
out. If the required move is ambiguous (diverged history, existing remote
branch), stop and lay out the options instead of guessing.

### 6. Push and open the PR

```bash
git push -u origin <branchName>
gh pr create --base <base> --head <branchName> \
  --title "<title>" \
  --body "$(cat <<'EOF'
<one-line summary>

<what-changed bullets>

Fixes HUME-123
EOF
)"
```

- Put the issue identifier in the body (`Fixes <IDENTIFIER>` / the issue URL) so
  Linear links the PR and advances the ticket. Follow the repo's PR template if
  one exists (`.github/pull_request_template.md`).
- Respect any commit-trailer conventions the environment specifies.

### 7. Report

Give the user both links: the Linear issue URL and the `gh`-returned PR URL, and
one line on what the branch is called. Note anything you skipped or assumed
(e.g. "left `main` pointing at origin", "used the sole team FOO").
