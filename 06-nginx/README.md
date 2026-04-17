# 06 — Nginx

## What is Nginx and Why Do We Use It?

Nginx (pronounced "engine-x") is a web server that we use as a **reverse proxy**. In our stack, Nginx sits in front of everything — it's the single entry point that receives all incoming HTTP requests and forwards them to the right service.

```
Browser → Nginx (:80) → Next.js app (:3000)
                      → API service (:4000)  (if we had one)
```

Why not just expose the Next.js app directly on port 80? A few reasons:

- **Single entry point.** All traffic enters through Nginx. You can add SSL, rate limiting, and logging in one place.
- **Service routing.** Nginx can route `/api` to one service, `/` to another, `/docs` to another — without the services knowing about each other.
- **SSL termination.** HTTPS is handled at the Nginx layer. Services behind it communicate over plain HTTP.
- **Static file serving.** Nginx is extremely fast at serving static files (images, CSS, JS bundles) and can handle this without involving your app server.

---

## Core Concepts

### The nginx.conf Structure

Nginx config is organized into blocks. The main ones you'll work with:

```nginx
events {
  # Worker connection settings (usually leave as default)
  worker_connections 1024;
}

http {
  # HTTP-level settings, shared across all servers

  server {
    # A virtual server — handles a specific host/port combination
    listen 80;
    server_name localhost;

    location / {
      # Rules for requests matching this path
      proxy_pass http://app:3000;
    }
  }
}
```

### `location` Blocks

Location blocks match request paths and define what to do with them. Nginx tests them in order:

```nginx
server {
  listen 80;

  # Exact match (highest priority)
  location = /health {
    return 200 "ok";
  }

  # Prefix match — matches /api, /api/tasks, /api/anything
  location /api {
    proxy_pass http://api-service:4000;
  }

  # Default: everything else goes to the Next.js app
  location / {
    proxy_pass http://app:3000;
  }
}
```

### Reverse Proxy Headers

When Nginx proxies a request, the upstream service sees Nginx's IP, not the original client's. Add these headers so your app can read the real client IP and protocol:

```nginx
location / {
  proxy_pass http://app:3000;

  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
}
```

### WebSocket Support

Next.js uses WebSockets for hot-module reloading in development. Add these to proxy WebSocket connections:

```nginx
location / {
  proxy_pass http://app:3000;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

---

## A Complete Config for Our Stack

```nginx
events {
  worker_connections 1024;
}

http {
  upstream nextjs {
    server app:3000;
  }

  server {
    listen 80;
    server_name localhost;

    # Next.js app
    location / {
      proxy_pass http://nextjs;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }

    # MinIO API
    location /storage/ {
      proxy_pass http://minio:9000/;
      proxy_set_header Host $http_host;
    }
  }
}
```

---

## Nginx in Docker Compose

```yaml
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
```

The `:ro` suffix mounts the config file as read-only inside the container.

To reload the config without restarting the container:
```bash
docker exec task-tracker-nginx-1 nginx -s reload
```

To test the config for syntax errors:
```bash
docker exec task-tracker-nginx-1 nginx -t
```

## Helpful (optional, but encouraged) videos
[Fireship: Nginx in 100 Seconds](https://www.youtube.com/watch?v=JKxlsvZXG7c)

---

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) and this file will also have your Mini Project instructions

## Tasteful Memes
<img src="../memes/NginxMeme1.jpg" height="400" width="400">
<img src="../memes/NginxMeme2.jpeg" height="400" width="400">