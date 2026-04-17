# 08 — GitHub: Exercises & Mini Project

---

## Exercise 1: Init, Commit, Push

If your `task-tracker` folder isn't already a git repo:

```bash
cd task-tracker
git init
git add .
git commit -m "Initial commit: task tracker scaffold"
```

Create a new **empty** repo on GitHub (no README, no .gitignore — you already have those), then push:

```bash
git remote add origin https://github.com/YOUR_USERNAME/task-tracker.git
git branch -M main
git push -u origin main
```

Visit your repo on GitHub. Confirm all files are there.

**Checkpoint:** You can initialize a repo, make a commit, and push to GitHub.

---

## Exercise 2: Feature Branch and Pull Request

Simulate adding a new feature via a feature branch.

```bash
# Start from main
git checkout main
git pull origin main

# Create a feature branch
git checkout -b feature/add-task-count-badge
```

Make a small change — add a task count badge to the nav in `app/app/layout.tsx` (or any minor visible change):

```tsx
// Just add a count number next to "Tasks" in the nav
<Link href="/tasks" className="hover:underline">
  Tasks <span className="text-xs text-gray-400">(4)</span>
</Link>
```

Commit and push:
```bash
git add .
git commit -m "Add task count badge to nav"
git push origin feature/add-task-count-badge
```

Now open a Pull Request on GitHub:
1. Go to your repo → **Pull requests → New pull request**
2. Set base: `main`, compare: `feature/add-task-count-badge`
3. Write a title: `Add task count badge to navigation`
4. Write a short description explaining what you changed
5. Click **Create pull request**

**Don't merge it yet** — you'll do that in Exercise 3.

**Checkpoint:** You can create a branch, make a commit, push, and open a PR.

---

## Exercise 3: Code Review Simulation

If you're doing this with a colleague, swap PRs and review each other's. If you're solo:

**As the reviewer:**
1. Open the PR on GitHub
2. Click the **Files changed** tab
3. Click the `+` icon next to a line to add a comment — try commenting: "Should this count be dynamic or hardcoded? Could we pull it from the API?"
4. Click **Review changes → Request changes** and submit

**As the author:**
1. Read the feedback
2. Either update the code to address it, or reply explaining your decision
3. Re-push if you made changes: `git push origin feature/add-task-count-badge`
4. Re-request review or resolve the conversation

**Approve and merge:**
1. Click **Review changes → Approve**
2. Click **Merge pull request → Confirm merge**
3. Delete the branch (GitHub will prompt you)

**Locally, clean up:**
```bash
git checkout main
git pull origin main
git branch -d feature/add-task-count-badge
```

**Checkpoint:** You've completed the full feature branch → PR → review → merge cycle.

---

## Exercise 4: Issues

Create three issues for future work on the Task Tracker:

1. **`Add task creation form`** — label: `enhancement`
   - Description: "Currently the + New Task button does nothing. Add a modal form that lets users create tasks with a title, description, and assignee."

2. **`Tasks page should support filtering by status`** — label: `enhancement`
   - Description: "The filter buttons in the UI are not wired up. Clicking 'In Progress' should filter the table."

3. **`Fix: task created_at shows UTC time, should be local time`** — label: `bug`
   - Description: "The createdAt column shows raw ISO timestamps. Format them to the user's local timezone."

Now try linking an issue to a branch. Create a branch and make a commit that references the issue:

```bash
git checkout -b fix/task-date-formatting
# Make a change...
git commit -m "Format task dates to local timezone — closes #3"
git push origin fix/task-date-formatting
```

Open a PR for it. In the PR description, add `Closes #3`. When you merge, issue #3 will close automatically.

**Checkpoint:** You can create issues, label them, and link them to PRs.

---

## Mini Project: Set Up a GitHub Project Board

Set up a GitHub Project for the Task Tracker to track onboarding progress.

### 1. Create the Project

1. Go to your GitHub profile → **Projects → New project**
2. Choose **Board** view
3. Name it "Task Tracker — Onboarding"
4. Create columns: **Backlog**, **In Progress**, **In Review**, **Done**

### 2. Add Issues to the Board

Add the three issues from Exercise 4 to the board. Place them in **Backlog**.

### 3. Simulate Moving Work

- Move "Add task creation form" to **In Progress**
- Create the feature branch: `git checkout -b feature/task-creation-form`
- Make a placeholder commit and push, open a PR
- Move the issue to **In Review** on the board

### 4. Add a Custom Field

In the Project settings, add a custom field:
- Field name: **Priority**
- Type: **Single select**
- Options: `High`, `Medium`, `Low`

Set priorities on your issues.

### 5. Try the Table View

Switch from Board view to **Table view**. Notice you can sort and filter by any field. This is useful for sprint planning.

---

## Useful Git Reference Card

Save this — you'll use it daily:

```bash
# Daily workflow
git status                          # Where am I?
git pull origin main               # Get latest
git checkout -b feature/my-thing   # New branch
git add .                          # Stage all
git commit -m "Clear message"      # Commit
git push origin feature/my-thing   # Push

# Reviewing changes
git log --oneline                  # Recent commits
git diff                           # Unstaged changes
git diff --staged                  # Staged changes
git show <hash>                    # What a commit changed

# Fixing mistakes
git reset --soft HEAD~1            # Undo last commit (keep changes staged)
git reset HEAD~1                   # Undo last commit (unstage changes)
git stash / git stash pop          # Temporarily shelve changes
```

---

## Reflection Questions

1. What's the difference between `git fetch` and `git pull`?
2. Why do we branch instead of committing directly to `main`?
3. What does `Closes #42` in a PR description do?
4. What's the difference between `git reset --soft HEAD~1` and `git reset HEAD~1`?
5. You're halfway through a feature and your teammate asks you to review their PR urgently. You have unstaged changes. What's the cleanest way to switch context?

---

Move on to [09 — GitHub Actions](../09-github-actions/README.md)!