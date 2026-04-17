# 09 — GitHub Actions: Exercises

Make sure your `task-tracker` repo is pushed to GitHub before starting.

---

## Exercise 1: Your First Workflow

Create the workflows directory:
```bash
mkdir -p .github/workflows
```

**`.github/workflows/hello.yml`**
```yaml
name: Hello World

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: Print a greeting
        run: echo "Hello from GitHub Actions!"

      - name: Show environment info
        run: |
          echo "Runner OS: ${{ runner.os }}"
          echo "Triggered by: ${{ github.actor }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
```

Push it:
```bash
git add .github/workflows/hello.yml
git commit -m "Add hello world workflow"
git push origin main
```

Go to your repo → **Actions** tab. You should see the workflow running. Click into it to see the logs.

You can also trigger it manually: **Actions → Hello World → Run workflow**.

**Checkpoint:** You can create a workflow and see it run in the Actions tab.

---

## Exercise 2: Add npm Scripts

Before writing a CI workflow, make sure your Next.js app has the scripts it needs.

In `app/package.json`, confirm these scripts exist (add them if they're missing):

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit"
  }
}
```

Test them locally:
```bash
cd app
npm run lint
npm run typecheck
npm run build
```

Fix any errors before continuing. A CI pipeline is only useful if it can actually pass.

**Checkpoint:** Your app's lint, typecheck, and build all pass locally.

---

## Exercise 3: CI Workflow

**`.github/workflows/ci.yml`**
```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  ci:
    name: Lint, typecheck, build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "npm"
          cache-dependency-path: app/package-lock.json

      - name: Install dependencies
        working-directory: app
        run: npm ci

      - name: Lint
        working-directory: app
        run: npm run lint

      - name: Type check
        working-directory: app
        run: npm run typecheck

      - name: Build
        working-directory: app
        run: npm run build
        env:
          DATABASE_URL: "postgresql://dummy:dummy@localhost:5432/dummy"
          MINIO_ENDPOINT: "localhost"
          MINIO_PORT: "9000"
          MINIO_ACCESS_KEY: "dummy"
          MINIO_SECRET_KEY: "dummy"
          MINIO_BUCKET: "dummy"
```

Push it on a feature branch and open a PR — you'll see the CI check appear on the PR. It must pass before you can merge (once you set up branch protection, which is the next exercise).

**Checkpoint:** CI runs on every PR and shows a pass/fail status.

---

## Exercise 4: Branch Protection

Configure GitHub to require CI to pass before merging:

1. Go to repo → **Settings → Branches**
2. Click **Add rule** (or **Add branch protection rule**)
3. Branch name pattern: `main`
4. Check: ✅ **Require status checks to pass before merging**
5. Search for and add: `Lint, typecheck, build` (the job name from your CI workflow)
6. Check: ✅ **Require branches to be up to date before merging**
7. Save

Now try merging a PR with a failing CI — GitHub will block it.

To test: introduce a TypeScript error on a branch, push it, open a PR, and watch the merge button grey out.

**Checkpoint:** You cannot merge to main without passing CI.

---

## Exercise 5: Workflow with Multiple Jobs

Refactor CI to have separate jobs that run in parallel:

**`.github/workflows/ci.yml`** (updated)
```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "npm"
          cache-dependency-path: app/package-lock.json
      - run: npm ci
        working-directory: app
      - run: npm run lint
        working-directory: app

  typecheck:
    name: Type check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "npm"
          cache-dependency-path: app/package-lock.json
      - run: npm ci
        working-directory: app
      - run: npm run typecheck
        working-directory: app

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, typecheck]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "npm"
          cache-dependency-path: app/package-lock.json
      - run: npm ci
        working-directory: app
      - run: npm run build
        working-directory: app
        env:
          DATABASE_URL: "postgresql://dummy:dummy@localhost:5432/dummy"
          MINIO_ENDPOINT: "localhost"
          MINIO_PORT: "9000"
          MINIO_ACCESS_KEY: "dummy"
          MINIO_SECRET_KEY: "dummy"
          MINIO_BUCKET: "dummy"
```

Notice: `lint` and `typecheck` run in parallel, `build` waits for both with `needs`.

Watch the workflow visualization in the Actions tab — you'll see the dependency graph.

**Checkpoint:** You can structure jobs to run in parallel and in sequence.

---

Ready? Head to [MINI_PROJECT.md](./MINI_PROJECT.md)!