# 08 — GitHub

## Git vs GitHub

**Git** is a version control system — a tool that tracks changes to files over time and lets multiple people work on the same codebase without overwriting each other's work.

**GitHub** is a hosting platform for Git repositories. It adds collaboration features on top of Git: pull requests, code review, issues, projects, and Actions (CI/CD — covered in module 09).

You can use Git without GitHub (there are alternatives like GitLab and Bitbucket), but GitHub is what we use.

---

## Core Git Concepts

### The Three Zones

Understanding where your changes live at any point is the key to avoiding confusion:

```
Working Directory    →    Staging Area    →    Repository (commits)
(files on disk)          (git add)             (git commit)
```

- **Working directory:** Your actual files. Changes here are "unstaged."
- **Staging area (index):** Changes you've marked for inclusion in the next commit.
- **Repository:** The permanent history of commits.

### Essential Commands

```bash
# See the state of your working directory and staging area
git status

# Stage a file
git add filename.ts

# Stage all changes
git add .

# Commit staged changes
git commit -m "Add task creation API route"

# See recent commits
git log --oneline

# See what changed in a file
git diff filename.ts

# See what's staged
git diff --staged
```

### Branches

A branch is a lightweight pointer to a commit. Branching is how we work on features in isolation without affecting the main codebase.

```bash
# List branches (current branch marked with *)
git branch

# Create and switch to a new branch
git checkout -b feature/add-task-form

# Switch to an existing branch
git checkout main

# Merge a branch into current branch
git merge feature/add-task-form

# Delete a branch (after it's merged)
git branch -d feature/add-task-form
```

### Remote

The **remote** (usually named `origin`) is the copy of the repo on GitHub.

```bash
# See configured remotes
git remote -v

# Push a branch to the remote
git push origin feature/add-task-form

# Pull changes from the remote
git pull origin main

# Fetch remote changes without merging
git fetch origin
```

### The Typical Workflow

```bash
# 1. Start from an up-to-date main
git checkout main
git pull origin main

# 2. Create a feature branch
git checkout -b feature/task-status-filter

# 3. Make changes, stage, and commit (repeat as needed)
git add .
git commit -m "Add status filter to task table"

# 4. Push your branch to GitHub
git push origin feature/task-status-filter

# 5. Open a Pull Request on GitHub
# 6. Get reviewed and approved
# 7. Merge into main
```

---

## Feature Branching

We use **feature branches** — one branch per feature, bug fix, or change. Never commit directly to `main`.

Branch naming conventions:
- `feature/add-task-form` — new functionality
- `fix/task-status-not-saving` — bug fix
- `chore/update-dependencies` — maintenance

### Why Not Commit to Main?

- `main` should always be deployable. In-progress work on a branch can't break it.
- Branches let you work in parallel with teammates without conflicts.
- Pull requests give the team a chance to review before changes land.

---

## Pull Requests

A Pull Request (PR) is a request to merge your branch into another (usually `main`). It's also the primary place for code review.

### Opening a PR

After pushing your branch:
1. Go to the repo on GitHub
2. Click the **"Compare & pull request"** banner, or go to **Pull requests → New pull request**
3. Set the base branch (`main`) and compare branch (your feature branch)
4. Write a title and description — explain *what* changed and *why*
5. Click **Create pull request**

### PR Description Best Practices

A good PR description includes:
- **What:** A summary of the change
- **Why:** The motivation (link to the issue if there is one)
- **How to test:** Steps for the reviewer to verify it works
- **Screenshots:** For UI changes

### Code Review

Reviewers can:
- **Comment** on specific lines
- **Request changes** — the author must address feedback before merging
- **Approve** — they're happy with the change

As an author, respond to every comment — either fix it or explain why you're not. Don't take feedback personally.

---

## Issues

GitHub Issues track work to be done — bugs, features, questions.

### Creating an Issue

1. Go to the repo → **Issues → New issue**
2. Write a clear title
3. Describe the problem or feature with enough context for anyone to pick it up
4. Add labels (`bug`, `enhancement`, `question`)
5. Assign to someone if appropriate

### Linking Issues to PRs

In a PR description or commit message, use keywords to auto-close issues on merge:

```
Closes #42
Fixes #17
Resolves #88
```

When the PR merges to `main`, issue #42 closes automatically.

---

## GitHub Projects

GitHub Projects is a kanban/board tool built on top of issues and PRs. We use it to track the status of work across a sprint or milestone.

Common columns:
- **Backlog** — Ideas and future work
- **Todo** — Prioritized, ready to start
- **In Progress** — Being worked on now
- **In Review** — PR open, awaiting review
- **Done** — Merged

Issues and PRs can be added to a project and moved between columns manually, or automatically via GitHub Actions.

---

## Useful Git Commands for Fixing Mistakes

```bash
# Undo the last commit but keep the changes staged
git reset --soft HEAD~1

# Undo the last commit and unstage the changes
git reset HEAD~1

# Discard all unstaged changes in a file (careful — irreversible)
git checkout -- filename.ts

# Stash changes temporarily (to switch branches with dirty state)
git stash
git stash pop

# See what a commit changed
git show <commit-hash>

# Find which commit introduced a bug (binary search through history)
git bisect start
```

---

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) to practice the full PR workflow, then [MINI_PROJECT.md](./MINI_PROJECT.md) to set up a GitHub Project for the Task Tracker.