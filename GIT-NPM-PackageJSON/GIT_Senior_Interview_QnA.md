# 🎯 Git Interview Q&A — Senior Full Stack Developer (5+ Years)

> 20 questions across 6 categories that test **internals**, **tradeoffs**, and **real-world judgment** — not just command syntax.

---

## 💡 What Separates Senior-Level Answers

| Trait | Junior | Senior (5+ years) |
|---|---|---|
| **Knowledge depth** | Knows the command | Explains *why* it works |
| **Tradeoffs** | One right answer | Context-aware decisions |
| **War stories** | "I think it works like..." | "I used this when..." |
| **Risk awareness** | Runs commands | Knows what can go wrong |

---

## 📌 Table of Contents

1. [Branching](#-category-1-branching)
2. [History](#-category-2-history)
3. [Workflow](#-category-3-workflow)
4. [Internals](#-category-4-internals)
5. [Collaboration](#-category-5-collaboration)
6. [Advanced](#-category-6-advanced)
7. [Quick Revision Table](#-quick-revision-table)

---

## 🌿 Category 1: Branching

---

### Q1. What is the difference between `git merge` and `git rebase`? When would you choose one over the other?

**Answer:**

```
MERGE — preserves full history with a merge commit
──────────────────────────────────────────────────

main:     A ── B ── C ──────────── M   (M = merge commit)
                    \             /
feature:             D ── E ── F

Result on main:  A ── B ── C ── M
                              ↑
                    history shows the branch existed


REBASE — rewrites commits to create a linear history
──────────────────────────────────────────────────────

main:     A ── B ── C
                    \
feature:             D ── E ── F

After git rebase main (on feature branch):

main:     A ── B ── C
                    \
feature:             D' ── E' ── F'   (commits rewritten)

Result: clean linear history, no merge commit
```

**When to choose:**

| Situation | Use |
|---|---|
| Merging a finished feature into main | `merge` (preserves history) |
| Updating feature branch with latest main | `rebase` (keeps history clean) |
| Public/shared branch | `merge` (never rewrite shared history) |
| Local-only feature branch | `rebase` (cleaner log) |

> ⚠️ **Golden Rule:** Never rebase commits already pushed to a shared remote — it rewrites history and causes divergence for others.

**Interview answer:**
> "Merge preserves full history including the branch structure, while rebase rewrites commits onto a new base creating a linear history. I use merge for integrating completed features into main so the history is honest and traceable. I use rebase for updating my local feature branch with the latest main to keep things clean before raising a PR. The key rule I follow: never rebase anything that's already been pushed and shared."

---

### Q2. Explain `git cherry-pick` and describe a real scenario where you'd use it.

**Answer:**

Cherry-pick applies a specific commit from one branch onto another using its SHA.

```bash
git cherry-pick <commit-sha>
git cherry-pick <sha> --no-commit    # stage only, don't commit
git cherry-pick <sha> -x             # append original SHA in message
```

**Diagram:**

```
feature-branch:  A ── B ── C ── D(hotfix) ── E
                              ↑
                         this commit fixes a critical bug

main:            A ── B ── C
                              \
          git cherry-pick D    D'   ← only this commit applied
                              ↑
                    same change, new SHA (D')
```

**Real scenario:**
> "A developer accidentally committed a production hotfix onto a feature branch instead of main. We couldn't merge the entire feature branch — it was unfinished. So we cherry-picked just that one commit onto main and deployed it. Saved us without releasing incomplete code."

---

## 📜 Category 2: History

---

### Q3. How do you find which commit introduced a bug? Explain `git bisect`.

**Answer:**

Git bisect does a binary search through commit history to find the exact commit that introduced a bug.

```bash
git bisect start
git bisect bad                    # current commit is broken
git bisect good <known-good-sha>  # last known working commit
# Git checks out midpoint → you test → mark good or bad
git bisect good   # or git bisect bad
# repeat until Git finds the culprit commit
git bisect reset  # return to original HEAD
```

**Diagram:**

```
Commits: A ── B ── C ── D ── E ── F ── G (current, broken)
                                       ↑
                              git bisect bad

Known good: A
         ↑
  git bisect good A

Round 1: Git checks out D (midpoint)
  → you test → broken → git bisect bad

Round 2: Git checks out B (midpoint of A-D)
  → you test → works  → git bisect good

Round 3: Git checks out C
  → you test → broken → git bisect bad

Result: "C is the first bad commit" ✓
```

**Automate with a script:**
```bash
git bisect run npm test           # auto-runs tests at each step
```

**Interview answer:**
> "Git bisect is incredibly powerful for bug hunting. It uses binary search so even with 1000 commits, you find the culprit in about 10 steps. I've automated it with git bisect run pointing to a test script — Git runs the tests at each step and finds the bad commit completely hands-free."

---

### Q4. What is the difference between `git reset`, `git revert`, and `git restore`?

**Answer:**

```
git reset   → moves the HEAD pointer (rewrites history)
git revert  → creates a NEW commit that undoes a previous one
git restore → discards working tree changes (no history touch)
```

**Diagram:**

```
RESET (rewrites history — dangerous on shared branches):
──────────────────────────────────────────────────────
Before:  A ── B ── C ── D  (HEAD)
After reset to B:  A ── B  (HEAD)   C and D are gone

REVERT (safe for shared branches):
──────────────────────────────────────────────────────
Before:  A ── B ── C ── D  (HEAD)
After revert C:  A ── B ── C ── D ── C'  (HEAD)
                                     ↑
                           new commit that undoes C

RESTORE (working directory only, no history change):
──────────────────────────────────────────────────────
git restore index.html    → discards unsaved changes in that file
git restore --staged .    → unstages files (keeps changes)
```

| Command | Touches history? | Safe on shared branch? | Use case |
|---|---|---|---|
| `git reset` | Yes — rewrites | ❌ No | Local cleanup |
| `git revert` | No — adds commit | ✅ Yes | Undo on shared/main |
| `git restore` | No | ✅ Yes | Discard file changes |

---

### Q5. How would you recover a commit accidentally deleted after a hard reset?

**Answer:**

Use `git reflog` — it logs every movement of HEAD, including resets, checkouts, and branch deletions.

```bash
git reflog                        # see all recent HEAD movements
# Output:
# abc1234 HEAD@{0}: reset: moving to origin/main
# def5678 HEAD@{1}: commit: feat: add login page  ← your lost commit
# ...

git checkout def5678              # go to lost commit
git branch recover-branch         # save it as a branch
# OR
git cherry-pick def5678           # apply it to current branch
```

**Diagram:**

```
Before reset:   A ── B ── C ── D (your commit)
                               ↑ HEAD

git reset --hard origin/main

After reset:    A ── B ── C
                           ↑ HEAD
                D is "gone" but still in object store!

git reflog shows: HEAD@{1} was at D
git checkout <D's sha> → you have it back ✓
```

> ⚠️ Reflog entries expire after **90 days** by default. After that, the object may be garbage collected.

---

## ⚙️ Category 3: Workflow

---

### Q6. Explain the Gitflow workflow. What are its limitations for modern CI/CD?

**Answer:**

```
GITFLOW BRANCH STRUCTURE:
──────────────────────────────────────────────────────────────

main     ──●──────────────────────────────●── (production)
            \                            /
hotfix        ●── fix ──────────────────●
                                       /
develop  ──●──────────────────────────●────── (integration)
            \          \             /
feature/A    ●──────────●           /
                  \                /
feature/B          ●──────────────●
                         \
release/1.0               ●────────● (release prep)
```

**Limitations for CI/CD:**

| Problem | Impact |
|---|---|
| Long-lived feature branches | Merge conflicts accumulate |
| Multiple branch levels | Overhead for small teams |
| Scheduled releases assumed | Doesn't fit continuous deployment |
| Slow integration | Features sit unintegrated for days/weeks |

**Modern alternative — Trunk-Based Development:**
```
main  ──●──●──●──●──●──●──  (deploy every commit)
         \  \  \
          short-lived branches (< 2 days)
          merged via PRs with feature flags
```

**Interview answer:**
> "Gitflow made sense for teams with scheduled releases — it's structured and predictable. But for CI/CD pipelines it creates friction: long-lived branches mean delayed integration and painful merges. Most modern teams I've worked with use trunk-based development with feature flags instead — branches live for hours or days, not weeks."

---

### Q7. What are Git hooks and how have you used them in a full-stack project?

**Answer:**

Git hooks are scripts that run automatically at specific Git lifecycle events, stored in `.git/hooks/`.

```
HOOK TRIGGER POINTS:
──────────────────────────────────────────────────────
pre-commit     → before commit is created
commit-msg     → validates commit message format
pre-push       → before push to remote
post-merge     → after a merge completes
post-checkout  → after switching branches
```

**Real full-stack usage:**

```bash
# pre-commit hook (via Husky + lint-staged)
# .husky/pre-commit
npx lint-staged
# runs ESLint + Prettier on staged files only

# commit-msg hook (via commitlint)
# .husky/commit-msg
npx commitlint --edit $1
# enforces: feat: / fix: / chore: / docs: format

# pre-push hook
npm run test:unit
# blocks push if tests fail
```

**Setup with Husky (shareable across team):**
```bash
npm install --save-dev husky lint-staged
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

---

### Q8. How do you handle long-running feature branches in a large team?

**Answer:**

```
THE PROBLEM:
──────────────────────────────────────────────────────
main:     ──●──●──●──●──●──●──●──●──  (8 new commits)
               \
feature:        ●─────────────────────  (2 weeks old)
                                        ↑
                              massive merge conflict ahead

SOLUTIONS:
──────────────────────────────────────────────────────
1. Keep branches short-lived (< 2–3 days)
2. Feature flags  → merge incomplete work safely, toggle in code
3. Daily rebase   → rebase onto main every morning
4. Sub-PRs        → split large feature into independent chunks
5. Code review SLA → enforce PR turnaround within 24 hours
```

**Feature flag example:**
```javascript
// Merge to main anytime — flag controls visibility
if (featureFlags.isEnabled('new-checkout-flow')) {
  return <NewCheckout />;
}
return <OldCheckout />;
```

---

## 🔬 Category 4: Internals

---

### Q9. How does Git store data internally? What are blobs, trees, and commits?

**Answer:**

Git is a **content-addressable file system**. Everything is stored as objects, identified by their SHA-1 hash.

```
THREE OBJECT TYPES:
──────────────────────────────────────────────────────

BLOB  → stores raw file content (no filename, no metadata)
        SHA: abc123 → "console.log('hello')"

TREE  → maps filenames to blob SHAs (like a directory listing)
        SHA: def456 → {
          "index.html" → blob:aaa111
          "app.js"     → blob:bbb222
          "src/"       → tree:ccc333
        }

COMMIT → points to a tree + parent commit(s) + metadata
         SHA: ghi789 → {
           tree:    def456       ← snapshot of project
           parent:  prev-commit-sha
           author:  "Dev <dev@co.com>"
           message: "feat: add login"
         }

DIAGRAM:
──────────────────────────────────────────────────────

commit ghi789
    │
    ├── tree def456
    │       ├── blob: index.html content
    │       ├── blob: app.js content
    │       └── tree: src/
    │               └── blob: login.js content
    │
    └── parent: prev-commit
```

**Why this matters:**
- Branches are just **pointers to commit SHAs** — creating a branch is instant
- Git detects corruption because any content change = different SHA
- `git diff` compares blob SHAs, not file contents line by line

---

### Q10. What is the difference between `HEAD`, `origin/HEAD`, and `FETCH_HEAD`?

**Answer:**

```
HEAD
──────────────────────────────────────────────────────
Points to your current local branch or commit.
Usually: HEAD → refs/heads/main → commit abc123

Detached HEAD: HEAD → commit abc123 directly (no branch)


origin/HEAD
──────────────────────────────────────────────────────
Remote-tracking pointer. Indicates the default branch
of the remote (origin). Updated when you run git fetch.

origin/HEAD → origin/main (usually)


FETCH_HEAD
──────────────────────────────────────────────────────
Records the branch/SHA last fetched from remote.
Created/updated every time you run git fetch.

Useful for:
  git fetch origin
  git merge FETCH_HEAD    ← merge what was just fetched


DIAGRAM:
──────────────────────────────────────────────────────

[Remote: GitHub]          [Local]
  main → C3               HEAD → main → C3
  HEAD → main             origin/main → C3
                          origin/HEAD → origin/main
                          FETCH_HEAD → C3 (after fetch)
```

---

### Q11. Explain how `git stash` works internally and what its limitations are.

**Answer:**

Git stash creates **two hidden commits** — one for the index and one for the working tree — stored under `refs/stash`.

```
INTERNAL STRUCTURE:
──────────────────────────────────────────────────────

refs/stash
    │
    └── stash commit (WIP on main: abc123 message)
            ├── parent 1: current HEAD
            ├── parent 2: index commit  (staged changes)
            └── parent 3: working tree commit (unstaged)

LIMITATIONS:
──────────────────────────────────────────────────────
✗ Local only — not pushed to remote
✗ Untracked files excluded (use git stash -u to include)
✗ Ignored files excluded (use git stash -a for all)
✗ Deep stash stacks cause confusion
✗ Conflicts on pop need manual resolution + git stash drop
```

**Better alternatives for complex cases:**
```bash
git stash -u              # include untracked files
git stash -m "wip: login feature"  # named stash
# OR just commit a WIP commit:
git commit -m "WIP: do not merge" && git push  # safer for long work
```

---

## 🤝 Category 5: Collaboration

---

### Q12. How do you handle merge conflicts in a CI/CD pipeline?

**Answer:**

```
PREVENTION (upstream):
──────────────────────────────────────────────────────
1. Short-lived branches (< 2 days)
2. Require rebase before PR merge
3. Branch protection rules (enforce up-to-date branches)
4. Merge drivers for auto-resolvable files

RESOLUTION (when it happens):
──────────────────────────────────────────────────────
git fetch origin
git rebase origin/main          # conflicts surface here
# fix conflicts in editor
git add .
git rebase --continue
git push --force-with-lease     # safe force push
```

**Auto-resolve package-lock.json conflicts** via `.gitattributes`:
```
# .gitattributes
package-lock.json merge=npm-merge-driver
```

---

### Q13. What is a detached HEAD state and how do you recover?

**Answer:**

```
NORMAL STATE:
──────────────────────────────────────────────────────
HEAD → main → commit C3
(HEAD points to a branch, branch points to commit)

DETACHED HEAD:
──────────────────────────────────────────────────────
HEAD → commit C3 directly  (no branch in between)

Happens when:
  git checkout <sha>         # checking out a commit
  git checkout v1.0.0        # checking out a tag
  during/after git rebase


DANGER:
──────────────────────────────────────────────────────
main:  A ── B ── C
                 ↑ HEAD (detached)
                  \
                   D ── E   ← new commits in detached state

When you git checkout main:
main:  A ── B ── C
                 ↑ HEAD
       D and E are now ORPHANED (unreachable, will be GC'd)


RECOVERY:
──────────────────────────────────────────────────────
# While still detached (before switching):
git branch save-my-work       # create branch to preserve D, E

# After switching away (use reflog):
git reflog                    # find SHA of E
git branch save-my-work <sha> # recreate branch
```

---

### Q14. How would you squash commits before a PR merge and why is it important?

**Answer:**

```bash
# Interactive rebase — squash last N commits
git rebase -i HEAD~4

# In the editor:
pick abc1234 feat: start login page
squash def5678 wip: continue login
squash ghi9012 fix typo
squash jkl3456 fix tests

# → becomes one clean commit: "feat: add login page"
```

**Why it matters:**

```
WITHOUT squash (noisy main history):
──────────────────────────────────────────────────────
main: A ── B ── wip ── fix typo ── more wip ── fix tests ── C

WITH squash (clean main history):
──────────────────────────────────────────────────────
main: A ── B ── feat: add login page ── C
```

> Teams using `semantic-release` or auto-changelog tools **require** clean, conventional commit messages — squashing enables this.

---

### Q15. How would you enforce commit message standards across a team?

**Answer:**

**Conventional Commits format:**
```
feat: add OAuth login
fix: resolve token expiry bug
chore: upgrade webpack to v5
docs: update API readme
BREAKING CHANGE: remove legacy auth endpoint
```

**Setup:**
```bash
npm install --save-dev husky @commitlint/cli @commitlint/config-conventional

# commitlint.config.js
module.exports = { extends: ['@commitlint/config-conventional'] }

# .husky/commit-msg
npx commitlint --edit $1
```

**CI enforcement for PR titles (GitHub Actions):**
```yaml
- uses: amannn/action-semantic-pull-request@v5
```

**Payoff — automated changelog:**
```bash
npx semantic-release   # auto-bumps version + generates CHANGELOG.md
```

---

## 🚀 Category 6: Advanced

---

### Q16. What is `git worktree` and when would you use it?

**Answer:**

`git worktree` lets you check out **multiple branches simultaneously** in different folders — from the same repo, without cloning.

```bash
git worktree add ../hotfix-branch hotfix/payment-bug
# now you have:
# /project          → your current branch (feature/login)
# /hotfix-branch    → hotfix/payment-bug checked out here
```

**Diagram:**

```
Single .git directory — multiple working trees:
──────────────────────────────────────────────────────

/my-project/          (branch: feature/login)
    ├── .git/          ← shared Git brain
    ├── src/
    └── ...

/my-project-hotfix/   (branch: hotfix/payment-bug)
    ├── src/           ← independent working tree
    └── ...            (no .git — shares parent's)
```

**Use cases:**
- Reviewing a PR while continuing your own work (no stashing)
- Running tests on another branch simultaneously
- Comparing two versions of the app side-by-side

---

### Q17. Explain shallow clone and when it's useful in CI/CD.

**Answer:**

```bash
git clone --depth=1 https://github.com/user/repo.git
# fetches only the latest commit — no full history
```

**Diagram:**

```
FULL CLONE:
──────────────────────────────────────────────────────
A ── B ── C ── D ── E ── F ── G  (all 5 years of history)
↑ downloads everything → slow CI startup

SHALLOW CLONE (--depth=1):
──────────────────────────────────────────────────────
                              G  (only latest commit)
↑ downloads only what's needed → fast CI startup


TRADEOFFS:
──────────────────────────────────────────────────────
✓ Much faster clone in CI (seconds vs minutes)
✓ Less disk/bandwidth usage
✗ Can't use git bisect beyond shallow depth
✗ Can't use git log / git blame on old history
✗ Some operations need --unshallow to convert
```

**GitHub Actions default:**
```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 1    # shallow by default
    # fetch-depth: 0  # full history when you need it
```

---

### Q18. How do submodules work and what are the common pitfalls?

**Answer:**

Submodules embed another repo at a **fixed commit** inside a parent repo.

```bash
git submodule add https://github.com/org/lib.git libs/lib
git submodule update --init --recursive   # after clone
```

**Diagram:**

```
parent-repo/
    ├── .gitmodules     ← stores submodule URL + path
    ├── src/
    └── libs/
        └── lib/        ← points to commit SHA abc123
                           (not the full history, just a pointer)

Parent repo stores: "libs/lib is at commit abc123"
NOT the actual files — just the reference.
```

**Common pitfalls:**

| Pitfall | What happens | Fix |
|---|---|---|
| Forget `--recursive` on clone | Submodule folder is empty | `git submodule update --init --recursive` |
| Commit in submodule, forget parent | Parent still points to old SHA | `cd parent && git add libs/lib && git commit` |
| Teammate pulls, gets old submodule | Confusing errors | Always run `git submodule update` after pull |

> Many teams avoid submodules entirely in favor of **package managers** (npm, pip) or **monorepos**.

---

### Q19. What is `git gc` and how does it affect repository performance?

**Answer:**

`git gc` (garbage collection) cleans up and compresses the repository's internal object store.

```bash
git gc              # standard cleanup
git gc --aggressive # deeper compression (slower, run rarely)
git gc --auto       # only runs if threshold of loose objects met
```

**What it does:**

```
BEFORE gc:
──────────────────────────────────────────────────────
.git/objects/
    ├── ab/cd1234...   ← loose object
    ├── ef/gh5678...   ← loose object
    ├── ij/kl9012...   ← loose object
    └── (thousands more individual files)

AFTER gc:
──────────────────────────────────────────────────────
.git/objects/
    └── pack/
        ├── pack-abc123.pack   ← all objects compressed together
        └── pack-abc123.idx    ← index for fast lookup

Results:
  ✓ Smaller disk usage
  ✓ Faster git operations
  ✓ Removes unreachable/orphaned objects (after grace period)
```

> GitHub, GitLab, and Bitbucket run gc server-side automatically. You rarely need to run it manually unless working with a very large local repo.

---

### Q20. How would you move a folder from one Git repo to another while preserving commit history?

**Answer:**

Use `git filter-repo` (modern replacement for deprecated `filter-branch`):

```bash
# Step 1: Clone the source repo
git clone https://github.com/org/source-repo.git
cd source-repo

# Step 2: Keep only the folder you want (rewrites history)
git filter-repo --path src/my-module/

# Step 3: In the target repo, add source as a remote
cd ../target-repo
git remote add source-temp ../source-repo
git fetch source-temp

# Step 4: Merge with unrelated histories allowed
git merge source-temp/main --allow-unrelated-histories

# Step 5: Clean up
git remote remove source-temp
```

**Diagram:**

```
source-repo history (before filter):
──────────────────────────────────────────────────────
A(all files) ── B(all files) ── C(all files)

After git filter-repo --path src/my-module/:
──────────────────────────────────────────────────────
A(module only) ── B(module only) ── C(module only)
↑ only commits that touched this folder, preserved ✓

After merge into target-repo:
──────────────────────────────────────────────────────
target history + module history merged
all commit authors, dates, messages intact ✓
```

---

## 📋 Quick Revision Table

| # | Question | Core concept to nail |
|---|---|---|
| 1 | merge vs rebase | Linear vs preserved history + golden rule |
| 2 | cherry-pick | Apply single commit across branches |
| 3 | git bisect | Binary search for bug-introducing commit |
| 4 | reset vs revert vs restore | History-safe vs destructive |
| 5 | recover lost commit | git reflog + 90 day expiry |
| 6 | Gitflow limitations | Long-lived branches vs trunk-based |
| 7 | Git hooks | pre-commit, commit-msg, pre-push + Husky |
| 8 | Long-running branches | Feature flags + daily rebase |
| 9 | Git internals | Blob, tree, commit objects + SHA |
| 10 | HEAD vs FETCH_HEAD | Three types of HEAD pointers |
| 11 | stash internals | Two hidden commits, local only |
| 12 | CI/CD conflicts | Prevention + merge drivers |
| 13 | Detached HEAD | Recovery via reflog + branch |
| 14 | Squash commits | Clean history + semantic-release |
| 15 | Commit standards | Conventional Commits + commitlint |
| 16 | git worktree | Multiple branches, one repo |
| 17 | Shallow clone | CI speed vs history tradeoffs |
| 18 | Submodules | Fixed SHA pointer + pitfalls |
| 19 | git gc | Object packing + performance |
| 20 | Move folder with history | git filter-repo workflow |

---

## 🎤 Interview Answer Formula

For any Git question, structure your answer like this:

```
1. DEFINITION  → one sentence, what it does
2. INTERNALS   → why/how it works under the hood
3. EXAMPLE     → real scenario from your experience
4. TRADEOFFS   → what can go wrong, when NOT to use it
```

**Example for git rebase:**
> "Rebase rewrites commits onto a new base, creating a linear history **(definition)**. Internally it replays each commit as a new object with a new SHA **(internals)**. I use it daily to update my feature branch with latest main before raising a PR **(example)**. The key risk is never rebasing commits already pushed to a shared branch — it rewrites history and forces teammates to deal with diverged refs **(tradeoff)**."

---

*Level: Senior Full Stack Developer — 5+ years | Focus: Internals, Tradeoffs, Real-world judgment*
