# 09 — GitHub Actions: Mini Project

## Task Tracker — Full CI/CD Pipeline

Build a complete pipeline: CI on every PR, plus a deployment workflow that triggers on merge to `main`. By the end, every PR is automatically validated and every merge to main triggers a build that could be handed off to any deployment target.

---

## What You're Building

```
PR opened/updated
       ↓
  CI Workflow
  ├── lint (parallel)
  ├── typecheck (parallel)
  └── build (after lint + typecheck pass)
       ↓
  ✅ PR can merge

Merge to main
       ↓
  Deploy Workflow
  ├── Build Docker image
  ├── Tag with commit SHA
  └── (Push to registry / deploy — stubbed for now)
```

---

## Step 1: CI Workflow (already done in exercises)

If you completed Exercise 5, you already have `.github/workflows/ci.yml`. Make sure it's in place.

---

## Step 2: Add a Lint Config

Make sure ESLint is configured in `app/.eslintrc.json`:

```json
{
  "extends": ["next/core-web-vitals"]
}
```

And TypeScript strict mode is on in `app/tsconfig.json`:
```json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

Run locally and fix any issues before continuing:
```bash
cd app && npm run lint && npm run typecheck
```

---

## Step 3: Deploy Workflow

**`.github/workflows/deploy.yml`**
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    name: Build Docker image
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./app
          push: false        # Set to true when you have a registry configured
          tags: |
            task-tracker:latest
            task-tracker:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Deployment summary
        run: |
          echo "## Deployment Summary" >> $GITHUB_STEP_SUMMARY
          echo "- **Commit:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Branch:** ${{ github.ref_name }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Triggered by:** ${{ github.actor }}" >> $GITHUB_STEP_SUMMARY
          echo "- **Status:** Image built successfully ✅" >> $GITHUB_STEP_SUMMARY
```

> The `$GITHUB_STEP_SUMMARY` trick writes a markdown summary to the Actions job summary page — a great way to surface key info without digging through logs.

Push to main and check the Actions tab. You'll see both the CI and Deploy workflows run. The deploy workflow builds the Docker image (but doesn't push it anywhere yet — add your registry credentials to Secrets when you have a real deploy target).

---

## Step 4: Add a PR Comment with Build Status

Add a step to the CI workflow that comments on the PR when the build succeeds:

**In `.github/workflows/ci.yml`**, add a final job:

```yaml
  comment:
    name: PR Comment
    runs-on: ubuntu-latest
    needs: [build]
    if: github.event_name == 'pull_request'
    permissions:
      pull-requests: write
    steps:
      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `✅ CI passed for commit \`${{ github.sha }}\`\n\n- Lint: passed\n- Type check: passed\n- Build: passed`
            })
```

Open a new PR — CI will post a comment when it passes.

---

## Step 5: Protect Main

If you haven't already (from Exercise 4 in the exercises):

1. Repo → **Settings → Branches → Add rule**
2. Branch: `main`
3. ✅ Require status checks: add `Lint`, `Type check`, `Build`
4. ✅ Require branches to be up to date
5. ✅ Require pull request reviews before merging (set to 1 approval)
6. Save

Now `main` is fully protected — nothing lands without CI passing and a code review.

---

## Step 6: Workflow for Dependency Updates (Bonus)

Add a weekly scheduled workflow that checks for outdated dependencies and opens an issue:

**`.github/workflows/deps-check.yml`**
```yaml
name: Dependency Check

on:
  schedule:
    - cron: "0 9 * * 1"  # Every Monday at 9am UTC
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Check for outdated packages
        working-directory: app
        run: npm outdated || true  # Don't fail if outdated packages exist

      - name: Run npm audit
        working-directory: app
        run: npm audit --audit-level=high
```

---

## What You've Built

Here's the complete picture of what happens on your Task Tracker repo now:

| Event | What Runs |
|-------|-----------|
| PR opened or updated | CI: lint + typecheck + build in parallel |
| All CI jobs pass | Comment posted on PR, merge button unlocks |
| Merge to `main` | Docker image built and tagged with commit SHA |
| Every Monday 9am | Dependency audit runs |

---

## The Full Task Tracker at a Glance

Looking back at all 9 modules, here's the full stack you've built and understand:

```
GitHub Actions (CI/CD)
     ↓ deploys
GitHub (source control + PR workflow)
     ↓ contains
task-tracker/
  docker-compose.yml
  ├── nginx (reverse proxy on :80)
  │     ↓ proxies to
  ├── app (Next.js on :3000)
  │     ├── app/ (App Router pages + API routes)
  │     ├── lib/prisma.ts (DB client)
  │     ├── lib/storage.ts (MinIO/S3 client)
  │     └── components/ (Tailwind-styled UI)
  ├── db (PostgreSQL on :5432)
  │     └── managed by Prisma migrations
  └── minio (object storage on :9000)
```

Congratulations — you've completed the onboarding track!

---

## Reflection Questions

1. What's the difference between CI and CD?
2. Why do we pass a dummy `DATABASE_URL` during the build step in CI?
3. What does `needs: [lint, typecheck]` do on the `build` job?
4. What's the risk of merging to `main` directly without branch protection?
5. Where would you store a real deploy token or database password used in a workflow?