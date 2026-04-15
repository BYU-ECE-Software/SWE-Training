# 05 — PostgreSQL

## What is PostgreSQL?

PostgreSQL (Postgres) is the relational database we use on every project. A relational database stores data in **tables** — rows and columns, like a spreadsheet — and lets you express relationships between tables using **foreign keys**.

Prisma (module 04) abstracts most direct database interaction, but understanding what's happening underneath makes you a much better developer. When a query is slow, when a migration fails, or when you need to inspect production data quickly, you'll need to think in SQL.

---

## Core Concepts

### Tables, Rows, and Columns

A table is a named collection of rows. Every row has the same set of columns, each with a defined data type.

```sql
CREATE TABLE tasks (
  id          SERIAL PRIMARY KEY,
  title       TEXT NOT NULL,
  status      TEXT NOT NULL DEFAULT 'todo',
  assignee_id INTEGER REFERENCES users(id),
  created_at  TIMESTAMP DEFAULT NOW()
);
```

- `SERIAL` — auto-incrementing integer
- `PRIMARY KEY` — uniquely identifies each row
- `NOT NULL` — the column must have a value
- `REFERENCES users(id)` — this is a foreign key; the value must exist in `users.id`

### Primary Keys

Every table should have a primary key — a column (or set of columns) that uniquely identifies each row. We always use `id SERIAL PRIMARY KEY` (or `id BIGSERIAL PRIMARY KEY` for large tables).

### Foreign Keys

A foreign key is a column in one table that references the primary key of another. It enforces that a relationship is valid — you can't set `assignee_id = 999` if user 999 doesn't exist.

```
users                    tasks
────────────             ──────────────────────
id | name                id | title   | assignee_id
───┼──────               ───┼─────────┼────────────
1  | Alex                1  | Task A  | 1  ← references users.id=1
2  | Sam                 2  | Task B  | 2  ← references users.id=2
                         3  | Task C  | NULL (unassigned)
```

### Indexes

An index is a data structure that speeds up lookups on a column. Without an index, finding all tasks for user 5 requires scanning every row. With an index on `assignee_id`, Postgres can jump directly to the matching rows.

```sql
-- Create an index
CREATE INDEX idx_tasks_assignee_id ON tasks(assignee_id);

-- Prisma creates indexes on foreign keys automatically
```

The trade-off: indexes speed up reads but slow down writes (the index must be updated on every insert/update/delete). Index columns you filter or join on frequently.

---

## Essential SQL

### SELECT

```sql
-- All tasks
SELECT * FROM tasks;

-- Specific columns
SELECT id, title, status FROM tasks;

-- With a filter
SELECT * FROM tasks WHERE status = 'todo';

-- Multiple conditions
SELECT * FROM tasks WHERE status = 'todo' AND assignee_id IS NOT NULL;

-- Order and limit
SELECT * FROM tasks ORDER BY created_at DESC LIMIT 10;
```

### JOINs

A JOIN combines rows from two tables based on a condition — typically a foreign key relationship.

```sql
-- INNER JOIN: only rows that match in both tables
SELECT
  tasks.title,
  tasks.status,
  users.name AS assignee_name
FROM tasks
INNER JOIN users ON tasks.assignee_id = users.id;

-- LEFT JOIN: all tasks, even unassigned ones (assignee_name will be NULL)
SELECT
  tasks.title,
  users.name AS assignee_name
FROM tasks
LEFT JOIN users ON tasks.assignee_id = users.id;
```

### INSERT, UPDATE, DELETE

```sql
-- Insert
INSERT INTO tasks (title, status) VALUES ('Write tests', 'todo');

-- Insert and return the new row
INSERT INTO tasks (title, status) VALUES ('Write tests', 'todo') RETURNING *;

-- Update
UPDATE tasks SET status = 'done' WHERE id = 3;

-- Delete
DELETE FROM tasks WHERE id = 3;
```

### Aggregates

```sql
-- Count all tasks
SELECT COUNT(*) FROM tasks;

-- Count by status
SELECT status, COUNT(*) AS total
FROM tasks
GROUP BY status;

-- Average, sum, min, max
SELECT
  AVG(id) AS avg_id,
  MIN(created_at) AS oldest,
  MAX(created_at) AS newest
FROM tasks;
```

---

## Connecting via psql

`psql` is the Postgres command-line client. It's useful for quick inspection and running queries.

```bash
# Connect to Postgres running in your Docker container
docker exec -it task-tracker-db-1 psql -U dev -d tasktracker
```

Once inside:

```sql
\dt              -- list tables
\d tasks         -- describe the tasks table (columns, types, constraints)
\d+ tasks        -- more detail including indexes
\l               -- list databases
\c other_db      -- connect to a different database
\q               -- quit
```

---

## What Prisma Generates

When you run `prisma migrate dev`, Prisma generates SQL and applies it. You can inspect the migrations:

```bash
cat prisma/migrations/*/migration.sql
```

You'll see real `CREATE TABLE` and `ALTER TABLE` statements — knowing SQL means you can understand exactly what Prisma is doing, which helps when migrations fail or behave unexpectedly.

---

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) to practice SQL hands-on, then [MINI_PROJECT.md](./MINI_PROJECT.md) to inspect the Task Tracker schema directly in Postgres.