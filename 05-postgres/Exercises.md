# 05 — PostgreSQL: Exercises & Mini Project

## Prerequisites

Your Postgres container from module 01 should be running:
```bash
cd task-tracker && docker compose up -d db
```

Connect to it:
```bash
docker exec -it task-tracker-db-1 psql -U dev -d tasktracker
```

---

## Exercise 1: Explore the Schema

Inspect what Prisma created for you:

```sql
-- List all tables
\dt

-- Describe the tasks table
\d tasks

-- Describe the users table
\d users

-- See indexes Prisma created
\di
```

**Questions to answer:**
- What data type is the `status` column? (Hint: Prisma creates a Postgres enum)
- What constraints does `email` on `users` have?
- What index did Prisma create on `tasks.assignee_id`?

**Checkpoint:** You can navigate a database schema using psql.

---

## Exercise 2: Basic Queries

Run each of these — predict the output before running:

```sql
-- 1. All tasks
SELECT * FROM tasks;

-- 2. Just titles and statuses
SELECT title, status FROM tasks;

-- 3. Only unassigned tasks
SELECT title FROM tasks WHERE assignee_id IS NULL;

-- 4. Tasks ordered by creation, newest first
SELECT title, created_at FROM tasks ORDER BY created_at DESC;

-- 5. How many tasks are in each status?
SELECT status, COUNT(*) AS total
FROM tasks
GROUP BY status
ORDER BY total DESC;
```

**Checkpoint:** You can write SELECT queries with WHERE, ORDER BY, and GROUP BY.

---

## Exercise 3: JOINs

```sql
-- 1. All tasks with their assignee's name (inner join — excludes unassigned)
SELECT tasks.title, tasks.status, users.name AS assignee
FROM tasks
INNER JOIN users ON tasks.assignee_id = users.id;

-- 2. All tasks including unassigned (left join)
SELECT tasks.title, tasks.status, users.name AS assignee
FROM tasks
LEFT JOIN users ON tasks.assignee_id = users.id;

-- 3. Each user and how many tasks they're assigned
SELECT users.name, COUNT(tasks.id) AS task_count
FROM users
LEFT JOIN tasks ON tasks.assignee_id = users.id
GROUP BY users.name
ORDER BY task_count DESC;
```

Notice the difference between results 1 and 2. The `LEFT JOIN` includes tasks where `assignee_id IS NULL`; the `INNER JOIN` does not.

**Checkpoint:** You understand the difference between INNER JOIN and LEFT JOIN.

---

## Exercise 4: INSERT, UPDATE, DELETE

```sql
-- 1. Add a new task
INSERT INTO tasks (title, status) VALUES ('Test SQL queries', 'TODO') RETURNING *;

-- 2. Assign it to user 1
UPDATE tasks SET assignee_id = 1 WHERE title = 'Test SQL queries' RETURNING *;

-- 3. Mark it done
UPDATE tasks SET status = 'DONE' WHERE title = 'Test SQL queries';

-- 4. Delete it
DELETE FROM tasks WHERE title = 'Test SQL queries';

-- 5. Confirm it's gone
SELECT COUNT(*) FROM tasks WHERE title = 'Test SQL queries';
```

**Checkpoint:** You can perform full CRUD in raw SQL.

---

## Exercise 5: EXPLAIN

Postgres can show you how it executes a query. This is useful for understanding performance.

```sql
-- Show the query plan
EXPLAIN SELECT * FROM tasks WHERE assignee_id = 1;

-- Show the plan WITH actual execution times (runs the query)
EXPLAIN ANALYZE SELECT * FROM tasks WHERE assignee_id = 1;
```

Look for `Index Scan` vs `Seq Scan`. An index scan uses an index (fast for large tables). A sequential scan reads every row. With only a few rows, Postgres might choose a seq scan even with an index — that's normal.

**Checkpoint:** You know how to read a basic query plan.

---

## Mini Project: Inspect and Query the Task Tracker

With the seed data from module 04 loaded, answer these questions using raw SQL:

1. How many tasks are in each status? (use GROUP BY)
2. Which user has the most tasks assigned?
3. List all tasks that are either `IN_PROGRESS` or `DONE`, with assignee names, ordered by status.
4. What would happen if you ran `DELETE FROM users WHERE id = 1` when that user has tasks assigned to them? Try it and read the error message. What constraint is protecting you?

<details>
<summary>Answers</summary>

```sql
-- 1. Tasks per status
SELECT status, COUNT(*) FROM tasks GROUP BY status;

-- 2. Most tasks
SELECT users.name, COUNT(tasks.id) AS total
FROM users
JOIN tasks ON tasks.assignee_id = users.id
GROUP BY users.name
ORDER BY total DESC
LIMIT 1;

-- 3. Active tasks with assignees
SELECT tasks.title, tasks.status, users.name AS assignee
FROM tasks
LEFT JOIN users ON tasks.assignee_id = users.id
WHERE tasks.status IN ('IN_PROGRESS', 'DONE')
ORDER BY tasks.status;

-- 4. The foreign key constraint on tasks.assignee_id will raise:
-- ERROR: update or delete on table "users" violates foreign key constraint
-- "tasks_assignee_id_fkey" on table "tasks"
-- You'd need to reassign or delete the tasks first, or use ON DELETE CASCADE.
```
</details>

---

## Reflection Questions

1. What's the difference between a primary key and a foreign key?
2. What's the difference between `INNER JOIN` and `LEFT JOIN`?
3. If you had a table with 10 million tasks and you frequently query `WHERE assignee_id = ?`, what would you add and why?
4. Prisma abstracts SQL away. Why is it still worth understanding raw SQL?

---

Move on to [06 — Nginx](../06-nginx/README.md)!