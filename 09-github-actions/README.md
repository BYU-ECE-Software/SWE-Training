# 09 — GitHub Actions

## What is CI/CD?

**CI (Continuous Integration)** means automatically running tests, linting, and type checks every time someone pushes code or opens a PR. The goal is to catch problems immediately — before they land in `main`.

**CD (Continuous Deployment/Delivery)** means automatically deploying the app when a change is merged to `main`. No manual steps, no "remember to deploy" — it just happens.

GitHub Actions is the CI/CD platform built into GitHub. We define pipelines (called **workflows**) as YAML files in `.github/workflows/`.

---

## Core Concepts

### Workflow

A workflow is a YAML file that defines a pipeline. It lives in `.github/workflows/`. A repo can have multiple workflows.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint-and-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
```

### Triggers (`on`)

What causes the workflow to run:

```yaml
on:
  push:
    branches: [main]           # Any push to main
  pull_request:
    branches: [main]           # Any PR targeting main
  workflow_dispatch:           # Manual trigger from GitHub UI
  schedule:
    - cron: "0 9 * * 1"       # Every Monday at 9am UTC
```

### Jobs

A workflow contains one or more **jobs**. Jobs run in parallel by default. Each job runs on a fresh virtual machine.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  test:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]   # Wait for lint and test to pass first
    steps: [...]
```

### Steps

Each job has **steps** — sequential commands. Steps can be:
- **Actions** (`uses`): reusable community-built steps from the Actions marketplace
- **Run commands** (`run`): shell commands

```yaml
steps:
  - uses: actions/checkout@v4          # Check out the repo code
  - uses: actions/setup-node@v4        # Install Node.js
    with:
      node-version: 18
      cache: "npm"                     # Cache node_modules for speed
  - run: npm ci                        # Install dependencies
  - run: npm run lint                  # Run a command
  - run: npm run build
```

### Secrets

Sensitive values (API keys, deploy tokens) are stored in GitHub **Secrets** (repo → Settings → Secrets and variables → Actions) and referenced in workflows:

```yaml
- name: Deploy
  env:
    DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
  run: ./scripts/deploy.sh
```

Secrets are never exposed in logs — GitHub masks them.

### Contexts and Expressions

You can use expressions to access information about the run:

```yaml
# Only run on the main branch
if: github.ref == 'refs/heads/main'

# Access the commit SHA
run: echo "Deploying commit ${{ github.sha }}"

# Access an environment variable
run: echo "Running as ${{ github.actor }}"
```

---

## A Complete CI Workflow for Next.js

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  ci:
    name: Lint, typecheck, and build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "npm"
          cache-dependency-path: app/package-lock.json

      - name: Install dependencies
        working-directory: app
        run: npm ci

      - name: Run ESLint
        working-directory: app
        run: npm run lint

      - name: Run TypeScript type check
        working-directory: app
        run: npx tsc --noEmit

      - name: Build
        working-directory: app
        run: npm run build
        env:
          # Provide a dummy DATABASE_URL so the build doesn't fail
          DATABASE_URL: "postgresql://dummy:dummy@localhost:5432/dummy"
```

This runs on every PR and push to main. If any step fails, the workflow fails and GitHub shows a red ✗ on the PR — reviewers know not to merge.

---

## Status Checks

In GitHub repo settings, you can mark specific workflows as **required status checks**. This means a PR cannot be merged unless the CI workflow passes. This is how you enforce code quality as a team policy, not just a guideline.

Set it up: repo → Settings → Branches → Branch protection rules → `main` → Require status checks to pass before merging.

---

## Helpful (optional, but encouraged) videos
[Fireship CI/CD in 100 Seconds](https://www.youtube.com/watch?v=scEDHsr3APg)
[Fireship Github Actions](https://www.youtube.com/watch?v=eB0nUzAI7M8)

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) to write workflows for the Task Tracker, then [MINI_PROJECT.md](./MINI_PROJECT.md) to build a full CI/CD pipeline.

## Tasteful Memes
<img src="../memes/GitHubActionsMeme1.webp" width="400" height="400">
