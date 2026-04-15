# 01 — Docker

## What is Docker and why do we use it?

Imagine you build an app on your MacBook. It works perfectly. You hand it to a colleague and it breaks immediately — wrong Node version, a missing environment variable, a different OS. Docker solves this by packaging your app **together with everything it needs to run** into a single portable unit called a **container**.

Think of it like this: a container is a shipping container. The contents (your app) are sealed inside with everything they need. It doesn't matter whether the ship is sailing across the Pacific or sitting in a warehouse — the container behaves the same.

We use Docker on every project so that:
- Local dev matches production
- New team members can run the full stack with one command
- We don't get "works on my machine" bugs

---

## Core Concepts

### Image vs Container

These two words come up constantly and people mix them up.

- An **image** is a blueprint. It's a read-only snapshot of an app and its environment. Think of it like a class definition in code.
- A **container** is a running instance of an image. Like an object instantiated from that class.

You can run many containers from the same image simultaneously.

```
Image (blueprint)  →  Container (running instance)
      ↓                       ↓
  node:18             your app, running, with its own filesystem
```

### Dockerfile

A `Dockerfile` is a recipe for building an image. Each line is an instruction that adds a layer to the image.

```dockerfile
# Start from an existing base image (Node 18 on Alpine Linux)
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package files first (why? see the "layer caching" note below)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the source code
COPY . .

# Build the Next.js app
RUN npm run build

# Tell Docker which port this app listens on
EXPOSE 3000

# The command to run when the container starts
CMD ["npm", "start"]
```

> **Layer caching:** Docker caches each layer. If `package.json` hasn't changed, `npm install` won't re-run on the next build — it uses the cache. This is why we copy `package*.json` and install *before* copying source code. Source code changes often; dependencies don't.

### docker-compose

Running a single container is fine. But a real app has multiple services — a Next.js server, a Postgres database, maybe a MinIO instance. `docker-compose` lets you define and run all of them together with a single `docker-compose.yml` file.

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: tasktracker
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Key things to notice:
- `ports: "3000:3000"` means **host port : container port**. Traffic hitting your laptop on 3000 is forwarded to port 3000 inside the container.
- `depends_on` ensures `db` starts before `app`.
- `volumes` gives Postgres a persistent place to store data. Without this, your database would be wiped every time you restart the container.

### Useful Commands

```bash
# Build and start all services defined in docker-compose.yml
docker compose up --build

# Start in the background (detached mode)
docker compose up -d

# Stop everything
docker compose down

# Stop and delete volumes (wipes your DB — use with care)
docker compose down -v

# See running containers
docker ps

# Tail logs from a specific service
docker compose logs -f app

# Open a shell inside a running container
docker exec -it <container_name> sh
```

---

## How Docker Fits Our Stack

In our projects, Docker Compose typically runs:

| Service | Image | What it does |
|---------|-------|-------------|
| `app` | Built from our Dockerfile | The Next.js app |
| `db` | `postgres:16` | The database |
| `minio` | `minio/minio` | Local file storage |
| `nginx` | `nginx:alpine` | Reverse proxy / routing |

When you clone a project and run `docker compose up`, all four start together. No manual database installs, no port conflicts with other projects.

---

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) to build and run your first containers, then [MINI_PROJECT.md](./MINI_PROJECT.md) to set up the Task Tracker's Docker foundation.