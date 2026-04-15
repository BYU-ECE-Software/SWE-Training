# 02 — Next.js App Router: Exercises

## Setup

Scaffold a new Next.js app to work in for these exercises:

```bash
npx create-next-app@latest nextjs-exercises --typescript --tailwind --eslint --app --no-src-dir
cd nextjs-exercises
npm run dev
```

Visit http://localhost:3000 to confirm it's running.

---

## Exercise 1: Pages and Routing

Create two new pages by adding folders and `page.tsx` files.

**Create `app/about/page.tsx`:**
```tsx
export default function AboutPage() {
  return <h1>About</h1>;
}
```

Visit http://localhost:3000/about — it works automatically.

Now create a dynamic route for tasks:

**Create `app/tasks/page.tsx`:**
```tsx
export default function TasksPage() {
  return <h1>All Tasks</h1>;
}
```

**Create `app/tasks/[id]/page.tsx`:**
```tsx
export default function TaskDetailPage({
  params,
}: {
  params: { id: string };
}) {
  return <h1>Task #{params.id}</h1>;
}
```

Visit http://localhost:3000/tasks/42 — you should see "Task #42". Change the URL to any number.

**Checkpoint:** You understand how folder names become URL segments and how `[id]` creates a dynamic segment.

---

## Exercise 2: Layouts and Navigation

**Update `app/layout.tsx`** to add a navigation bar that appears on every page:

```tsx
import Link from "next/link";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <nav style={{ padding: "1rem", borderBottom: "1px solid #eee" }}>
          <Link href="/">Home</Link>
          {" | "}
          <Link href="/tasks">Tasks</Link>
          {" | "}
          <Link href="/about">About</Link>
        </nav>
        <main style={{ padding: "1rem" }}>
          {children}
        </main>
      </body>
    </html>
  );
}
```

Click through the nav links and notice the page content changes but the nav stays — that's the layout at work.

Now add a **nested layout** that only wraps task pages:

**Create `app/tasks/layout.tsx`:**
```tsx
export default function TasksLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <h2 style={{ color: "gray" }}>Tasks Section</h2>
      {children}
    </div>
  );
}
```

Visit `/tasks` and `/tasks/42` — both show the "Tasks Section" header. Visit `/about` — it doesn't.

**Checkpoint:** You understand how layouts compose and nest.

---

## Exercise 3: Server Components vs Client Components

This is the big one. You'll build the same feature two ways.

### Part A — Server Component (data fetching)

**Update `app/tasks/page.tsx`** to fetch data on the server:

```tsx
// No "use client" — this is a Server Component

type Post = {
  id: number;
  title: string;
  completed: boolean;
};

async function getTasks(): Promise<Post[]> {
  // fetch() works directly in server components
  const res = await fetch("https://jsonplaceholder.typicode.com/todos?_limit=10");
  return res.json();
}

export default async function TasksPage() {
  const tasks = await getTasks(); // runs on the server!

  return (
    <div>
      <h1>Tasks</h1>
      <ul>
        {tasks.map((task) => (
          <li key={task.id} style={{ textDecoration: task.completed ? "line-through" : "none" }}>
            {task.title}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Open DevTools → Network tab. Notice there's no fetch request to `jsonplaceholder` from the browser — it happened on the server.

### Part B — Client Component (interactivity)

Create a separate component that needs browser interactivity:

**Create `app/tasks/TaskFilter.tsx`:**
```tsx
"use client";

import { useState } from "react";

type Props = {
  onFilterChange: (showCompleted: boolean) => void;
};

export default function TaskFilter({ onFilterChange }: Props) {
  const [showCompleted, setShowCompleted] = useState(true);

  const toggle = () => {
    const next = !showCompleted;
    setShowCompleted(next);
    onFilterChange(next);
  };

  return (
    <button onClick={toggle}>
      {showCompleted ? "Hide completed" : "Show completed"}
    </button>
  );
}
```

Now update the page — notice we need to make the page itself a client component to use state for filtering. This is the trade-off: once you need `useState`, the component (and its data fetching) moves to the client.

```tsx
// app/tasks/page.tsx — now a client component to support filtering
"use client";

import { useEffect, useState } from "react";
import TaskFilter from "./TaskFilter";

type Task = { id: number; title: string; completed: boolean };

export default function TasksPage() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [showCompleted, setShowCompleted] = useState(true);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/todos?_limit=10")
      .then((r) => r.json())
      .then(setTasks);
  }, []);

  const filtered = showCompleted ? tasks : tasks.filter((t) => !t.completed);

  return (
    <div>
      <h1>Tasks</h1>
      <TaskFilter onFilterChange={setShowCompleted} />
      <ul>
        {filtered.map((task) => (
          <li key={task.id}>{task.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

Now check DevTools — the fetch IS visible in the network tab because it's running in the browser.

**Checkpoint:** You understand the trade-off between server and client components, and when you need to use `"use client"`.

---

## Exercise 4: API Routes

Create a simple API endpoint.

**Create `app/api/tasks/route.ts`:**
```ts
const MOCK_TASKS = [
  { id: 1, title: "Set up Docker", status: "done" },
  { id: 2, title: "Learn Next.js", status: "in_progress" },
  { id: 3, title: "Write tests", status: "todo" },
];

export async function GET() {
  return Response.json(MOCK_TASKS);
}

export async function POST(request: Request) {
  const body = await request.json();

  if (!body.title) {
    return Response.json({ error: "title is required" }, { status: 400 });
  }

  const newTask = {
    id: MOCK_TASKS.length + 1,
    title: body.title,
    status: "todo",
  };

  // In a real app we'd save to the DB here
  MOCK_TASKS.push(newTask);

  return Response.json(newTask, { status: 201 });
}
```

Test it:
```bash
# GET
curl http://localhost:3000/api/tasks

# POST
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "My new task"}'
```

**Checkpoint:** You can create API endpoints and handle different HTTP methods.

---

## Reflection Questions

1. What's the difference between a Server Component and a Client Component? When would you choose each?
2. If you have a Server Component that fetches data and you need to add a `useState` hook to it, what do you have to change?
3. What's the difference between `layout.tsx` and `page.tsx`?
4. How does Next.js know that `app/tasks/[id]/page.tsx` maps to `/tasks/42`?
5. When you use `Link` from next/link instead of a regular `<a>` tag, what does Next.js do differently?

---

Ready? Head to [MINI_PROJECT.md](./MINI_PROJECT.md).