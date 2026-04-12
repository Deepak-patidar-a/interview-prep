# 🧠 Git Interview Guide — 5+ Years Full Stack Developer

> A complete revision guide covering core Git concepts with diagrams, interview answers, and real-world context.

---

## 📌 Table of Contents

1. [git init](#1-git-init)
2. [git clone](#2-git-clone--cloning-a-branch)
3. [git pull vs git push](#3-git-pull-vs-git-push)
4. [git reset --hard origin/main](#4-git-reset---hard-originmain)
5. [git stash](#5-git-stash)
6. [git stash pop vs git stash apply](#6-git-stash-pop-vs-git-stash-apply)
7. [Quick Reference Cheat Sheet](#7-quick-reference-cheat-sheet)
8. [Interview Tips](#8-interview-tips)

---

## 1. `git init`

### What it does
Creates a brand new, empty Git repository in your current folder by setting up a hidden `.git/` directory.

### Diagram

```
Your Folder (before)          Your Folder (after git init)
─────────────────────         ──────────────────────────────
 index.html                    index.html
 app.js                        app.js
                               .git/           ← Git brain lives here
                                 ├── HEAD
                                 ├── config
                                 ├── objects/
                                 └── refs/
```

### Command
```bash
mkdir my-project
cd my-project
git init
```

### git init vs git clone

```
git init                          git clone
─────────────────────────         ──────────────────────────────
Starting from SCRATCH             Repo ALREADY EXISTS remotely
No remote by default              Remote (origin) set automatically
No history                        Full history included
You add remote manually           Ready to pull/push immediately

  [Empty Folder]                    [GitHub / GitLab]
       │                                   │
   git init                            git clone
       │                                   │
  [Local Repo]                      [Local Copy]
  (no remote)                       (origin = remote)
```

### How to answer in interview
> "git init initializes a new Git repository locally — it creates the .git directory which is Git's internal storage for all objects, refs, and config. I use it when starting a fresh project that isn't on any remote yet. After init, I'd typically do git remote add origin <url> to connect it to GitHub."

---

## 2. `git clone` — Cloning a Branch

### What it does
Downloads a copy of an existing remote repository to your local machine.

### Clone full repo (then switch branch)
```bash
git clone https://github.com/user/repo.git
cd repo
git switch branch-name         # switch to your branch
```

### Clone a specific branch directly
```bash
git clone -b branch-name https://github.com/user/repo.git
cd repo
```

### Start working — full flow
```bash
git checkout -b my-feature     # create your own branch
# ... make changes to files ...
git status                     # see what changed
git add .                      # stage all changes
git commit -m "feat: add login page"
git push origin my-feature     # push to remote
```

### Diagram — Clone & Work Flow

```
[GitHub Remote]
      │
  git clone
      │
      ▼
[Local: main branch]
      │
  git checkout -b my-feature
      │
      ▼
[Local: my-feature branch]   ← you work here
      │
  edit files → git add → git commit
      │
  git push origin my-feature
      │
      ▼
[GitHub: my-feature branch]  ← now visible to team
```

### How to answer in interview
> "I use git clone to get a copy of the remote repo. If I need a specific branch, I use -b flag. After cloning, I always create my own feature branch with git checkout -b rather than working directly on main or develop. Then the normal cycle is: edit → git add → git commit → git push."

---

## 3. `git pull` vs `git push`

### git pull — Remote → Local
Fetches latest changes from remote and merges them into your current branch.

```bash
git pull origin main
```

Internally it runs:
```bash
git fetch origin      # step 1: download changes
git merge origin/main # step 2: merge into local
```

### git push — Local → Remote
Uploads your local commits to the remote repository.

```bash
git push origin my-feature
```

### Diagram

```
┌─────────────────────────────────────────────────┐
│                                                 │
│   [Remote: GitHub]          [Local Machine]     │
│                                                 │
│   main branch               main branch        │
│   commit A                  commit A            │
│   commit B         ◄──────  commit B            │
│   commit C       git push   commit C  ← yours  │
│                                                 │
│   main branch               main branch        │
│   commit A                  commit A            │
│   commit B         ──────►  commit B            │
│   commit C       git pull   commit C  ← theirs │
│                                                 │
└─────────────────────────────────────────────────┘

  git push  →  You share YOUR work with team
  git pull  ←  You get TEAM's work on your machine
```

### How to answer in interview
> "git pull keeps my local branch in sync with remote — it's fetch + merge combined. git push shares my committed work with the team. I always pull before starting new work to avoid conflicts, and push after committing to make my changes visible for code review."

---

## 4. `git reset --hard origin/main`

### What it does
Throws away ALL your local changes and resets your branch to exactly match the remote. This is **destructive and irreversible** (without reflog).

### ⚠️ Safe Usage (always fetch first)
```bash
git fetch origin
git reset --hard origin/main
```

### Diagram

```
Before reset:
─────────────────────────────────────────
Local:   A ── B ── C ── D(your changes)
Remote:  A ── B ── C
                   ↑
               origin/main

After git reset --hard origin/main:
─────────────────────────────────────────
Local:   A ── B ── C        ← D is GONE
Remote:  A ── B ── C
                   ↑
               HEAD now here

⚠️  Commit D and all uncommitted changes are DELETED
```

### When to use it
- Your local branch is completely messed up
- You want to start fresh from the remote state
- During a botched rebase or merge

### How to answer in interview
> "git reset --hard origin/main is a nuclear option — it discards all local commits and changes, resetting to exactly what's on the remote. I always run git fetch first to make sure origin/main is updated, and I only use this when I'm certain I want to throw away local work. For safety, I usually git stash first as a backup."

---

## 5. `git stash`

### What it does
Temporarily saves your uncommitted changes (staged + unstaged) into a stack, giving you a clean working directory.

### Basic commands
```bash
git stash              # save changes
git stash pop          # restore + delete stash
git stash apply        # restore, keep stash
git stash list         # see all stashes
git stash drop         # delete a stash manually
```

### Safe reset workflow using stash
```bash
git stash                        # 1. save your work safely
git fetch origin
git reset --hard origin/main     # 2. reset safely (nothing to lose)
git stash pop                    # 3. bring your work back
```

### Diagram

```
Working Directory                    Stash Stack
─────────────────                    ─────────────────────────
 edited files                        (empty)
 staged changes
      │
  git stash
      │
      ▼
 clean directory   ──────────────►  stash@{0}: WIP on main
      │
  ... do reset, switch branch, etc ...
      │
  git stash pop
      │
      ▼
 your changes are back ◄────────── stash@{0} (removed)
```

### How to answer in interview
> "git stash is like a clipboard for uncommitted work. I use it when I need to quickly switch context — like an urgent hotfix comes in while I'm mid-feature. I stash my work, fix the bug, push it, then pop my stash and resume. It's also essential before running git reset --hard so you don't lose anything."

---

## 6. `git stash pop` vs `git stash apply`

### The one key difference

```
git stash pop    →  Restore changes  +  DELETE stash entry
git stash apply  →  Restore changes  +  KEEP stash entry
```

### Commands
```bash
git stash pop                  # restore + remove from stash list
git stash apply                # restore + keep in stash list
git stash apply stash@{1}      # apply a specific stash
git stash drop                 # manually delete after apply
```

### Diagram

```
Stash Stack before:
┌─────────────────────────┐
│ stash@{0}: WIP feature  │
│ stash@{1}: WIP bugfix   │
└─────────────────────────┘

After git stash pop:            After git stash apply:
┌─────────────────────────┐     ┌─────────────────────────┐
│ stash@{0}: WIP bugfix   │     │ stash@{0}: WIP feature  │  ← still here
│ (stash@{0} was removed) │     │ stash@{1}: WIP bugfix   │
└─────────────────────────┘     └─────────────────────────┘
  Changes restored ✓              Changes restored ✓
  Stash entry gone ✓              Stash entry preserved ✓
```

### Real-world use case for `apply`
```bash
# Apply same changes to multiple branches
git stash apply              # apply to branch-1
git checkout branch-2
git stash apply              # apply same stash to branch-2
git stash drop               # clean up manually when done
```

### When to use which

| Situation | Use |
|---|---|
| Normal daily use, done with the stash | `git stash pop` |
| Applying same changes to multiple branches | `git stash apply` |
| Not sure if restore will cause conflicts | `git stash apply` (safer) |
| Want to test before committing to the restore | `git stash apply` |

### Handling conflicts after stash pop/apply
```bash
git stash pop          # conflict happens in same file
# → manually resolve the conflicts in your editor
git add .              # mark as resolved
git stash drop         # clean up the stash entry
```

### How to answer in interview
> "Both restore stashed changes, but pop deletes the stash entry after restoring while apply keeps it. I use pop for everyday use since I rarely need the stash again. I reach for apply when I want to apply the same stash to multiple branches, or when I'm unsure if a conflict might occur and I want the stash preserved as a safety net."

---

## 7. Quick Reference Cheat Sheet

```
┌────────────────────────────────────────────────────────────────┐
│                    GIT COMMAND CHEAT SHEET                     │
├─────────────────────┬──────────────────────────────────────────┤
│ git init            │ Create new local repo                    │
│ git clone <url>     │ Copy remote repo locally                 │
│ git clone -b <br>   │ Clone specific branch                    │
├─────────────────────┼──────────────────────────────────────────┤
│ git status          │ See changed/staged files                 │
│ git add .           │ Stage all changes                        │
│ git commit -m "msg" │ Save snapshot locally                    │
├─────────────────────┼──────────────────────────────────────────┤
│ git pull            │ Remote → Local (fetch + merge)           │
│ git push            │ Local → Remote                           │
├─────────────────────┼──────────────────────────────────────────┤
│ git stash           │ Save work temporarily                    │
│ git stash pop       │ Restore + delete stash                   │
│ git stash apply     │ Restore + keep stash                     │
│ git stash list      │ See all stashes                          │
│ git stash drop      │ Delete a stash manually                  │
├─────────────────────┼──────────────────────────────────────────┤
│ git fetch origin    │ Download remote changes (no merge)       │
│ git reset --hard    │ ⚠️  Discard all local changes            │
│ git reflog          │ Recover lost commits                     │
└─────────────────────┴──────────────────────────────────────────┘
```

### Data flow overview

```
[Your Editor]
     │
  edit files
     │
     ▼
[Working Directory]  ──── git stash ────►  [Stash Stack]
     │                                          │
  git add                                  git stash pop/apply
     │                                          │
     ▼                                          │
[Staging Area]  ◄──────────────────────────────┘
     │
  git commit
     │
     ▼
[Local Repository]  ◄──── git pull ────  [Remote Repository]
     │                                    (GitHub/GitLab)
  git push
     │
     ▼
[Remote Repository]
```

---

## 8. Interview Tips

### General approach for any Git question

1. **Say what it does** — one sentence definition
2. **Show you know the internals** — why it works that way
3. **Give a real scenario** — when you actually used it
4. **Mention the tradeoffs/risks** — shows senior awareness

### Common follow-up questions to prepare for

| They ask... | They want to know... |
|---|---|
| "What's the difference between fetch and pull?" | You know pull = fetch + merge |
| "When would you NOT use reset --hard?" | You know it's destructive on shared branches |
| "How do you recover a lost commit?" | You know about git reflog |
| "What happens if stash pop causes a conflict?" | You know to resolve + git stash drop |
| "How do you apply stash to multiple branches?" | You know git stash apply |

### Red flags to avoid in interviews
- Saying "I just Google the commands" without understanding internals
- Not knowing the difference between fetch and pull
- Not mentioning risks of destructive commands like reset --hard
- Confusing stash pop and stash apply

---

*Last updated: 2026 | Level: 5+ years Full Stack Developer*
