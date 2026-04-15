# 04 — Prisma

## What is Prisma?

Prisma is an ORM (Object-Relational Mapper) for TypeScript. It lets you interact with your database using TypeScript instead of writing raw SQL, and it gives you full type safety — your editor autocompletes query fields and flags type errors.

The three main pieces are:

1. **Prisma Schema** (`schema.prisma`) — defines your data models and maps them to database tables
2. **Prisma Migrate** — generates and runs SQL migrations from your schema changes
3. **Prisma Client** — an auto-generated, type-safe query library

---

## The Schema File

Everything starts in `prisma/schema.prisma`. Here's a minimal example:

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  tasks     Task[]
  createdAt DateTime @default(now())
}

model Task {
  id          Int       @id @default(autoincrement())
  title       String
  description String?
  status      Status    @default(TODO)
  assignee    User?     @relation(fields: [assigneeId], references: [id])
  assigneeId  Int?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}

enum Status {
  TODO
  IN_PROGRESS
  DONE
}
```

### Key Schema Concepts

**Field types:** `String`, `Int`, `Boolean`, `DateTime`, `Float`. Append `?` to make a field optional.

**`@id`** marks the primary key. `@default(autoincrement())` auto-increments it.

**`@unique`** adds a unique constraint.

**`@default(now())`** sets the field to the current timestamp on creation. `@updatedAt` automatically updates on every write.

**Relations** are defined as fields that reference another model. The `@relation` directive specifies which foreign key field connects them:
```prisma
assignee    User?     @relation(fields: [assigneeId], references: [id])
assigneeId  Int?
```
`assigneeId` is the actual foreign key column in the database. `assignee` is a virtual field that Prisma uses for joins — it doesn't exist as a column.

**Enums** define a fixed set of allowed values. Prisma maps these to database enums.

---

## Migrations

When you change the schema, you create a migration to apply those changes to the database:

```bash
# Create a new migration from your schema changes
npx prisma migrate dev --name add_task_table

# Apply pending migrations (used in production/CI)
npx prisma migrate deploy

# Reset the database and re-run all migrations (dev only — destructive!)
npx prisma migrate reset
```

`migrate dev` does three things: generates SQL, applies it to your database, and regenerates the Prisma Client.

Migrations live in `prisma/migrations/` and should be committed to version control. Never edit them by hand after they've been applied.

---

## Prisma Client

After running a migration, Prisma generates a typed client. You use it like this:

```ts
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

// Find all tasks
const tasks = await prisma.task.findMany();

// Find tasks with a specific status, include the assignee
const openTasks = await prisma.task.findMany({
  where: { status: "TODO" },
  include: { assignee: true },
  orderBy: { createdAt: "desc" },
});

// Find one by ID
const task = await prisma.task.findUnique({
  where: { id: 1 },
});

// Create
const newTask = await prisma.task.create({
  data: {
    title: "Write tests",
    status: "TODO",
    assigneeId: 2,
  },
});

// Update
const updated = await prisma.task.update({
  where: { id: 1 },
  data: { status: "DONE" },
});

// Delete
await prisma.task.delete({
  where: { id: 1 },
});
```

Everything is fully typed. If you type `prisma.task.findMany({ where: { stati...`, your editor will autocomplete `status` and only accept valid `Status` enum values.

### Singleton Pattern

In Next.js, you should not create a new `PrismaClient()` on every request — during development, hot reloading would create many connections and exhaust the database pool. The standard fix:

```ts
// lib/prisma.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ["query"], // logs queries in dev — remove in production
  });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

Import this throughout the app: `import { prisma } from "@/lib/prisma"`.

---

## Seeding

`prisma/seed.ts` is a convention for populating the database with initial data:

```ts
// prisma/seed.ts
import { prisma } from "../lib/prisma";

async function main() {
  await prisma.user.createMany({
    data: [
      { email: "alex@example.com", name: "Alex" },
      { email: "sam@example.com", name: "Sam" },
    ],
    skipDuplicates: true,
  });

  await prisma.task.createMany({
    data: [
      { title: "Set up Docker Compose", status: "DONE" },
      { title: "Scaffold Next.js", status: "IN_PROGRESS" },
      { title: "Write Prisma schema", status: "TODO" },
    ],
  });
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

Add to `package.json`:
```json
"prisma": {
  "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
}
```

Run with: `npx prisma db seed`

---

## Useful CLI Commands

```bash
# Open Prisma Studio — a visual DB browser
npx prisma studio

# Regenerate client after schema changes without running a migration
npx prisma generate

# Inspect the current database state
npx prisma db pull
```

---

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md), then [MINI_PROJECT.md](./MINI_PROJECT.md) to connect the Task Tracker to a real database.