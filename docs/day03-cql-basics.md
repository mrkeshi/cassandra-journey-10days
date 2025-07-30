# Day 03 – Getting Started with CQL

Before diving into **CQL (Cassandra Query Language)**, it's helpful to keep a few things in mind:

- **Cassandra does not support** `DEFAULT` **values.**  
  If a value is not provided during insertion, it will simply be **omitted** — there's no default behavior like in traditional RDBMS.

- **Cassandra is fast, simple, and highly scalable**, but it does **not support all features** found in relational databases (e.g., `DEFAULT`, `NOT NULL`).  
  Instead, **data validation and default value handling** are expected to be **managed at the application level**.

- In Cassandra, a **`NULL` essentially means the column was not set (i.e., not stored).**  
  It's not the same as a SQL-style `NULL` — it's more like the column **doesn't exist** for that row.

✅ In the following sections, we’ll cover the full range of **CRUD operations** (Create, Read, Update, Delete) using CQL.


## ✍️ Create Table and Keyspace

📥 In this section, we’ll explore how to **create a keyspace**, define **Cassandra tables**, and understand the **structure of primary keys**.

---

### 🔹 Creating a Keyspace

A *keyspace* in Cassandra is similar to a database in relational systems. It defines how data is replicated and stored across nodes.

```sql
CREATE KEYSPACE IF NOT EXISTS my_keyspace 
WITH replication = {
  'class': 'SimpleStrategy', 
  'replication_factor': 1

};
AND durable_writes = true;
```

```
USE my_keyspace;
```
- `USE my_keyspace`  This command sets the current keyspace for your session, so all subsequent CQL commands will operate within this keyspace unless otherwise specified.
-  `durable_writes` (default: true) ensures that writes are first recorded to the commit log on disk before being acknowledged.
- `SimpleStrategy` is recommended only for single-node or development setups.  
- `replication_factor` defines how many copies of data are stored. For production, use `NetworkTopologyStrategy`.

---

### 🔹 Creating a Table

Let’s create a `users` table with various data types:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    age INT,
    is_active BOOLEAN,
    signup_date TIMESTAMP,
    interests LIST<TEXT>,
    settings MAP<TEXT, TEXT>
);
```

#### 🔸 Field Types Used:

| Type       | Description                               |
|------------|-------------------------------------------|
| `UUID`     | Universally Unique Identifier             |
| `TEXT`     | UTF-8 encoded string                      |
| `INT`      | Integer                                   |
| `BOOLEAN`  | True/False                                |
| `TIMESTAMP`| Date and time                             |
| `LIST<T>`  | Ordered collection of elements (duplicates allowed) |
| `MAP<K,V>` | Key-value structure                       |

> 💡 Cassandra also supports `SET<TEXT>` (like `LIST` but unordered and no duplicates).

---

### 🔹 Understanding PRIMARY KEY

The `PRIMARY KEY` in Cassandra determines both **data distribution** and **clustering (ordering)** within partitions.

```sql
PRIMARY KEY ((Partition_Key_Columns), Clustering_Columns)
```

#### ✔ Short Answer:

- Cassandra allows only **one Partition Key**.  
- But it can be **composite**, made up of **multiple columns**.

#### 🧠 Full Explanation:

The `PRIMARY KEY` is divided into two parts:

- **Partition Key** → Determines *which node* stores the data.
- **Clustering Columns** → Determines *how data is sorted* within the partition.

**Example:**

```sql
CREATE TABLE example (
    country TEXT,
    city TEXT,
    name TEXT,
    age INT,
    PRIMARY KEY ((country, city), name)
);
```

- **Partition Key:** `(country, city)` → Data is partitioned based on this composite key.
- **Clustering Key:** `name` → Data is sorted by name within each partition.

So all rows with the same `country` and `city` are stored together, sorted by `name`.

---

### ⚠️ Case Sensitivity in Cassandra

Cassandra is **not case-sensitive** by default:

```sql
SELECT name FROM users;
SELECT NAME FROM USERS;
SELECT NaMe FROM UsErS;
```

All of the above are treated the same.

> ❗ If you use double quotes (e.g., `"Name"`), then the identifier becomes **case-sensitive**. Avoid using quotes unless necessary.

**Example:**

```sql
CREATE TABLE "USERS" (
    "Name" TEXT PRIMARY KEY,
    age INT
);

-- This works:
SELECT "Name" FROM "USERS";

-- These will cause errors because the identifiers are case-sensitive when quoted:
SELECT name FROM users;
SELECT "name" FROM "USERS";
SELECT "Name" FROM users;

---


## ✍️ INSERT Data in Cassandra

📥 In this section, we’ll explore how to **insert data into tables** using CQL, including single-row inserts and batch inserts.

---

### 🔹 Basic INSERT

You can insert a single row using the `INSERT INTO` statement:

```sql
INSERT INTO users (
    id, name, email, age, is_active, signup_date, interests, settings
) VALUES (
    uuid(), 'Ali', 'ali@example.com', 30, true, toTimestamp(now()), ['coding', 'reading'], {'theme': 'dark', 'lang': 'fa'}
);
```

- `uuid()` generates a unique identifier.
- `toTimestamp(now())` inserts the current date and time.
- `LIST` and `MAP` values can be passed using `[ ]` and `{ }`.

---

### 🔹 Partial INSERT

You don't need to provide all columns — only those you want to set:

```sql
INSERT INTO users (id, name, email) 
VALUES (uuid(), 'Sara', 'sara@example.com');
```

Missing columns (like `age`, `signup_date`, etc.) will be treated as **unset** — they won't be stored.

---

### 🔹 BATCH INSERT

Use `BEGIN BATCH` to group multiple insert statements together:

```sql
BEGIN BATCH
INSERT INTO users (id, name, email, age) VALUES (uuid(), 'Reza', 'reza@example.com', 25);
INSERT INTO users (id, name, email, age) VALUES (uuid(), 'Niloofar', 'niloofar@example.com', 28);
INSERT INTO users (id, name, email, age) VALUES (uuid(), 'Mehdi', 'mehdi@example.com', 31);
APPLY BATCH;
```

> ⚠️ Batching is useful for atomic inserts **within the same partition**. Avoid large or cross-partition batches for performance.

---

### 🔎 Sample Output Table

After inserting the records, a query like:

```sql
SELECT * FROM users;
```

May return:

| id                                   | name     | email               | age | is_active | signup_date         | interests             | settings                         |
|--------------------------------------|----------|---------------------|-----|-----------|----------------------|------------------------|-----------------------------------|
| 550e8400-e29b-41d4-a716-446655440000 | Ali      | ali@example.com     | 30  | true      | 2025-07-30 14:20:15  | ['coding', 'reading']  | {'theme': 'dark', 'lang': 'fa'}  |
| 8b3c8d2e-3d47-4ae1-9936-4ed2ee7a1234 | Sara     | sara@example.com    | null| null      | null                 | null                   | null                              |
| 72f8d9f7-9841-4d35-9a15-47f8a7e9b567 | Reza     | reza@example.com    | 25  | null      | null                 | null                   | null                              |
| 4f2a1c17-6c6b-4d2d-a1d2-1a2f3d45e123 | Niloofar | niloofar@example.com| 28  | null      | null                 | null                   | null                              |
| 1a2b3c4d-5e6f-7a8b-9c0d-112233445566 | Mehdi    | mehdi@example.com   | 31  | null      | null                 | null                   | null     |

> 💡 Use `SELECT column1, column2 FROM users;` to fetch only specific fields.

---

### ✅ Summary

- `INSERT INTO` is used to add new rows to a table.
- You can insert full or partial rows.
- Use `BEGIN BATCH` to insert multiple rows together.
- Cassandra doesn't support auto-increment or default values — all logic must come from the application.


---


> ❗ Cassandra does **not** support SQL-style multi-row `INSERT` syntax.  
> For example, this is **not valid** in Cassandra:
>
> ```sql
> INSERT INTO users (id, name) VALUES
>   (uuid(), 'Ali'),
>   (uuid(), 'Sara');
> ```
> Each `INSERT` must insert **only one row at a time**. Use `BEGIN BATCH` if you want to group multiple inserts.

---


## 🔍 SELECT Queries in Cassandra

In this section, we’ll explore how to **query data** from a Cassandra table using CQL `SELECT` statements. We’ll look at best practices, limitations, and query patterns.

---

### 📌 Basic SELECT

```sql
SELECT * FROM users;
```

Fetches all columns from all rows in the `users` table.

```sql
SELECT id, name, email FROM users;
```

Fetches only the specified columns.

---

### 🎯 Query by Partition Key

```sql
SELECT * FROM users WHERE id = some_uuid;
```

This is the most efficient way to query in Cassandra — **by partition key**.  
Cassandra knows exactly where the data is and fetches it directly from the correct node.

---

### ❌ Query Without Partition Key (Bad Practice)

```sql
SELECT * FROM users WHERE name = 'Ali';
```

😬 This query **does not use the partition key**, so Cassandra must **scan all nodes** to find the result.

> ⚠️ Cassandra uses a distributed architecture with a **Token Ring** and **Partitioner + Token Map**, which works like a hash table to locate data. Without a partition key, it cannot use that optimization.

---

### 🔁 Query with `IN`

```sql
SELECT * FROM users WHERE id IN (uuid1, uuid2, uuid3);
```

Allows fetching multiple known partitions at once.  
> ✅ Acceptable for small lists — not recommended for large-scale scans.

---

### 📉 LIMIT

```sql
SELECT * FROM users LIMIT 10;
```

Fetches only the first 10 rows — **useful for pagination**, but does not guarantee global ordering across partitions.

---

### 🔃 ORDER BY (Within Partition Only)

```sql
SELECT * FROM users WHERE id = some_partition_key ORDER BY age DESC;
```

- `ORDER BY` only works **within a single partition**.
- The column used for ordering **must be a clustering column**.

> ❗ Using `ORDER BY` without filtering by the full partition key will cause an error.

---

### 🧪 ALLOW FILTERING

```sql
SELECT * FROM users WHERE age > 25 ALLOW FILTERING;
```

- Lets you filter by **non-primary key columns**.
- ⚠️ This may cause Cassandra to **scan the entire dataset** and is **not efficient**.

> 💡 Only use `ALLOW FILTERING` if you **know the data is small** or the performance impact is acceptable.

---

### 📦 Accessing Collection Elements

**From `LIST`:**

```sql
SELECT interests[0] FROM users WHERE id = some_uuid;
```

Returns the first interest from the `interests` list.

**From `MAP`:**

```sql
SELECT settings['theme'] FROM users WHERE id = some_uuid;
SELECT keys(settings) FROM users WHERE id = some_uuid;
SELECT values(settings) FROM users WHERE id = some_uuid;
```

- Access specific values
- Retrieve all keys or all values from a map

---
## ✏️ UPDATE & DELETE Queries in Cassandra

This guide explains how to update and delete data in Cassandra using CQL. While the syntax may look similar to SQL, there are important limitations and best practices to follow.

---

### 🛠️ UPDATE

```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE partition_key = value
  [AND clustering_column = value ...];
```

> ✅ You must specify the **partition key** in the `WHERE` clause.  
> ✅ If the table uses a **clustering column**, you must also provide its value to update a specific row.

---

### 📦 Example: Orders Table

```sql
CREATE TABLE orders (
    user_id UUID,
    order_date TIMESTAMP,
    status TEXT,
    total DECIMAL,
    PRIMARY KEY ((user_id), order_date)
);
```

---

#### ✅ Update a specific order:

```sql
UPDATE orders
SET status = 'shipped'
WHERE user_id = some_uuid_value AND order_date = some_timestamp_value;
```

---

### ⚠️ UPDATE Limitations

- ❌ You **cannot** update rows without specifying the **partition key**.
- ❌ You **cannot** change the value of the **partition key** itself.
- ❗ To update a specific row in tables with clustering columns, you **must** specify the full clustering key.

---

### ❌ Invalid Update Example

```sql
UPDATE orders SET status = 'pending' WHERE status = 'shipped';
```

This is invalid — `status` is not a primary key.

---

### 🗑️ DELETE

```sql
DELETE FROM table_name
WHERE partition_key = value
  [AND clustering_column = value ...];
```

Deletes a specific row if clustering column is provided. Deletes the whole partition if only the partition key is given.

---

#### ✅ Delete a specific order:

```sql
DELETE FROM orders
WHERE user_id = some_uuid_value AND order_date = some_timestamp_value;
```

#### 🧹 Delete an entire partition:

```sql
DELETE FROM orders
WHERE user_id = some_uuid_value;
```

> ⚠️ This removes **all rows** with that `user_id`.

---

### 🧩 Removing Items from Collections

```sql
UPDATE users SET interests = interests - ['music'] WHERE id = some_uuid_value;
```

Removes `'music'` from the `interests` list for the specified user.

---

> 📌 Always design your queries around how Cassandra distributes and stores data — prioritize reads by partition key and avoid filtering large datasets.
