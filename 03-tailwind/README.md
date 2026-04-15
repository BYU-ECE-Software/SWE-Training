# 03 — Tailwind CSS

## What is Tailwind?

Tailwind is a utility-first CSS framework. Instead of writing CSS classes like `.card { padding: 1rem; border-radius: 8px; }`, you compose styles directly in your HTML using small, single-purpose utility classes: `className="p-4 rounded-lg"`.

This feels weird at first — especially if you're used to "separation of concerns" between HTML and CSS. But it solves real problems:

- **No naming things.** Coming up with good class names for everything is surprisingly hard and time-consuming.
- **No dead CSS.** Unused styles don't pile up because styles are colocated with the elements that use them.
- **Consistent design.** Tailwind's spacing, color, and typography scales are built-in. `p-4` is always 16px. You stop making one-off decisions.
- **Easy to read diffs.** When a style changes, you see it right on the element.

---

## Core Concepts

### Spacing and Sizing

Tailwind uses a numeric scale where each unit is 4px.

| Class | Value |
|-------|-------|
| `p-1` | padding: 4px |
| `p-2` | padding: 8px |
| `p-4` | padding: 16px |
| `p-6` | padding: 24px |
| `p-8` | padding: 32px |

The same scale applies to margin (`m-`), width (`w-`), height (`h-`), gap, and more.

Directional variants: `px-4` (horizontal), `py-4` (vertical), `pt-4` (top), `pb-4` (bottom), `pl-4` (left), `pr-4` (right).

### Colors

Tailwind has a built-in color palette with numeric shades from 50 (lightest) to 950 (darkest).

```html
<div class="bg-blue-100 text-blue-900">Light blue card</div>
<div class="bg-red-500 text-white">Red button</div>
<div class="text-gray-600">Muted text</div>
```

For borders: `border border-gray-200` (border with color), `border-2` (thickness).

### Flexbox and Grid

```html
<!-- Flexbox row, centered vertically, space between -->
<div class="flex items-center justify-between">
  <span>Left</span>
  <span>Right</span>
</div>

<!-- Flexbox column, gap between items -->
<div class="flex flex-col gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

<!-- CSS Grid, 3 equal columns, gap -->
<div class="grid grid-cols-3 gap-4">
  <div>Col 1</div>
  <div>Col 2</div>
  <div>Col 3</div>
</div>
```

### Typography

```html
<h1 class="text-2xl font-semibold">Heading</h1>
<p class="text-sm text-gray-600">Muted body text</p>
<span class="font-mono text-xs">Code</span>
```

Text sizes: `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`, `text-3xl`, ...
Font weights: `font-normal`, `font-medium`, `font-semibold`, `font-bold`

### Borders and Rounded Corners

```html
<div class="border border-gray-200 rounded-lg p-4">Card</div>
<button class="rounded-full px-4 py-2">Pill button</button>
```

### Hover, Focus, and State Variants

Prefix any utility with a variant name followed by `:`:

```html
<button class="bg-black text-white hover:bg-gray-800 focus:ring-2 focus:ring-gray-400">
  Click me
</button>

<input class="border border-gray-300 focus:border-blue-500 focus:outline-none" />
```

### Responsive Design

Tailwind is mobile-first. Unprefixed utilities apply to all screen sizes. Prefixed utilities apply at that breakpoint and above.

| Prefix | Breakpoint |
|--------|-----------|
| `sm:` | 640px+ |
| `md:` | 768px+ |
| `lg:` | 1024px+ |
| `xl:` | 1280px+ |

```html
<!-- 1 column on mobile, 3 columns on md screens and up -->
<div class="grid grid-cols-1 md:grid-cols-3 gap-4">
```

### Dark Mode

With `darkMode: 'class'` in `tailwind.config.ts`, prefix utilities with `dark:`:

```html
<div class="bg-white dark:bg-gray-900 text-black dark:text-white">
  Adapts to dark mode
</div>
```

---

## Composing with `cn()`

When you have conditional classes, string concatenation gets messy. The `cn()` utility (using `clsx` + `tailwind-merge`) is the standard solution:

```bash
npm install clsx tailwind-merge
```

```ts
// lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```tsx
import { cn } from "@/lib/utils";

function Badge({ status }: { status: "todo" | "done" | "in_progress" }) {
  return (
    <span
      className={cn(
        "px-2 py-0.5 rounded-full text-xs font-medium",
        {
          "bg-gray-100 text-gray-700": status === "todo",
          "bg-yellow-100 text-yellow-800": status === "in_progress",
          "bg-green-100 text-green-800": status === "done",
        }
      )}
    >
      {status}
    </span>
  );
}
```

---

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) to practice building components, then [MINI_PROJECT.md](./MINI_PROJECT.md) to polish the Task Tracker's UI.