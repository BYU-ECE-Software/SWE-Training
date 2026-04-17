# 06 — Nginx: Exercises & Mini Project

---

## Exercise 1: Your First nginx.conf

Create a new folder `nginx-demo/` with the following files:

**`nginx-demo/nginx.conf`**
```nginx
events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location / {
      return 200 "Hello from Nginx!\n";
      add_header Content-Type text/plain;
    }
  }
}
```

**`nginx-demo/docker-compose.yml`**
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
```

Run it:
```bash
cd nginx-demo
docker compose up
curl http://localhost:8080
# → Hello from Nginx!
```

**Checkpoint:** You can run Nginx with a custom config and serve a basic response.

---

## Exercise 2: Proxy to a Backend

Add a simple backend service to `nginx-demo/`:

**`nginx-demo/backend/server.js`**
```js
const http = require("http");
http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ message: "From backend", path: req.url }));
}).listen(3000, () => console.log("Backend on 3000"));
```

**`nginx-demo/backend/Dockerfile`**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY server.js .
CMD ["node", "server.js"]
```

Update `docker-compose.yml`:
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend

  backend:
    build: ./backend
```

Update `nginx.conf` to proxy to the backend:
```nginx
events { worker_connections 1024; }

http {
  server {
    listen 80;

    location / {
      proxy_pass http://backend:3000;
      proxy_set_header Host $host;
    }
  }
}
```

```bash
docker compose up --build
curl http://localhost:8080/hello
# → {"message":"From backend","path":"/hello"}
```

**Checkpoint:** Nginx is proxying requests to a backend service.

---

## Exercise 3: Multiple Routes

Update `nginx.conf` to route different paths to different responses:

```nginx
events { worker_connections 1024; }

http {
  server {
    listen 80;

    location = /health {
      return 200 "healthy\n";
      add_header Content-Type text/plain;
    }

    location /api/ {
      proxy_pass http://backend:3000/;
      proxy_set_header Host $host;
    }

    location / {
      return 200 "Welcome to the app\n";
      add_header Content-Type text/plain;
    }
  }
}
```

Test each route:
```bash
curl http://localhost:8080/health
curl http://localhost:8080/api/tasks
curl http://localhost:8080/
```

**Checkpoint:** You can route different paths to different upstreams or responses.

---

## Exercise 4: Test and Reload

Make a deliberate syntax error in `nginx.conf` (e.g., remove a semicolon), then:

```bash
# Test config without restarting — should report an error
docker exec nginx-demo-nginx-1 nginx -t
```

Fix it, then reload without downtime:
```bash
docker exec nginx-demo-nginx-1 nginx -s reload
```

**Checkpoint:** You can validate and reload Nginx config without restarting the container.

---

## Mini Project: Wire Up the Task Tracker

Update the Task Tracker's Nginx config to properly route all traffic.

### 1. Create the nginx config file

**`task-tracker/nginx/nginx.conf`**
```nginx
events {
  worker_connections 1024;
}

http {
  upstream nextjs_app {
    server app:3000;
  }

  server {
    listen 80;
    server_name localhost;

    # Health check endpoint
    location = /health {
      return 200 "healthy\n";
      add_header Content-Type text/plain;
    }

    # Everything else → Next.js
    location / {
      proxy_pass http://nextjs_app;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      # Increase timeout for slow server starts in dev
      proxy_read_timeout 120s;
    }
  }
}
```

### 2. Update docker-compose.yml

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

### 3. Test it

```bash
docker compose up --build

# Health check
curl http://localhost/health

# App (should show the Task Tracker)
curl -I http://localhost
```

---

## Reflection Questions

1. What's the difference between a web server and a reverse proxy?
2. Why do we set `proxy_set_header X-Real-IP $remote_addr`?
3. What happens if you proxy to `http://app:3000` but the `app` service isn't running yet?
4. What does `nginx -s reload` do differently from restarting the container?

---

Move on to [07 — MinIO](../07-minio/README.md)!