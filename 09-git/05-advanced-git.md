# Advanced Git

## Git Rebase

Rebase moves (replays) your commits on top of another branch's latest commit, creating a linear history.

### Basic Rebase

```bash
# You're on feature branch, main has moved ahead
git checkout feature/login
git rebase main
```

**Before rebase:**

```
main:    A---B---C
              \
feature:       D---E
```

**After rebase:**

```
main:    A---B---C
                  \
feature:           D'---E'
```

Commits D and E are **rewritten** (new hashes D', E') — they now sit on top of C.

### Interactive Rebase

Interactive rebase lets you edit, squash, reorder, or drop commits.

```bash
# Rebase last 4 commits interactively
git rebase -i HEAD~4
```

This opens your editor:

```text
pick a1b2c3d Add user model
pick e4f5g6h Add validation (WIP)
pick i7j8k9l Fix typo in model
pick m0n1o2p Add user controller

# Commands:
# p, pick   = use commit as-is
# r, reword = use commit, but edit the message
# e, edit   = use commit, but stop for amending
# s, squash = meld into previous commit (keep message)
# f, fixup  = meld into previous commit (discard message)
# d, drop   = remove commit entirely
```

### Squashing Commits

Combine messy WIP commits into one clean commit:

```text
pick a1b2c3d Add user model
squash e4f5g6h Add validation (WIP)
squash i7j8k9l Fix typo in model
pick m0n1o2p Add user controller
```

Result: three commits become one with a combined message.

### Rewording a Commit Message

```text
reword a1b2c3d Add user model (bad message)
pick e4f5g6h Add user controller
```

Git will pause and let you edit the message for the reworded commit.

---

## Rebase vs Merge

### When to Use Merge

```bash
git checkout main
git merge feature/login
```

- Preserves complete history (you can see when branches diverged and joined).
- Creates merge commits.
- Safe for shared/public branches.

### When to Use Rebase

```bash
git checkout feature/login
git rebase main
```

- Creates linear, clean history.
- No merge commits.
- Use for local/personal branches before merging.

### The Golden Rule of Rebasing

> **Never rebase commits that have been pushed to a shared branch.**

Rebase rewrites commit hashes. If others have based work on the original commits, their history diverges from yours — causing chaos.

```bash
# ✅ Safe: rebase your local feature branch onto updated main
git checkout my-feature
git rebase main

# ❌ Dangerous: rebase main (shared branch)
git checkout main
git rebase some-branch  # NEVER DO THIS
```

### Comparison Table

| Aspect                   | Merge                              | Rebase                            |
| ------------------------ | ---------------------------------- | --------------------------------- |
| History                  | Non-linear (shows branch topology) | Linear (one straight line)        |
| Merge commits            | Yes                                | No                                |
| Rewrites history         | No                                 | Yes (new commit hashes)           |
| Safe for shared branches | Yes                                | No                                |
| Best for                 | Integrating completed features     | Keeping feature branch up to date |

---

## Cherry-Pick

Cherry-pick applies a **specific commit** from one branch to another — without merging the entire branch.

```bash
# Apply commit abc123 to your current branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick without auto-committing (stage changes only)
git cherry-pick --no-commit abc123
```

### Use Cases

- Hotfix: a bug fix on `develop` needs to go to `main` immediately.
- Backporting: apply a feature commit to an older release branch.
- Recovering: grab a commit from a deleted branch.

### Example: Hotfix

```bash
# Bug fix is on develop (commit abc123)
git checkout main
git cherry-pick abc123
git push origin main

# Now main has the fix without merging all of develop
```

**Note:** Cherry-pick creates a **new commit** (different hash) — the original still exists on its branch.

---

## Git Stash

Stash temporarily saves uncommitted changes so you can switch branches with a clean working directory.

### Basic Stash

```bash
# Save current changes (tracked files)
git stash

# Save with a description
git stash push -m "WIP: login form styling"

# Include untracked files
git stash push -u -m "WIP: includes new files"

# Include everything (even ignored files)
git stash push -a -m "WIP: everything"
```

### Retrieving Stashed Changes

```bash
# Apply most recent stash and REMOVE from stash list
git stash pop

# Apply most recent stash but KEEP in stash list
git stash apply

# Apply a specific stash
git stash apply stash@{2}
```

### Managing Stashes

```bash
# List all stashes
git stash list
# stash@{0}: On feature/login: WIP: login form styling
# stash@{1}: On main: WIP: quick experiment

# Show what's in a stash (diff)
git stash show stash@{0} -p

# Drop a specific stash
git stash drop stash@{1}

# Clear all stashes
git stash clear
```

### Analogy

Stash is like putting your half-finished puzzle into a box, working on another puzzle, then pulling the first one back out exactly where you left off.

---

## Git Bisect

Bisect uses **binary search** to find the exact commit that introduced a bug.

### How It Works

```bash
# Start bisect
git bisect start

# Mark current commit as bad (has the bug)
git bisect bad

# Mark a known good commit (before the bug existed)
git bisect good a1b2c3d

# Git checks out a commit halfway between good and bad
# Test it, then tell git:
git bisect good   # if this commit does NOT have the bug
# OR
git bisect bad    # if this commit DOES have the bug

# Git narrows down and checks out another commit...
# Repeat until git finds the first bad commit

# When done:
git bisect reset  # Return to original branch
```

### Automated Bisect

If you have a test script that exits 0 (good) or 1 (bad):

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run npm test
# Git automatically finds the commit that broke the tests
```

### Example Output

```
Bisecting: 3 revisions left to test after this (roughly 2 steps)
[abc123] Add caching layer        ← git checks this out
...
abc123 is the first bad commit
```

Out of 128 commits, bisect finds the culprit in about 7 steps (log₂128 = 7).

---

## Git Reset

Reset moves the branch pointer backward, effectively "undoing" commits. The three modes differ in what happens to your changes.

### Three Modes

```bash
# Soft: undo commit, keep changes STAGED
git reset --soft HEAD~1

# Mixed (default): undo commit, keep changes UNSTAGED
git reset HEAD~1

# Hard: undo commit, DISCARD all changes
git reset --hard HEAD~1
```

### Visual Comparison

```
Commit: A --- B --- C (HEAD)

After git reset --soft HEAD~1:
  HEAD → B
  Staging area: has C's changes (ready to commit)
  Working directory: unchanged

After git reset --mixed HEAD~1:  (default)
  HEAD → B
  Staging area: empty
  Working directory: has C's changes (unstaged)

After git reset --hard HEAD~1:
  HEAD → B
  Staging area: empty
  Working directory: empty (C's changes are GONE)
```

### When to Use Each

| Mode      | Use Case                                                            |
| --------- | ------------------------------------------------------------------- |
| `--soft`  | Undo commit but keep changes staged (want to re-commit differently) |
| `--mixed` | Undo commit and unstage (want to review before re-staging)          |
| `--hard`  | Completely discard recent commits and their changes                 |

### Important Warning

`git reset --hard` is **destructive** for uncommitted work. Only use it on commits that have not been pushed to a shared branch.

```bash
# Safe: reset unpushed local commits
git reset --hard HEAD~2

# Dangerous: resetting pushed commits requires force push
git push --force-with-lease  # Rewrites shared history!
```

---

## Git Revert

Revert creates a **new commit** that undoes the changes of a previous commit — without rewriting history.

```bash
# Revert the most recent commit
git revert HEAD

# Revert a specific commit
git revert abc123

# Revert without auto-committing
git revert --no-commit abc123

# Revert a merge commit (specify which parent to keep)
git revert -m 1 abc123
```

### Reset vs Revert

```
Reset (rewrites history):
A --- B --- C       →       A --- B  (C is gone)

Revert (adds new commit):
A --- B --- C       →       A --- B --- C --- C'  (C' undoes C)
```

|                          | Reset                  | Revert                |
| ------------------------ | ---------------------- | --------------------- |
| Rewrites history         | Yes                    | No                    |
| Safe for shared branches | No                     | Yes                   |
| Creates new commit       | No                     | Yes                   |
| Use for                  | Local/unpushed commits | Pushed/shared commits |

### Example: Revert a Bad Deploy

```bash
# The last commit broke production
git revert HEAD
git push origin main
# main now has a revert commit — production is fixed
# The original commit is still in history for reference
```

---

## Git Reflog

Reflog (reference log) records every change to HEAD — even ones invisible in `git log`. It is your safety net for recovering "lost" commits.

```bash
git reflog
# abc123 HEAD@{0}: commit: Add user controller
# def456 HEAD@{1}: reset: moving to HEAD~2   ← you reset!
# ghi789 HEAD@{2}: commit: Add validation
# jkl012 HEAD@{3}: commit: Add user model
```

### Recovering Lost Commits

```bash
# You accidentally ran git reset --hard HEAD~3
# Your commits seem gone... but reflog has them!

git reflog
# Find the commit hash before the reset

git checkout abc123
# Or restore your branch to that point:
git branch recovery-branch abc123
# Or reset back:
git reset --hard abc123
```

### Reflog Expiration

- Reflog entries expire after **90 days** (reachable) or **30 days** (unreachable).
- After expiration, `git gc` can permanently remove orphaned commits.
- Until then, almost nothing in Git is truly lost.

### Analogy

Reflog is like the "Recently Deleted" folder — even after you delete something, there is a grace period to recover it.

---

## Git Tags

Tags mark specific commits — typically used for release versions.

### Lightweight Tags

```bash
# Simple pointer to a commit (like a branch that never moves)
git tag v1.0.0
git tag v1.0.0 abc123  # Tag a specific commit
```

### Annotated Tags (Recommended for Releases)

```bash
# Includes tagger info, date, and message (stored as full object)
git tag -a v1.0.0 -m "Release version 1.0.0"
```

### Working with Tags

```bash
# List all tags
git tag

# Show tag details
git show v1.0.0

# Push tags to remote
git push origin v1.0.0      # Push specific tag
git push origin --tags       # Push all tags

# Delete a tag
git tag -d v1.0.0            # Delete locally
git push origin --delete v1.0.0  # Delete from remote

# Checkout a tag
git checkout v1.0.0          # Detached HEAD at that tag
```

### Semantic Versioning with Tags

```
v1.0.0  →  MAJOR.MINOR.PATCH

MAJOR: Breaking changes (v1.x.x → v2.0.0)
MINOR: New features, backwards compatible (v1.0.x → v1.1.0)
PATCH: Bug fixes (v1.0.0 → v1.0.1)
```

---

## Git Hooks

Hooks are scripts that run automatically at specific points in the Git workflow.

### Where Hooks Live

```
.git/hooks/
├── pre-commit       # Runs before commit is created
├── commit-msg       # Runs after commit message is written
├── pre-push         # Runs before push
├── post-merge       # Runs after a merge
└── ...
```

### Common Hooks

#### pre-commit (lint/format before committing)

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ Linting failed. Fix errors before committing."
  exit 1
fi

# Run formatter check
npm run format:check
if [ $? -ne 0 ]; then
  echo "❌ Files not formatted. Run 'npm run format' first."
  exit 1
fi
```

#### commit-msg (enforce message format)

```bash
#!/bin/sh
# .git/hooks/commit-msg

commit_msg=$(cat "$1")
pattern="^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,72}$"

if ! echo "$commit_msg" | grep -qE "$pattern"; then
  echo "❌ Invalid commit message format."
  echo "Expected: type(scope): description"
  echo "Example: feat(auth): add login endpoint"
  exit 1
fi
```

#### pre-push (run tests before pushing)

```bash
#!/bin/sh
# .git/hooks/pre-push

npm test
if [ $? -ne 0 ]; then
  echo "❌ Tests failed. Fix before pushing."
  exit 1
fi
```

### Using Husky (Recommended for Teams)

Hooks in `.git/hooks/` are not committed to the repo. **Husky** solves this by storing hooks in the project:

```bash
npm install -D husky
npx husky init

# Create a pre-commit hook
echo "npm run lint" > .husky/pre-commit
```

```json
// package.json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

### lint-staged (Only Lint Changed Files)

```bash
npm install -D lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,ts,jsx,tsx}": ["eslint --fix", "prettier --write"],
    "*.{css,md,json}": ["prettier --write"]
  }
}
```

```bash
# .husky/pre-commit
npx lint-staged
```

---

## .gitignore Patterns

The `.gitignore` file tells Git which files/folders to exclude from tracking.

### Pattern Syntax

```gitignore
# Comments start with #

# Ignore specific file
secrets.env

# Ignore by extension
*.log
*.tmp

# Ignore directory (and all contents)
node_modules/
dist/
.env

# Ignore files in any subdirectory
**/*.test.js

# Negate (un-ignore) a pattern
!important.log

# Ignore only in root directory (not subdirs)
/build

# Wildcard: matches any characters except /
doc/*.txt

# Double wildcard: matches any path depth
**/logs/**
```

### Typical Full-Stack .gitignore

```gitignore
# Dependencies
node_modules/

# Build output
dist/
build/
.next/

# Environment variables
.env
.env.local
.env.production

# IDE/editor files
.vscode/settings.json
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*

# Test coverage
coverage/

# Database files
*.sqlite
```

### Ignoring Already-Tracked Files

If a file is already tracked, adding it to `.gitignore` will not stop tracking it:

```bash
# Remove from tracking (keep the file locally)
git rm --cached .env
git commit -m "chore: stop tracking .env"

# Now .gitignore will take effect for .env
```

---

## Best Practices

1. **Rebase local branches, merge shared branches** — keep your feature branch up to date with rebase, but merge into main.
2. **Squash messy commits** before merging — no one needs to see "fix typo" 12 times.
3. **Use `--force-with-lease`** instead of `--force` — prevents overwriting teammates' work.
4. **Tag releases** with annotated tags — makes rollbacks and changelogs easy.
5. **Use git stash** instead of half-baked "WIP" commits.
6. **Set up Husky + lint-staged** — automated quality checks that the whole team shares.
7. **Use `git revert` for shared branches** — never `reset --hard` on pushed commits.
8. **Keep reflog in mind** — if you mess up, you can almost always recover.
9. **Write meaningful commit messages** — `git bisect` and `git log` are only useful if messages describe what changed.
10. **Use `.gitignore` from day one** — never commit `node_modules`, `.env`, or build artifacts.

---

## Common Mistakes

| Mistake                                    | Why It Is a Problem                                | Fix                                             |
| ------------------------------------------ | -------------------------------------------------- | ----------------------------------------------- |
| Rebasing shared branches                   | Rewrites history others depend on                  | Only rebase local/unpushed branches             |
| `git reset --hard` without checking reflog | Might lose important work                          | Use `--soft` or `--mixed` when unsure           |
| Force pushing without `--force-with-lease` | Can overwrite teammates' commits                   | Always use `--force-with-lease`                 |
| Stashing and forgetting                    | Changes sit in stash forever                       | Use descriptive messages, clean stash regularly |
| Committing `.env` files                    | Secrets exposed in repo history                    | Add to `.gitignore` before first commit         |
| Not using hooks                            | Style/lint issues slip into commits                | Set up Husky + lint-staged                      |
| Giant rebase conflicts                     | Rebasing 50 commits is painful                     | Rebase frequently (keep branches short)         |
| Cherry-picking without noting it           | Same change exists in two places, causes confusion | Document cherry-picks in commit message         |

---

## Summary

- **Rebase** replays commits onto another branch for linear history — never rebase shared branches.
- **Interactive rebase** lets you squash, reword, reorder, and drop commits before sharing.
- **Cherry-pick** applies a specific commit to another branch without full merge.
- **Stash** saves uncommitted work temporarily — use `pop` to restore, `list` to review.
- **Bisect** uses binary search to find the exact commit that introduced a bug.
- **Reset** moves HEAD backward — `--soft` (keep staged), `--mixed` (keep unstaged), `--hard` (discard all).
- **Revert** safely undoes a commit by creating a new inverse commit (safe for shared branches).
- **Reflog** records all HEAD movements — your safety net for recovering "lost" work.
- **Tags** mark release points — use annotated tags for versioning.
- **Hooks** automate quality checks (linting, testing) at key Git events — use Husky to share them.
- **`.gitignore`** keeps unwanted files out of the repo — always set up before the first commit.
