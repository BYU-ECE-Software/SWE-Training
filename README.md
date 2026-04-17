# Dev Onboarding — Task Tracker

## Roman Vish (with some help from Claude)

Welcome! This repo is your hands-on guide to our tech stack. Instead of watching videos, you'll **build things** — specifically, a simple Task Tracker app that grows module by module until it's a fully running full-stack app. (And watching a couple videos hehe)

## The Project

By the end of these modules you'll have a **Task Tracker** with:
- A Next.js frontend with a data table of tasks
- A Postgres database managed through Prisma
- File attachments stored in MinIO
- Everything running locally in Docker Compose
- Proxied through Nginx
- CI/CD via GitHub Actions

## Modules

| # | Module | Concepts | Est. Time |
|---|--------|----------|-----------|
| 01 | [Docker](./01-docker/README.md) | Images, containers, Compose | ~2 hrs |
| 02 | [Next.js App Router](./02-nextjs/README.md) | Routing, layouts, server components | ~3 hrs |
| 03 | [Tailwind CSS](./03-tailwind/README.md) | Utility classes, responsive, dark mode | ~1.5 hrs |
| 04 | [Prisma](./04-prisma/README.md) | Schema, migrations, client queries | ~2 hrs |
| 05 | [PostgreSQL](./05-postgres/README.md) | Relational concepts, raw SQL | ~1.5 hrs |
| 06 | [Nginx](./06-nginx/README.md) | Reverse proxy, routing | ~1.5 hrs |
| 07 | [MinIO](./07-minio/README.md) | Object storage, S3 SDK | ~1 hr |
| 08 | [GitHub](./08-github/README.md) | Git workflow, branching, PRs, Projects | ~2 hrs |
| 09 | [GitHub Actions](./09-github-actions/README.md) | CI/CD pipelines | ~2 hrs |

## How to Use This Repo

Each module folder has:
- `README.md` — concepts guide with explanations and context
- `EXERCISES.md` — step-by-step hands-on tasks
- `MINI_PROJECT.md` — a small project that ties everything together
- A `starter/` folder where applicable with code to build on

**Do the modules in order** — later modules build on earlier ones. That said, if you're already comfortable with a topic, skim the README and jump straight to the exercises.

## Prerequisites

- Node.js 22+ installed
- Docker Desktop installed
- A GitHub account
- A code editor (VS Code recommended)

---

> Got stuck? Open an issue in this repo describing what you tried — that's also practice for module 08!