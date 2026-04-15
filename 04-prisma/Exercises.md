# 04 — Prisma: Exercises

Make sure Postgres is running: `docker compose up -d db` from your task-tracker folder.

---

## Exercise 1: Initialize Prisma

```bash
npm install prisma @prisma/client
npx prisma init
```

This creates `prisma/schema.prisma` and adds `DATABASE_URL` to `.env`.

Update `.env` with your Postgres connection string:
```
DATABASE_URL="postgresql://dev:devpassword@localhost:5432/tasktracker"
```

---

## Exercise 2: Define a Schema

Replace the contents of `prisma/schema.prisma` with a simple blog schema (we'll switch to the task tracker schema in the mini project):

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Author {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
  posts Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  published Boolean  @default(false)
  author    Author   @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
}
```

Run the migration:

```bash
npx prisma migrate dev --name init
```

Open Prisma Studio to see the empty tables:

```bash
npx prisma studio
```

**Checkpoint:** You can define a schema and run a migration.

---

## Exercise 3: Write a Seed Script

Create `prisma/seed.ts`:

```ts
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

async function main() {
  const alice = await prisma.author.upsert({
    where: { email: "alice@example.com" },
    update: {},
    create: { name: "Alice", email: "alice@example.com" },
  });

  await prisma.post.createMany({
    data: [
      { title: "Hello World", body: "My first post", authorId: alice.id, published: true },
      { title: "Draft Post", body: "Work in progress", authorId: alice.id, published: false },
    ],
  });

  console.log("Seeded!");
}

main().catch(console.error).finally(() => prisma.$disconnect());
```

Install ts-node:
```bash
npm install -D ts-node
```

Add to `package.json`:
```json
"prisma": {
  "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
}
```

Run it:
```bash
npx prisma db seed
```

Check Prisma Studio — you should see the data.

**Checkpoint:** You can seed a database with initial data.

---

## Exercise 4: Query with the Client

Create `prisma/query.ts` and run each query, checking the output:

```ts
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

async function main() {
  // 1. Find all published posts with their authors
  const published = await prisma.post.findMany({
    where: { published: true },
    include: { author: true },
  });
  console.log("Published posts:", JSON.stringify(published, null, 2));

  // 2. Find a single author by email
  const author = await prisma.author.findUnique({
    where: { email: "alice@example.com" },
    include: { posts: true },
  });
  console.log("Alice's posts:", author?.posts.length);

  // 3. Update a post
  const updated = await prisma.post.update({
    where: { id: 2 },
    data: { published: true },
  });
  console.log("Updated post:", updated.title, "published:", updated.published);

  // 4. Count posts per author
  const counts = await prisma.post.groupBy({
    by: ["authorId"],
    _count: { id: true },
  });
  console.log("Post counts:", counts);
}

main().catch(console.error).finally(() => prisma.$disconnect());
```

Run with: `npx ts-node --compiler-options '{"module":"CommonJS"}' prisma/query.ts`

**Checkpoint:** You can query, filter, include relations, and update records.

---

## Reflection Questions

1. What's the difference between `findMany`, `findUnique`, and `findFirst`?
2. What does `include: { author: true }` do? What SQL does it roughly translate to?
3. What happens if you change `email String @unique` to `email String` in the schema and run a migration on a table that already has duplicate emails?
4. Why do we use a singleton pattern for `PrismaClient` in Next.js?
5. What's the difference between `prisma migrate dev` and `prisma migrate deploy`?

---

Ready? Head to [MINI_PROJECT.md](./MINI_PROJECT.md).