# 04 — Prisma: Mini Project

## Task Tracker — Real Database

Replace the `MOCK_TASKS` array with a live Postgres database. By the end, creating a task via the API will persist it, and the tasks page will show real data.

---

## Steps

### 1. Install Prisma in the app

```bash
cd task-tracker/app
npm install prisma @prisma/client
npm install -D ts-node
npx prisma init
```

Update `app/.env` (make sure this is gitignored):
```
DATABASE_URL="postgresql://dev:devpassword@localhost:5432/tasktracker"
```

When running in Docker, the host is `db` not `localhost`:
```
DATABASE_URL="postgresql://dev:devpassword@db:5432/tasktracker"
```

Use the `db` hostname inside Docker, `localhost` when running Next.js locally outside containers.

### 2. Define the Task Tracker schema

**`app/prisma/schema.prisma`**
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
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

### 3. Run the migration

```bash
npx prisma migrate dev --name init
```

### 4. Create the Prisma client singleton

**`app/lib/prisma.ts`**
```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ?? new PrismaClient({ log: ["query"] });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

### 5. Seed the database

**`app/prisma/seed.ts`**
```ts
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

async function main() {
  const alex = await prisma.user.upsert({
    where: { email: "alex@example.com" },
    update: {},
    create: { name: "Alex", email: "alex@example.com" },
  });
  const sam = await prisma.user.upsert({
    where: { email: "sam@example.com" },
    update: {},
    create: { name: "Sam", email: "sam@example.com" },
  });

  await prisma.task.createMany({
    data: [
      { title: "Set up Docker Compose", status: "DONE", assigneeId: alex.id },
      { title: "Scaffold Next.js app", status: "IN_PROGRESS", assigneeId: sam.id },
      { title: "Design database schema", status: "TODO", assigneeId: alex.id },
      { title: "Write Prisma migrations", status: "TODO" },
    ],
  });

  console.log("Seeded ✓");
}

main().catch(console.error).finally(() => prisma.$disconnect());
```

Add to `package.json`:
```json
"prisma": {
  "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
}
```

Run: `npx prisma db seed`

### 6. Update the API routes

**`app/app/api/tasks/route.ts`**
```ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";

export async function GET() {
  const tasks = await prisma.task.findMany({
    include: { assignee: true },
    orderBy: { createdAt: "desc" },
  });
  return NextResponse.json(tasks);
}

export async function POST(request: Request) {
  const body = await request.json();

  if (!body.title) {
    return NextResponse.json({ error: "title is required" }, { status: 400 });
  }

  const task = await prisma.task.create({
    data: {
      title: body.title,
      description: body.description,
      status: "TODO",
      assigneeId: body.assigneeId ?? null,
    },
    include: { assignee: true },
  });

  return NextResponse.json(task, { status: 201 });
}
```

Add a PATCH route for updating status:

**`app/app/api/tasks/[id]/route.ts`**
```ts
import { prisma } from "@/lib/prisma";
import { NextResponse } from "next/server";

export async function PATCH(
  request: Request,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const task = await prisma.task.update({
    where: { id: Number(params.id) },
    data: body,
    include: { assignee: true },
  });
  return NextResponse.json(task);
}

export async function DELETE(
  _request: Request,
  { params }: { params: { id: string } }
) {
  await prisma.task.delete({ where: { id: Number(params.id) } });
  return new Response(null, { status: 204 });
}
```

### 7. Update the tasks page to use real data

**`app/app/tasks/page.tsx`** (Server Component — fetches directly from DB):
```tsx
import { prisma } from "@/lib/prisma";
import StatusBadge from "@/components/StatusBadge";
import type { TaskStatus } from "@/components/StatusBadge";

export default async function TasksPage() {
  const tasks = await prisma.task.findMany({
    include: { assignee: true },
    orderBy: { createdAt: "desc" },
  });

  return (
    <div className="max-w-4xl">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-xl font-semibold text-gray-900">Tasks</h1>
          <p className="text-sm text-gray-500 mt-0.5">{tasks.length} tasks</p>
        </div>
      </div>

      <div className="bg-white border border-gray-200 rounded-lg overflow-hidden">
        <table className="w-full text-sm">
          <thead className="bg-gray-50 border-b border-gray-200">
            <tr>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">Title</th>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">Status</th>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">Assignee</th>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">Created</th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-100">
            {tasks.map((task) => (
              <tr key={task.id} className="hover:bg-gray-50">
                <td className="px-4 py-3 font-medium">{task.title}</td>
                <td className="px-4 py-3">
                  <StatusBadge status={task.status.toLowerCase() as TaskStatus} />
                </td>
                <td className="px-4 py-3 text-gray-600">{task.assignee?.name ?? "—"}</td>
                <td className="px-4 py-3 text-gray-500">
                  {task.createdAt.toLocaleDateString()}
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

### 8. Verify

```bash
# From task-tracker/
docker compose up --build

# Seed the DB
docker exec -it task-tracker-app-1 npx prisma db seed

# Visit http://localhost/tasks
```

You should see real tasks from the database.

---

## What You Now Have

- A real Postgres schema with `User` and `Task` models
- Full CRUD API routes backed by Prisma
- The tasks page fetching live data as a Server Component
- A seed script you can re-run any time

Move on to [05 — PostgreSQL](../05-postgres/README.md).