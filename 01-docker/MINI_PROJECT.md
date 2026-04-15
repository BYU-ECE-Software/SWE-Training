# 01 — Docker: Mini Project

## Task Tracker Foundation

Set up the Docker Compose environment that the rest of the modules will build on. By the end of this project you'll have a `docker-compose.yml` that runs four services together.

---

## What You're Building

```
┌─────────────────────────────────────────┐
│           docker-compose.yml            │
│                                         │
│  ┌──────────┐      ┌──────────────────┐ │
│  │  nginx   │─────▶│   app (Next.js)  │ │
│  │ :80      │      │   :3000          │ │
│  └──────────┘      └──────────────────┘ │
│                                         │
│  ┌──────────┐      ┌──────────────────┐ │
│  │ postgres │      │     minio        │ │
│  │ :5432    │      │     :9000        │ │
│  └──────────┘      └──────────────────┘ │
└─────────────────────────────────────────┘
```

Don't worry that Next.js, Nginx, and MinIO modules haven't been covered yet — the goal here is just to get a skeleton compose file running. The services will be fleshed out in later modules.

---

## Steps

### 1. Create the project folder

```bash
mkdir task-tracker
cd task-tracker
```

### 2. Create a minimal Next.js app placeholder

```bash
# We'll scaffold the real Next.js app in module 02.
# For now, create a basic placeholder.
mkdir app-placeholder
```

**`app-placeholder/Dockerfile`**
```dockerfile
FROM node:18-alpine
WORKDIR /app
RUN echo '{"name":"placeholder","scripts":{"start":"node server.js"}}' > package.json
RUN echo 'const http=require("http");http.createServer((_,r)=>{r.writeHead(200);r.end("Task Tracker coming soon");}).listen(3000,()=>console.log("ready"))' > server.js
EXPOSE 3000
CMD ["node", "server.js"]
```

### 3. Create the docker-compose.yml

**`task-tracker/docker-compose.yml`**
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - app

  app:
    build: ./app-placeholder
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      MINIO_ENDPOINT: minio
      MINIO_PORT: 9000
    depends_on:
      - db
      - minio

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - minio_data:/data
    ports:
      - "9000:9000"
      - "9001:9001"

volumes:
  pgdata:
  minio_data:
```

### 4. Create your .env file

**`task-tracker/.env`**
```
POSTGRES_USER=dev
POSTGRES_PASSWORD=devpassword
POSTGRES_DB=tasktracker

MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin
```

### 5. Create a .gitignore

**`task-tracker/.gitignore`**
```
.env
node_modules/
.next/
```

### 6. Start it up

```bash
docker compose up --build
```

**Expected output:**
- `nginx` starts (you can visit http://localhost:80)
- `app` starts and logs "Task Tracker coming soon"
- `db` logs Postgres startup messages
- `minio` starts (you can visit the MinIO console at http://localhost:9001)

### 7. Verify each service

```bash
# Check all four are running
docker ps

# Verify postgres is reachable
docker exec -it task-tracker-db-1 psql -U dev -d tasktracker -c "SELECT version();"

# Tail app logs
docker compose logs -f app
```

---

## Stretch Goal

Add a `healthcheck` to the `db` service so that `app` only starts once Postgres is actually ready to accept connections (not just started):

```yaml
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    ...

  app:
    depends_on:
      db:
        condition: service_healthy
```

---

## Done!

You now have the skeleton of the Task Tracker running. Keep this folder — every subsequent module will add to it.

Move on to [02 — Next.js App Router](../02-nextjs/README.md).