# 03 — Tailwind CSS: Exercises

Use the Next.js app from module 02, or run `npx create-next-app@latest tailwind-exercises --typescript --tailwind --app` for a fresh sandbox.

---

## Exercise 1: Build a Card Component

Without writing any CSS, build this card using only Tailwind utilities:

- White background, light gray border, rounded corners, padding
- A bold title
- A muted description text
- A colored badge (pick any color)

**Try it yourself first, then compare:**

```tsx
// components/TaskCard.tsx
export default function TaskCard() {
  return (
    <div className="bg-white border border-gray-200 rounded-lg p-4 max-w-sm">
      <div className="flex items-start justify-between mb-2">
        <h3 className="font-semibold text-gray-900">Set up Docker Compose</h3>
        <span className="bg-green-100 text-green-800 text-xs font-medium px-2 py-0.5 rounded-full">
          Done
        </span>
      </div>
      <p className="text-sm text-gray-500">
        Configure all four services to run together locally.
      </p>
      <div className="mt-3 text-xs text-gray-400">Assigned to: Alex</div>
    </div>
  );
}
```

**Checkpoint:** You can build a styled component without opening a `.css` file.

---

## Exercise 2: Responsive Grid

Build a grid of cards that changes from 1 column on mobile to 3 columns on desktop:

```tsx
export default function TaskGrid() {
  const tasks = [
    { id: 1, title: "Docker setup", status: "done" },
    { id: 2, title: "Next.js scaffold", status: "in_progress" },
    { id: 3, title: "Tailwind styling", status: "in_progress" },
    { id: 4, title: "Prisma schema", status: "todo" },
    { id: 5, title: "API routes", status: "todo" },
    { id: 6, title: "Write tests", status: "todo" },
  ];

  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 p-6">
      {tasks.map((task) => (
        <div key={task.id} className="bg-white border border-gray-200 rounded-lg p-4">
          <p className="font-medium">{task.title}</p>
          <p className="text-sm text-gray-500 mt-1">{task.status}</p>
        </div>
      ))}
    </div>
  );
}
```

Resize your browser window and watch the grid change.

**Checkpoint:** You can use responsive prefixes to adapt layout to screen size.

---

## Exercise 3: Conditional Classes with cn()

Install the utilities:

```bash
npm install clsx tailwind-merge
```

Create `lib/utils.ts`:
```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Now build a `StatusBadge` component that changes color based on status:

```tsx
import { cn } from "@/lib/utils";

type Status = "todo" | "in_progress" | "done";

export default function StatusBadge({ status }: { status: Status }) {
  return (
    <span
      className={cn(
        "inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium",
        status === "todo" && "bg-gray-100 text-gray-700",
        status === "in_progress" && "bg-yellow-100 text-yellow-800",
        status === "done" && "bg-green-100 text-green-800"
      )}
    >
      {status === "in_progress" ? "In Progress" : status.charAt(0).toUpperCase() + status.slice(1)}
    </span>
  );
}
```

Test it by rendering `<StatusBadge status="todo" />`, `<StatusBadge status="in_progress" />`, `<StatusBadge status="done" />`.

**Checkpoint:** You can apply conditional styles cleanly using `cn()`.

---

## Exercise 4: Hover and Focus States

Build a button with proper interactive states:

```tsx
export default function ActionButton({
  children,
  variant = "primary",
}: {
  children: React.ReactNode;
  variant?: "primary" | "secondary" | "danger";
}) {
  return (
    <button
      className={cn(
        "px-4 py-2 rounded-md text-sm font-medium transition-colors",
        "focus:outline-none focus:ring-2 focus:ring-offset-2",
        variant === "primary" &&
          "bg-black text-white hover:bg-gray-800 focus:ring-gray-500",
        variant === "secondary" &&
          "bg-white text-gray-700 border border-gray-300 hover:bg-gray-50 focus:ring-gray-400",
        variant === "danger" &&
          "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500"
      )}
    >
      {children}
    </button>
  );
}
```

Render all three variants and tab through them to see focus styles.

**Checkpoint:** You can handle hover, focus, and variants cleanly.

---

## Reflection Questions

1. What does "utility-first" mean and how is it different from writing regular CSS classes?
2. What does `md:grid-cols-3` mean and when does it take effect?
3. Why use `cn()` instead of a template literal like `` `px-4 ${condition ? 'bg-red-500' : 'bg-blue-500'}` ``?
4. What's the difference between `text-sm` and `font-medium`?

---

Ready? Head to [MINI_PROJECT.md](./MINI_PROJECT.md)!