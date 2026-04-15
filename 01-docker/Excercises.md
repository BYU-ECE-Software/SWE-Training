# 01 — Docker: Exercises

Work through these in order. Each exercise builds on the last.

---

## Exercise 1: Your First Container

No files to write yet — just get comfortable running containers.

```bash
# Pull and run an nginx web server
docker run -p 8080:80 nginx

# Visit http://localhost:8080 — you should see the nginx welcome page
# Press Ctrl+C to stop it
```

Now run it detached (in the background):

```bash
docker run -d -p 8080:80 --name my-nginx nginx

# Check it's running
docker ps

# Stop and remove it
docker stop my-nginx
docker rm my-nginx
```

**Checkpoint:** You understand the difference between `docker run`, `docker stop`, and `docker rm`.

---

## Exercise 2: Write a Dockerfile

Create a new folder called `hello-docker` and add these two files:

**`hello-docker/server.js`**
```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from inside a container!\n');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

**`hello-docker/package.json`**
```json
{
  "name": "hello-docker",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js"
  }
}
```

Now write the **`hello-docker/Dockerfile`** yourself. It should:
1. Start from `node:18-alpine`
2. Set the working directory to `/app`
3. Copy `package.json`
4. Run `npm install`
5. Copy the rest of the files
6. Expose port `3000`
7. Run `node server.js` as the start command

Build and run it:

```bash
cd hello-docker
docker build -t hello-docker .
docker run -p 3000:3000 hello-docker
# Visit http://localhost:3000
```

<details>
<summary>Stuck? Click to see the solution</summary>

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```
</details>

**Checkpoint:** You can write a Dockerfile from scratch and build an image from it.

---

## Exercise 3: Docker Compose with Two Services

Create a folder called `compose-demo`. Add the following files:

**`compose-demo/docker-compose.yml`**
```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: demo
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Copy your `server.js`, `package.json`, and `Dockerfile` from Exercise 2 into `compose-demo/`.

Then run it:

```bash
cd compose-demo
docker compose up --build
```

You should see both services start. Notice the `db` service logs show Postgres starting up.

Now try:
```bash
# Open a shell inside the running db container
docker exec -it compose-demo-db-1 psql -U dev -d demo

# Inside psql:
\dt        -- list tables (empty for now)
\q         -- quit
```

**Checkpoint:** You can define multiple services in a compose file and they can refer to each other by service name.

---

## Exercise 4: Environment Variables and .env

Hard-coding passwords in `docker-compose.yml` is fine locally but not ideal. Docker Compose automatically reads a `.env` file.

Create `compose-demo/.env`:
```
POSTGRES_USER=dev
POSTGRES_PASSWORD=devpassword
POSTGRES_DB=demo
```

Update `docker-compose.yml` to use them:
```yaml
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

Add `.env` to a `.gitignore` file:
```
.env
node_modules/
```

Restart and confirm it still works:
```bash
docker compose down
docker compose up --build
```

**Checkpoint:** You understand how to pass secrets via environment variables and keep them out of version control.

---

## Reflection Questions

Before moving on, make sure you can answer these:

1. What's the difference between an image and a container?
2. Why do we copy `package.json` and run `npm install` before copying the rest of the source code?
3. What does `ports: "3000:3000"` mean — which side is the host and which is the container?
4. What happens to your database data if you run `docker compose down -v`?
5. Why do services in a compose file refer to each other by service name (like `db`) instead of `localhost`?

---

Ready? Head to [MINI_PROJECT.md](./MINI_PROJECT.md).