---
layout: post
title: "claude worktree complete guide"
date: 2025-03-22
---

# Claude Code Worktrees: A Complete Guide

## What are worktrees?

When you run `claude --worktree my-feature` (or `claude -w my-feature`), Claude Code
creates an isolated copy of your repo in a separate directory with its own branch.
This lets you run multiple Claude sessions in parallel without them stepping on each
other's files.

Under the hood, it runs something like:

```
git worktree add .claude/worktrees/my-feature -b worktree-my-feature
```

This creates two things:

- **A directory** at `.claude/worktrees/my-feature/` — a full working copy of your files
- **A branch** called `worktree-my-feature` — branching off from wherever you currently are

Your main repo stays untouched on `main`. Each worktree is independent.

```
your-project/                          ← main branch, untouched
├── .claude/worktrees/
│   ├── my-feature/                    ← worktree 1 (branch: worktree-my-feature)
│   └── my-bugfix/                     ← worktree 2 (branch: worktree-my-bugfix)
```

---

## Part 1: Hands-on example with a merge conflict

This example creates a repo, runs two parallel sessions that edit the same file
in incompatible ways, merges them, and resolves the conflict.

### Step 1 — Create a test repo

```bash
mkdir worktree-demo && cd worktree-demo
git init

cat > app.py << 'EOF'
def greet(name):
    return f"Hello, {name}!"

def farewell(name):
    return f"Goodbye, {name}!"

def main():
    print(greet("World"))
    print(farewell("World"))

if __name__ == "__main__":
    main()
EOF

git add app.py
git commit -m "Initial commit: basic greeting app"
```

### Step 2 — Launch two Claude sessions in separate terminals

**Terminal 1 — the "formal" session:**

```bash
cd ~/worktree-demo
claude -w formal-greetings
```

Give Claude this prompt:

```
Refactor app.py so the greet() function returns a formal greeting like
"Good day, {name}. It is a pleasure to meet you." and the farewell()
function returns "Farewell, {name}. Until we meet again."
Also add a new function called introduce() that returns
"Allow me to introduce myself. I am your assistant."
Commit your changes when done.
```

**Terminal 2 — the "casual" session:**

```bash
cd ~/worktree-demo
claude -w casual-greetings
```

Give Claude this prompt:

```
Refactor app.py so the greet() function returns a casual greeting like
"Yo, what's up {name}!" and the farewell() function returns
"Later, {name}!"
Also add a new function called introduce() that returns
"Hey! I'm your buddy, nice to meet ya!"
Commit your changes when done.
```

Both sessions edit the same functions and both add `introduce()` with different
implementations. This guarantees a conflict.

### Step 3 — Exit both sessions

When each session finishes, exit with `/exit` or `Ctrl+C`.
Choose **"keep worktree"** when prompted.

Verify:

```bash
cd ~/worktree-demo
git worktree list
# Should show main + two worktrees
git log --oneline --all --graph
```

### Step 4 — Merge the first worktree (clean merge)

```bash
cd ~/worktree-demo
git merge worktree-formal-greetings --ff-only
```

This merges cleanly because `main` hasn't changed.

### Step 5 — Merge the second worktree (conflict)

```bash
git merge worktree-casual-greetings -m "Merge casual greetings"
```

This fails with:

```
CONFLICT (content): Merge conflict in app.py
```

You'll see conflict markers in the file:

```python
def greet(name):
<<<<<<< HEAD
    return f"Good day, {name}. It is a pleasure to meet you."
=======
    return f"Yo, what's up {name}!"
>>>>>>> worktree-casual-greetings
```

### Step 6 — Resolve the conflict with Claude

Start a new Claude session (not in a worktree):

```bash
cd ~/worktree-demo
claude
```

Prompt:

```
There are merge conflicts in app.py. Resolve them by combining both styles:
- greet(), farewell(), and introduce() should each accept a style parameter
  ("formal" or "casual") and return the appropriate version
- Update main() to demo both styles
- Then stage and commit with a message explaining the resolution
```

Claude will edit the file, remove conflict markers, and run:

```bash
git add app.py
git commit -m "Resolve merge conflict: combine formal and casual styles"
```

That `git add` + `git commit` is what completes a paused merge.

### Step 7 — Clean up

```bash
git worktree remove .claude/worktrees/formal-greetings
git branch -D worktree-formal-greetings
git worktree remove .claude/worktrees/casual-greetings
git branch -D worktree-casual-greetings
```

You need both commands per worktree because they clean up different things:

- `git worktree remove` — deletes the directory from disk
- `git branch -D` — deletes the branch name from Git's database

---

## Part 2: Using worktrees with a plan document

If you already have a plan and you're in a Claude session, you don't need to exit
and restart with `-w`. You can tell Claude to switch mid-session.

### Single task in a worktree

```
I want to implement the auth module from the plan. Work in a worktree for this.
```

Claude creates the worktree and moves into it right there.

### Parallel tasks using sub-agents

```
Here's my plan. Implement all three sections in parallel.
For each section, create a sub-agent working in its own worktree:
- worktree "auth" for Section 1 (authentication)
- worktree "api" for Section 2 (API endpoints)
- worktree "ui" for Section 3 (frontend components)
Each agent should commit when done.
```

Sub-agents are fire-and-forget — they do the work and report back.
They cannot ask you questions mid-task.

### Parallel tasks using agent teams (experimental)

If agents need to communicate with each other or ask you questions, use agent teams.
Enable them first:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Then:

```
Create an agent team to implement this plan.
One teammate on auth, one on API, one on frontend.
Have each work in its own worktree.
If anyone has questions about the plan, message me before guessing.
```

Teammates can message each other directly, share findings, and coordinate.
Use `Shift+Down` in the terminal to cycle between teammates and message them.

### Running dev servers in worktrees

If your project runs a server, each worktree needs its own port:

```
Implement the new dashboard feature in a worktree.
When testing, run the dev server on port 8001 (main is on 8000).
Don't commit the port change.
```

Best practice: make ports configurable via environment variable in your code:

```python
import os
port = int(os.environ.get("PORT", 8000))
```

Then each worktree just runs `PORT=8001 python server.py`.

### What gets merged?

Everything committed on the worktree branch. All added, modified, and deleted files.
There's no selectivity — it's a standard `git merge`.

To control what gets merged:

- Use `.gitignore` for generated files (sample outputs, build artifacts)
- Tell Claude what to commit and what not to
- Or use `git cherry-pick <hash>` to pick specific commits instead of merging the whole branch

---

## Part 3: The /merge-worktree command

Merging and cleaning up worktrees has a gotcha: if your shell is inside the worktree
directory when it gets deleted, Claude can't run any more commands. The session breaks.

### Setup

Save this file as `.claude/commands/merge-worktree.md` in your project:

```markdown
# Merge and clean up worktrees

## IMPORTANT: Do NOT run this from inside a worktree session

If the current CWD is inside a worktree (check with `git worktree list`),
STOP and tell the user:

> You're inside a worktree session. Please exit this session first,
> then start a new Claude session from the main repo root and run
> `/merge-worktree` from there.

Do not attempt to merge or remove a worktree while your CWD is inside it.
The shell will break once the directory is deleted.

---

## If running from the main repo root

If $ARGUMENTS is provided, treat it as worktree name(s) to merge (space-separated).
If no arguments, list all worktrees and ask which to merge.

### Steps for EACH worktree being merged

1. Confirm CWD is the main repo root (first entry in `git worktree list`).

2. Check if the worktree has uncommitted changes:

       git -C <worktree-path> status --porcelain

   If there are changes, ask whether to commit or discard. Do not proceed
   until resolved.

3. Merge the worktree branch:

       git merge <worktree-branch> --ff-only

   If fast-forward fails, ask whether to do a regular merge or rebase.
   Resolve any conflicts before continuing.

4. Remove the worktree AND delete the branch in a single command.
   Git requires the worktree to be removed before the branch can be deleted:

       git worktree remove <worktree-path> && git branch -D <worktree-branch>

5. If there are more worktrees to merge, repeat from step 2.

### After all worktrees are merged

Verify:

    git worktree list
    git branch
    git log --oneline -5

Confirm everything is merged and cleaned up.

### Rules

- NEVER run this while CWD is inside a worktree. Exit the session first.
- Always merge one worktree at a time. First is usually clean, later ones may conflict.
- If a merge has conflicts, resolve before moving to the next worktree.
- Worktree remove and branch delete must happen in one command to avoid shell death.
```

### Usage

After all worktree sessions are done and exited:

```bash
cd ~/your-project
claude
```

```
/merge-worktree                          # lists worktrees, asks which to merge
/merge-worktree auth                     # merges a specific worktree
/merge-worktree auth api ui              # merges multiple in order
```

### Also add to your CLAUDE.md

So Claude always follows the right order even outside the command:

```markdown
## Worktree Cleanup

When merging a worktree and cleaning up, always follow this order:
1. cd to the main repo root FIRST
2. Merge the worktree branch into main
3. Remove the worktree directory and delete the branch in one command:
   `git worktree remove <path> && git branch -D <branch>`

Never remove the worktree directory while your CWD is inside it.
Never try to delete a branch while its worktree still exists.
```

---

## Quick reference

| Command | What it does |
|---------|-------------|
| `claude -w <name>` | Create worktree + branch, start Claude in it |
| `git worktree list` | Show all worktrees and their branches |
| `git merge worktree-<name>` | Merge a worktree branch into current branch |
| `git worktree remove <path>` | Delete the worktree directory |
| `git branch -D worktree-<name>` | Delete the worktree branch |
| `git cherry-pick <hash>` | Merge specific commits only |
| `git merge --abort` | Cancel a merge with conflicts |

---

## Common gotchas

**"Shell CWD no longer exists"** — You tried to delete a worktree while Claude was
still sitting inside it. Always exit the session first, then merge from the main repo.

**"Branch is not fully merged"** — Git says `-d` won't work because the branch isn't
pushed to the remote. Use `-D` (capital D) — it's safe if you've already merged locally.

**"Cannot delete branch used by worktree"** — Git won't delete a branch that's checked
out in a worktree. Remove the worktree first, then delete the branch.

**Port conflicts** — Two servers can't use the same port. Use env vars to assign
different ports per worktree.

**Sub-agents can't ask questions** — They're fire-and-forget. Use agent teams instead
if you need communication between workers.

**CLAUDE.md loads once** — It's read at session start, not on every message. If you
edit it mid-session, tell Claude to re-read it.
