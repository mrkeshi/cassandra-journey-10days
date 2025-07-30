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

