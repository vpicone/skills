---
name: remove-pointless-comments
description: >-
  Removes pointless, noisy, or redundant code comments — comments that restate
  WHAT the code does rather than explaining WHY a non-obvious choice was made,
  and comments likely to go stale. Use when asked to clean up comments, strip
  redundant/obvious comments, de-noise comments, remove what-comments, delete
  noise comments, or tidy over-commented code.
---

# Remove Pointless Comments

Strip comments that don't earn their place, while protecting the ones that do.
Safe by default: when unsure, **keep and flag** rather than delete.

## Core rule

A comment earns its place only when it explains **WHY** a non-obvious or
counterintuitive choice was made. Remove a comment when **any** of these is true:

- It describes **WHAT** the code does and the code already says that clearly
  (`i++ // increment i`, `// loop over users`, `// return the result`).
- It explains a WHY that is already obvious from the code or context.
- It restates the function/variable name in prose.
- It is likely to **go stale fast**: specific line numbers, current values,
  "as of today", ticket states, or descriptions that duplicate logic that will change.
- It is **version-control commentary**, not source: "added by Bob",
  "changed this on 3/4", "old version below".

Self-explanatory code needs no comment. When in doubt about whether a WHY is
obvious, lean toward **KEEPING** it and flag it (see Borderline).

## ALWAYS preserve (never remove)

- **License / copyright headers**, shebangs (`#!/usr/bin/env bash`), encoding declarations.
- **Doc comments / docstrings**: JSDoc/TSDoc, Python docstrings, rustdoc `///`,
  Javadoc, etc. These document an API for external consumers and doc generators.
  Only touch these if the user **explicitly** asks to clean redundant doc comments.
- **Actionable markers**: `TODO`, `FIXME`, `HACK`, `XXX`, `NOTE`, `WARNING`.
- **Tool / compiler directives**: `eslint-disable`, `prettier-ignore`, `# noqa`,
  `# type: ignore`, `@ts-ignore`/`@ts-expect-error`, `// nolint`, pragmas,
  `#pragma`, codegen markers, region markers.
- **Any comment whose removal could change behavior or tooling output.**

## Borderline → flag, don't delete

- **Commented-out code**: do NOT auto-delete by default — it may be intentional. Flag it.
- **Partly useful, partly noise**: trim to the useful WHY rather than deleting
  wholesale; flag if the split is unclear.
- **Genuinely unsure**: keep it and list it under "flagged for review."

## Operating procedure

1. **Determine scope.**
   - If the user names specific files/paths, use those.
   - Otherwise default to the **git changed/staged files** in the working tree
     (`git status --porcelain`, `git diff --name-only`).
   - Only operate on the **whole repo** when the user explicitly asks for that.
2. **Edit in place, comments only.** Remove only comment syntax/content. Never
   alter executable code, string literals, or whitespace-significant code.
   When you remove a full-line comment, remove its whole line (don't leave a
   stray blank line or dangling indentation). When you remove a trailing
   comment, leave the code and trim the now-trailing whitespace.
3. **Be language-agnostic.** Handle line and block comments across common
   languages (`//`, `/* */`, `#`, `--`, `<!-- -->`, `"""` docstrings vs strings,
   etc.). Never mistake a comment-like token **inside a string literal** for a
   real comment (`url = "http://x"`, `re.compile("# not a comment")`).
4. **Verify comments only changed.** Confirm no code tokens were removed and
   files still parse/lint where feasible. Do **not** run formatters or make
   unrelated edits.
5. **Report concisely.** Count of comments removed per file, plus a clearly
   separated list of items **FLAGGED for review** (commented-out code,
   ambiguous cases). Tell the user to review via `git diff`. Do **not** ask for
   confirmation before editing — removal is the default — but surface anything
   borderline rather than silently deleting it.

## Examples

### TypeScript — remove WHAT, keep WHY, keep directive

```ts
// Before
function totalCents(items: Item[]) {
  // loop over items and add up the prices
  let sum = 0;
  for (const item of items) {
    sum += item.price; // add price to sum
  }
  // Stripe rejects sub-cent values, so round before returning
  // eslint-disable-next-line @typescript-eslint/no-magic-numbers
  return Math.round(sum * 100);
}
```

```ts
// After
function totalCents(items: Item[]) {
  let sum = 0;
  for (const item of items) {
    sum += item.price;
  }
  // Stripe rejects sub-cent values, so round before returning
  // eslint-disable-next-line @typescript-eslint/no-magic-numbers
  return Math.round(sum * 100);
}
```

Removed two WHAT comments. Kept the WHY (Stripe constraint) and the
load-bearing `eslint-disable` directive.

### Python — remove restatement, keep docstring + noqa

```py
# Before
def slugify(title: str) -> str:
    """Convert a title into a URL-safe slug."""
    # lowercase the title
    s = title.lower()
    import unicodedata  # noqa: E402
    # NFKD so accented chars decompose before we strip them
    return unicodedata.normalize("NFKD", s)
```

```py
# After
def slugify(title: str) -> str:
    """Convert a title into a URL-safe slug."""
    s = title.lower()
    import unicodedata  # noqa: E402
    # NFKD so accented chars decompose before we strip them
    return unicodedata.normalize("NFKD", s)
```

Removed `# lowercase the title` (WHAT). Kept the docstring (API doc), the
`# noqa` directive, and the non-obvious WHY about Unicode normalization.

### Go — remove stale/VCS noise, keep WARNING, flag commented-out code

```go
// Before
func retry(fn func() error) error {
    // changed from 5 to 3 on 3/4 by Sam
    const maxAttempts = 3
    // WARNING: callers assume fn is idempotent
    var err error
    for i := 0; i < maxAttempts; i++ {
        err = fn()
        if err == nil {
            return nil
        }
    }
    // err = fn() // old single-shot path
    return err
}
```

```go
// After
func retry(fn func() error) error {
    const maxAttempts = 3
    // WARNING: callers assume fn is idempotent
    var err error
    for i := 0; i < maxAttempts; i++ {
        err = fn()
        if err == nil {
            return nil
        }
    }
    // err = fn() // old single-shot path
    return err
}
```

Removed the VCS/stale comment (`changed from 5 to 3 on 3/4 by Sam`). Kept the
`WARNING` marker. **Flagged** the commented-out `// err = fn()` line for review
rather than deleting it.

### Shell — keep shebang + license, drop the obvious

```bash
# Before
#!/usr/bin/env bash
# Copyright 2026 Acme, Inc. — MIT License
set -euo pipefail
# set the output directory
out="./dist"
mkdir -p "$out"   # make the output directory
```

```bash
# After
#!/usr/bin/env bash
# Copyright 2026 Acme, Inc. — MIT License
set -euo pipefail
out="./dist"
mkdir -p "$out"
```

Kept the shebang and license header. Removed two WHAT comments.
