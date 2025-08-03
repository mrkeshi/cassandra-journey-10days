# Day 06 - Mastering TTL, Secondary Index, and Built-in Functions in Apache Cassandra

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)

Welcome to **Day 6** of our Apache Cassandra learning journey! 🎉 Today, we’ll dive into three essential topics: **TTL (Time To Live)**, **Secondary Indexes**, and **Built-in Functions** in Cassandra Query Language (CQL). These features empower you to manage data expiration, query non-primary key columns, and leverage powerful built-in functions for dynamic data handling. 🚀

By now, you’re familiar with Cassandra’s core concepts, data modeling, and collections. In this guide, we’ll explore these topics step-by-step with detailed explanations, practical examples, and exercises to solidify your understanding. Let’s make it fun, practical, and beginner-friendly! 💪

---

## 📖 Introduction to TTL, Secondary Indexes, and Built-in Functions

Here’s a quick overview of today’s topics:

- **TTL (Time To Live)**: Defines how long a piece of data (a column or cell) remains in Cassandra before it’s automatically deleted. Perfect for temporary data like OTP codes or session tokens.
- **Secondary Index**: Allows querying on non-primary key columns, making your queries more flexible but with performance considerations.
- **Built-in Functions**: CQL provides functions like `TTL()`, `WRITETIME()`, `now()`, and `count()` to manipulate and retrieve metadata or perform calculations.

In this guide, we’ll:
- Explain each concept with beginner-friendly details 📝
- Provide practical examples with real-world scenarios 🛠️
- Highlight best practices and limitations ⚠️
- Include exercises to test your knowledge 🧠
- Answer common questions (e.g., “Is TTL applied to rows or columns?”)

Let’s get started! 🚀

---

## 🔑 Section 1: TTL (Time To Live) – Managing Data Expiration

### ❓ What is TTL?

TTL (Time To Live) in Cassandra specifies how long a specific piece of data (a column or cell) remains valid before it’s automatically deleted. Once the TTL expires, Cassandra marks the data as a *tombstone* and removes it during compaction.

### 🎯 Why is TTL Important?

TTL is crucial for:
- **Automatic cleanup**: Removes temporary data like OTP codes, session tokens, or cached results.
- **Storage optimization**: Prevents unnecessary growth of your database.
- **Use cases**: Temporary notifications, user sessions, or promotional data.

### 🛠️ Practical Examples

Let’s explore TTL with a sample table for storing login verification codes.

#### 📦 Table Definition

```sql
CREATE TABLE login_codes (
    phone TEXT PRIMARY KEY,
    code TEXT
);
```

#### ✅ Example 1: Insert a Row with TTL

Insert a verification code that expires after 5 minutes (300 seconds).

```sql
INSERT INTO login_codes (phone, code)
VALUES ('09123456789', '8293')
USING TTL 300;
```

**What’s Happening?**
- The row (`phone`, `code`) will be deleted automatically after 300 seconds.
- Useful for one-time password (OTP) systems.

#### ✅ Example 2: Update a Column with TTL

Update the code for a phone number with a new TTL of 10 minutes (600 seconds).

```sql
UPDATE login_codes
USING TTL 600
SET code = '5932'
WHERE phone = '09123456789';
```

**What’s Happening?**
- Only the `code` column gets a TTL of 600 seconds.
- The `phone` column (primary key) remains unaffected.

#### ✅ Example 3: Set Default TTL for a Table

Create a table for temporary files with a default TTL of 24 hours (86400 seconds).

```sql
CREATE TABLE temp_files (
    id UUID PRIMARY KEY,
    filename TEXT,
    uploaded_at TIMESTAMP
)
WITH default_time_to_live = 86400;
```

**What’s Happening?**
- Any row inserted into this table will expire after 24 hours unless a custom TTL is specified.
- Ideal for temporary file metadata or logs.

#### ✅ Example 4: Check Remaining TTL

Query the remaining TTL for a specific column.

```sql
SELECT TTL(code)
FROM login_codes
WHERE phone = '09123456789';
```

**Expected Output**:
If 100 seconds remain, the query returns `100`.

#### ✅ Example 5: TTL on Specific Columns

Insert a row where only certain columns have TTL.

```sql
CREATE.TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);

INSERT INTO users (id, name, email)
VALUES (uuid(), 'Alireza', 'ali@example.com')
USING TTL 300;
```

**What’s Happening?**
- Both `name` and `email` columns get a TTL of 300 seconds.
- Now, update only the `email` column:

```sql
UPDATE users
USING TTL 600
SET email = 'kos@gmail.com'
WHERE id = <your-uuid>;
```

**Result**:
- `email` now has a TTL of 600 seconds.
- `name` retains its original TTL of 300 seconds.

### 🧠 Key Insight: TTL is Column-Level

- TTL is applied at the **column (cell) level**, not the entire row.
- Only columns updated or inserted in the query with the `USING TTL` clause receive the TTL.
- If you want an entire row to expire, ensure all non-primary key columns are included in the `INSERT` or `UPDATE` with TTL.

### ⚠️ Best Practices and Limitations

- **Use TTL for temporary data**: Ideal for short-lived data like OTPs or cache entries.
- **Avoid large TTL values**: Very long TTLs (e.g., years) can lead to tombstone buildup, impacting performance.
- **Monitor tombstones**: Expired data becomes tombstones, which are cleaned up during compaction. Ensure proper compaction settings.
- **Default TTL**: Use `default_time_to_live` for tables with predictable expiration needs.

---

## 🔑 Section 2: Secondary Index – Querying Non-Primary Key Columns

### ❓ What is a Secondary Index?

In Cassandra, queries are typically restricted to the **primary key** or **partition key** for performance reasons. A **Secondary Index** allows you to query non-primary key columns, but it comes with trade-offs.

### 🎯 Why Use Secondary Indexes?

- Query flexibility: Search on columns that aren’t part of the primary key.
- Use cases: Filtering users by city, email, or other attributes.

### ⚠️ Warning

- Secondary Indexes are **not suitable** for high-cardinality columns (e.g., columns with many unique values like `email`) or very large datasets.
- They can cause performance issues in distributed environments due to cross-node scans.

### 🛠️ Practical Examples

Let’s create a `users` table and add a Secondary Index.

#### 📦 Table Definition

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    city TEXT
);
```

#### ✅ Example 1: Create a Secondary Index

Add an index on the `city` column to allow queries by city.

```sql
CREATE INDEX ON users (city);
```

#### ✅ Example 2: Query Using Secondary Index

Now you can query by `city` without it being part of the primary key.

```sql
SELECT * FROM users WHERE city = 'Tehran';
```

**What’s Happening?**
- The Secondary Index enables this query, which would otherwise throw an error.
- Without the index, you’d get: `InvalidQueryException: Cannot execute this query as it might involve data from multiple partitions`.

#### ✅ Example 3: Query Without Index (Error Case)

Try querying `city` without an index:

```sql
SELECT * FROM users WHERE city = 'Tehran';
```

**Expected Output**:
- Error: `InvalidQueryException: No secondary index found for column city`.

#### ✅ Example 4: Drop a Secondary Index

If you no longer need the index:

```sql
DROP INDEX users_city_idx;
```

### 🧠 Best Practices for Secondary Indexes

- **Use sparingly**: Prefer partition key queries for performance.
- **Low cardinality**: Use Secondary Indexes on columns with few unique values (e.g., `city`, not `email`).
- **Alternatives**: Consider **materialized views** or denormalized tables for complex queries in large datasets.
- **Monitor performance**: Secondary Indexes can slow down writes and reads in large clusters.

---

## 🔑 Section 3: Built-in Functions in CQL

### ❓ What are Built-in Functions?

Cassandra provides a set of built-in functions in CQL to manipulate data, retrieve metadata, or perform calculations. These are especially useful for dynamic queries and debugging.

### 📌 Common Built-in Functions

| Function            | Description                                      | Example                              |
|---------------------|--------------------------------------------------|--------------------------------------|
| `TTL(column)`       | Returns the remaining TTL for a column (in seconds) | `SELECT TTL(code) FROM login_codes` |
| `WRITETIME(column)` | Returns the timestamp when a column was written   | `SELECT WRITETIME(name) FROM users` |
| `now()`             | Generates a time-based UUID                      | `INSERT INTO users (id) VALUES (now())` |
| `toTimestamp(uuid)` | Converts a UUID to a timestamp                   | `SELECT toTimestamp(now())`         |
| `count(*)`          | Counts the number of rows in a query result      | `SELECT count(*) FROM users`        |
| `dateOf(uuid)`      | Extracts the date from a time-based UUID         | `SELECT dateOf(id) FROM users`      |

### 🛠️ Practical Examples

#### ✅ Example 1: Check Write Timestamp

Query the timestamp when a user’s `email` was last updated.

```sql
SELECT WRITETIME(email)
FROM users
WHERE id = <your-uuid>;
```

**Expected Output**:
- A timestamp like `2025-08-03 13:01:23.123+0000`.

#### ✅ Example 2: Count Rows in a Table

Count the total number of users.

```sql
SELECT count(*)
FROM users;
```

**Expected Output**:
- A number, e.g., `42`.

#### ✅ Example 3: Generate a UUID with `now()`

Insert a new user with a time-based UUID.

```sql
INSERT INTO users (id, name)
VALUES (now(), 'Ali');
```

**What’s Happening?**
- `now()` generates a unique UUID based on the current timestamp.

#### ✅ Example 4: Extract Date from UUID

Get the date from a time-based UUID.

```sql
SELECT dateOf(id)
FROM users
WHERE id = <your-uuid>;
```

**Expected Output**:
- A date like `2025-08-03`.

### 🧠 Best Practices for Built-in Functions

- **Use for metadata**: Functions like `TTL()` and `WRITETIME()` are great for debugging or auditing.
- **Combine with queries**: Use `count(*)` or `now()` in application logic for dynamic operations.
- **Avoid overuse**: Functions like `count(*)` can be slow on large tables without proper partitioning.

---

## 🧪 Exercise: Apply What You’ve Learned

Let’s test your understanding with a practical exercise.

### 📝 Task: Create a Notifications Table with TTL

Create a table called `notifications` to store temporary messages that expire after 1 hour (3600 seconds). Then, insert a sample notification.

#### 📦 Table Definition

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY,
    user_id UUID,
    message TEXT
);
```

#### ✅ Your Task

Write a CQL query to insert a notification that expires after 1 hour.

**Your Answer**:

```sql
INSERT INTO notifications (id, user_id, message)
VALUES (now(), uuid(), 'Your order has been shipped!')
USING TTL 3600;
```

**What’s Happening?**
- `now()` generates a time-based UUID for `id`.
- `uuid()` generates a random UUID for `user_id`.
- The `message` column expires after 3600 seconds (1 hour).

#### ✅ Bonus Task

Query the remaining TTL for the `message` column of the notification you just inserted.

**Your Answer**:

```sql
SELECT TTL(message)
FROM notifications
WHERE id = <your-uuid>;
```

---

## 🧠 Additional Exercises

1. **TTL Challenge**: Create a table for temporary user sessions with a default TTL of 30 minutes. Insert a session and verify its TTL.
2. **Secondary Index Challenge**: Add a Secondary Index to the `notifications` table on `user_id`. Write a query to find all notifications for a specific `user_id`.
3. **Function Challenge**: Write a query to retrieve both the `WRITETIME` and `dateOf` for the `message` column in the `notifications` table.

---

## ⚠️ Common Pitfalls and Best Practices

- **TTL**:
  - Don’t use TTL for permanent data; it’s meant for temporary data.
  - Be cautious with very short TTLs (e.g., <10 seconds) as they can create many tombstones.
- **Secondary Indexes**:
  - Avoid on high-cardinality columns (e.g., `email`, `timestamp`).
  - Use materialized views or denormalized tables for complex queries in large datasets.
- **Built-in Functions**:
  - Use `count(*)` sparingly on large tables to avoid performance hits.
  - Combine `now()` and `toTimestamp()` for time-based operations.

---

## 🚀 What’s Next?

Tomorrow, we’ll explore **Cassandra Materialized Views** and **Batch Operations** to take your data modeling and querying skills to the next level! Stay tuned for more practical examples and exercises. 🎉

If you have questions or want more examples, let me know, and I’ll tailor the content to your needs! 💬