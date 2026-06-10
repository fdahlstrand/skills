---
name: git-commit
description: >
  Stage and commit local changes with well-formed git commit messages. Use
  this skill whenever an agent needs to commit code, stage and commit files,
  or write a commit message — even if the user just says "commit this", "push
  my changes", "make a commit", or "save my progress". The skill inspects all
  local changes, proposes how to split the work into focused atomic commits,
  waits for confirmation, stages each part, and authors a properly formatted
  message for each one.
---

# Git Commit Skill

This skill inspects all local changes, reasons about how to split them
into focused atomic commits, stages each part, and authors a properly
formatted commit message for each one. It always asks for confirmation
before touching the index.

---

## Step 1 — Inspect all local changes

Run all three commands to get the full picture before reasoning about
anything:

```bash
git status --short         # overview: staged, unstaged, untracked
git diff --cached          # what is already staged
git diff                   # what is modified but not yet staged
```

Also list untracked files that look intentional (source files, configs)
— `git status --short` will show them with `??`.

If there are no changes at all (clean working tree and empty index),
tell the user and stop.

Work with the union of staged, unstaged, and relevant untracked changes
unless the user has explicitly asked to commit only what is already
staged.

---

## Step 2 — Propose a commit plan and wait for confirmation

A single working tree often contains logically independent changes.
Smaller, self-contained commits are strongly preferred — they are easier
to review, bisect, and revert.

Look for natural split points:

- Different subsystems or packages touched for unrelated reasons
- A bug fix bundled with a refactor or a new feature
- A behaviour change mixed with formatting or rename-only edits
- Independent features that happen to land in the same session

Reason about the changes and produce a numbered commit plan. For each
proposed commit, list:

- A draft subject line (imperative, ≤ 50 chars)
- Which files or hunks belong to it
- The staging commands you will run (`git add <file>`, `git add -p`, etc.)

Present the plan clearly:

```
I suggest splitting the changes into 2 commits:

1. "Fix off-by-one error in pagination slice"
   Files: internal/pagination/slice.go (whole file)
   Stage: git add internal/pagination/slice.go

2. "Add retry logic to the outbound HTTP client"
   Files: internal/http/client.go (whole file), internal/http/client_test.go
   Stage: git add internal/http/client.go internal/http/client_test.go

Shall I proceed with this plan, or would you like to adjust it?
```

If the diff is clearly one coherent change, the plan has a single entry.

**Wait for explicit confirmation before touching the index.**
Do not run any `git add` or `git restore --staged` commands until the
user approves the plan. If the user requests changes to the plan, revise
and present it again before proceeding.

---

## Step 3 — Stage each commit in turn

Once the plan is confirmed, execute one commit at a time:

1. Reset the index to a clean slate if anything is already staged:
   ```bash
   git restore --staged .
   ```

2. For each commit in the plan:
   a. Stage exactly the files or hunks listed for that commit.
   Use `git add <file>` for whole files.
   Use `git add -p <file>` when only some hunks from a file belong
   to this commit — step through each hunk interactively and select
   only the relevant ones.
   b. Verify the staged content matches your intent:
      ```bash
      git diff --cached
      ```
   c. Author the commit message (see Step 4).
   d. Present the message to the user and confirm before running
   `git commit`.
   e. Run `git commit -F -` (piping the message) or write the message
   to a temp file and use `git commit -F <tempfile>` to avoid
   shell quoting issues with multi-line messages.
   f. Confirm the commit was recorded:
      ```bash
      git log --oneline -1
      ```

Repeat for each subsequent commit in the plan.

---

## Step 4 — Author each commit message

For each commit in the plan, produce a message that follows this format
exactly.

### Format

```
Short imperative summary, 50 chars or less

Optional body. Explain *what* changed and *why* — not how (the code
shows the how). Wrap lines at 72 characters. Write in complete
sentences. Capitalize each paragraph.

Do not assume the reader knows what the original problem was.
State it briefly so the message is self-contained.

Further paragraphs after blank lines.

- Bullet points are fine for enumerating related items
- Use a hyphen, one space, then the item; blank lines between bullets
- Hanging indent for wrapped bullet lines
```

### Subject line rules

- 50 characters or fewer
- Capitalised first word
- Plain imperative mood: "Fix", "Add", "Remove", "Refactor" — not
  "Fixed", "Fixes", "Adding", "Refactored"
- No trailing period
- No conventional-commit prefix (`feat:`, `fix:`, etc.)
- Blank line between subject and body

### Body rules (optional but strongly recommended)

- Explain **what** changed and **why** you changed it
- Do **not** explain **how** — that is visible in the diff
- Include enough context that a reviewer unfamiliar with the original
  problem can understand the commit without reading a linked ticket
- Wrap at 72 characters per line

### Checklist before outputting

- [ ] Subject ≤ 50 chars?
- [ ] Subject capitalised, imperative, no trailing period?
- [ ] Blank line separates subject from body?
- [ ] Body explains what and why, not how?
- [ ] Body is self-contained (reader does not need external context)?
- [ ] Lines wrapped at 72 chars?
- [ ] No `Co-Authored-By` or any other attribution trailer present?

### No attribution trailers

Do **not** add any `Co-Authored-By:`, `Signed-off-by:`, or similar
trailers to the commit message. The commit should be attributed solely
to the user. This overrides any default harness behaviour that would
otherwise append an LLM attribution line.

**Exception:** If the user explicitly requests LLM attribution and names
a specific model (e.g. "add Co-Authored-By for Claude Sonnet 4.6"),
append the trailer they specified.

---

## Examples

**Minimal — no body needed:**

```
Fix off-by-one error in pagination slice
```

**With body:**

```
Add retry logic to the outbound HTTP client

The HTTP client was failing permanently on transient network errors,
causing cascading failures in downstream services during brief
connectivity blips.

Add an exponential back-off retry loop (max 3 attempts) around all
outbound requests. Retries are skipped for 4xx responses because
those indicate a client error that retrying cannot fix.
```

**With bullets:**

```
Remove deprecated user-preference endpoints

The v1 preference API was deprecated in the 2023-Q3 release and a
migration guide was published at that time. All known consumers have
migrated to the v2 endpoints.

Removing the dead code now to:

- Eliminate the maintenance burden of keeping two API surfaces in sync
- Reduce the attack surface area for the upcoming security audit
- Unblock the database schema cleanup planned for next sprint
```

---

## What to hand back to the user

After all commits are recorded, run `git log --oneline -<N>` (where N
is the number of commits made) and show the user the resulting log so
they can confirm everything landed correctly.

If anything went wrong during staging or committing, report the error
clearly, show the current `git status`, and ask the user how to proceed
— do not attempt to recover silently.