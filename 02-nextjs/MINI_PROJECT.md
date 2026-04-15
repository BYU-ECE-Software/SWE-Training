# 02 — Next.js: Mini Project

## Task Tracker — App Shell

Replace the Docker placeholder from module 01 with a real Next.js app. By the end you'll have the Task Tracker's routing structure and a static (no real data yet) task table.

---

## Steps

### 1. Scaffold the Next.js app

Inside your `task-tracker/` folder from module 01:

```bash
npx create-next-app@latest app --typescript --tailwind --eslint --app --no-src-dir
```

Update `docker-compose.yml` to point to the new `app/` folder instead of `app-placeholder/`:

```yaml
  app:
    build: ./app
    ...
```

### 2. Update the Dockerfile

Replace `app/Dockerfile` with a proper multi-stage build:

```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm install

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json
EXPOSE 3000
CMD ["npm", "start"]
```

### 3. Create the app structure

Build out the following folder structure under `app/app/`:

```
app/app/
├── layout.tsx
├── page.tsx
├── tasks/
│   ├── page.tsx
│   └── [id]/
│       └── page.tsx
└── api/
    └── tasks/
        └── route.ts
```

### 4. Root layout with navigation

**`app/app/layout.tsx`**
```tsx
import type { Metadata } from "next";
import Link from "next/link";
import "./globals.css";

export const metadata: Metadata = {
  title: "Task Tracker",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <header className="border-b px-6 py-4 flex items-center gap-6">
          <span className="font-semibold text-lg">Task Tracker</span>
          <nav className="flex gap-4 text-sm">
            <Link href="/" className="hover:underline">Home</Link>
            <Link href="/tasks" className="hover:underline">Tasks</Link>
          </nav>
        </header>
        <main className="p-6">{children}</main>
      </body>
    </html>
  );
}
```

### 5. Home page

**`app/app/page.tsx`**
```tsx
import Link from "next/link";

export default function HomePage() {
  return (
    <div className="max-w-lg">
      <h1 className="text-2xl font-semibold mb-2">Welcome to Task Tracker</h1>
      <p className="text-gray-600 mb-4">
        A simple app to manage tasks. Built module by module during onboarding.
      </p>
      <Link
        href="/tasks"
        className="inline-block bg-black text-white px-4 py-2 rounded hover:bg-gray-800"
      >
        View Tasks →
      </Link>
    </div>
  );
}
```

### 6. Tasks page with a static table

**`app/app/tasks/page.tsx`**
```tsx
// Static data for now — module 04 (Prisma) will replace this with real DB data

const MOCK_TASKS = [
  { id: 1, title: "Set up Docker Compose", status: "done", assignee: "Alex" },
  { id: 2, title: "Scaffold Next.js app", status: "in_progress", assignee: "Sam" },
  { id: 3, title: "Design database schema", status: "todo", assignee: "Jordan" },
  { id: 4, title: "Write Prisma migrations", status: "todo", assignee: "Alex" },
];

const STATUS_STYLES: Record<string, string> = {
  done: "bg-green-100 text-green-800",
  in_progress: "bg-yellow-100 text-yellow-800",
  todo: "bg-gray-100 text-gray-700",
};

export default function TasksPage() {
  return (
    <div>
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-xl font-semibold">Tasks</h1>
        <button className="bg-black text-white px-3 py-1.5 rounded text-sm hover:bg-gray-800">
          + New Task
        </button>
      </div>

      <table className="w-full text-sm border-collapse">
        <thead>
          <tr className="border-b text-left text-gray-500">
            <th className="pb-2 pr-4">Title</th>
            <th className="pb-2 pr-4">Status</th>
            <th className="pb-2">Assignee</th>
          </tr>
        </thead>
        <tbody>
          {MOCK_TASKS.map((task) => (
            <tr key={task.id} className="border-b hover:bg-gray-50">
              <td className="py-3 pr-4 font-medium">{task.title}</td>
              <td className="py-3 pr-4">
                <span className={`px-2 py-0.5 rounded-full text-xs font-medium ${STATUS_STYLES[task.status]}`}>
                  {task.status.replace("_", " ")}
                </span>
              </td>
              <td className="py-3 text-gray-600">{task.assignee}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### 7. API route (stub)

**`app/app/api/tasks/route.ts`**
```ts
// Stub — will be connected to Prisma in module 04

const MOCK_TASKS = [
  { id: 1, title: "Set up Docker Compose", status: "done" },
  { id: 2, title: "Scaffold Next.js app", status: "in_progress" },
];

export async function GET() {
  return Response.json(MOCK_TASKS);
}

export async function POST(request: Request) {
  const body = await request.json();
  if (!body.title) {
    return Response.json({ error: "title is required" }, { status: 400 });
  }
  return Response.json({ id: Date.now(), ...body, status: "todo" }, { status: 201 });
}
```

### 8. Rebuild and run

```bash
# From task-tracker/
docker compose up --build
```

Visit http://localhost (port 80, routed through nginx from module 01) or http://localhost:3000 directly.

---

## What You Now Have

- A real Next.js app with a nav, home page, and task table
- A routing structure that's ready for real data
- An API stub at `/api/tasks`
- The Docker setup from module 01 still intact

The table shows static data for now. In module 04 (Prisma), you'll replace `MOCK_TASKS` with live queries to a real Postgres database.

Move on to [03 — Tailwind CSS](../03-tailwind/README.md).