# 03 — Tailwind CSS: Mini Project

## Task Tracker — Polished UI

Take the plain task table from module 02 and give it a real, polished look using Tailwind. You'll build a `StatusBadge` component, a proper table layout, and a modal skeleton for creating tasks.

---

## Steps

### 1. Create shared utilities

**`app/lib/utils.ts`**
```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```bash
npm install clsx tailwind-merge
```

### 2. Create a StatusBadge component

**`app/components/StatusBadge.tsx`**
```tsx
import { cn } from "@/lib/utils";

export type TaskStatus = "todo" | "in_progress" | "done";

const LABELS: Record<TaskStatus, string> = {
  todo: "To Do",
  in_progress: "In Progress",
  done: "Done",
};

export default function StatusBadge({ status }: { status: TaskStatus }) {
  return (
    <span
      className={cn(
        "inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium",
        status === "todo" && "bg-gray-100 text-gray-600",
        status === "in_progress" && "bg-yellow-100 text-yellow-800",
        status === "done" && "bg-green-100 text-green-800"
      )}
    >
      {LABELS[status]}
    </span>
  );
}
```

### 3. Update the tasks page

Replace the plain table in `app/app/tasks/page.tsx` with a polished version:

```tsx
import StatusBadge, { type TaskStatus } from "@/components/StatusBadge";

const MOCK_TASKS = [
  { id: 1, title: "Set up Docker Compose", status: "done" as TaskStatus, assignee: "Alex", createdAt: "2024-01-10" },
  { id: 2, title: "Scaffold Next.js app", status: "in_progress" as TaskStatus, assignee: "Sam", createdAt: "2024-01-11" },
  { id: 3, title: "Design database schema", status: "todo" as TaskStatus, assignee: "Jordan", createdAt: "2024-01-12" },
  { id: 4, title: "Write Prisma migrations", status: "todo" as TaskStatus, assignee: "Alex", createdAt: "2024-01-12" },
];

export default function TasksPage() {
  return (
    <div className="max-w-4xl">
      {/* Header */}
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-xl font-semibold text-gray-900">Tasks</h1>
          <p className="text-sm text-gray-500 mt-0.5">{MOCK_TASKS.length} tasks total</p>
        </div>
        <button className="inline-flex items-center gap-1.5 bg-black text-white text-sm font-medium px-3.5 py-2 rounded-md hover:bg-gray-800 transition-colors">
          <span>+</span> New Task
        </button>
      </div>

      {/* Filter bar */}
      <div className="flex gap-2 mb-4">
        {["All", "To Do", "In Progress", "Done"].map((filter) => (
          <button
            key={filter}
            className={cn(
              "px-3 py-1.5 rounded-md text-sm font-medium transition-colors",
              filter === "All"
                ? "bg-gray-900 text-white"
                : "bg-white text-gray-600 border border-gray-200 hover:bg-gray-50"
            )}
          >
            {filter}
          </button>
        ))}
      </div>

      {/* Table */}
      <div className="bg-white border border-gray-200 rounded-lg overflow-hidden">
        <table className="w-full text-sm">
          <thead className="bg-gray-50 border-b border-gray-200">
            <tr>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">
                Title
              </th>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">
                Status
              </th>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">
                Assignee
              </th>
              <th className="text-left text-xs font-medium text-gray-500 uppercase tracking-wide px-4 py-3">
                Created
              </th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-100">
            {MOCK_TASKS.map((task) => (
              <tr key={task.id} className="hover:bg-gray-50 transition-colors">
                <td className="px-4 py-3 font-medium text-gray-900">{task.title}</td>
                <td className="px-4 py-3">
                  <StatusBadge status={task.status} />
                </td>
                <td className="px-4 py-3 text-gray-600">{task.assignee}</td>
                <td className="px-4 py-3 text-gray-500">{task.createdAt}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

> Note: The `cn` import at the top of the file needs `import { cn } from "@/lib/utils";`

### 4. Add an empty state

The table will eventually support filtering. Add an empty state for when there are no results:

```tsx
{MOCK_TASKS.length === 0 && (
  <tr>
    <td colSpan={4} className="px-4 py-12 text-center text-gray-400">
      No tasks found.
    </td>
  </tr>
)}
```

---

## What You Now Have

A task table that looks like a real product — with a badge component you can reuse, a filter bar skeleton, hover states, and proper typography. The data is still static; that changes in module 04.

Move on to [04 — Prisma](../04-prisma/README.md).