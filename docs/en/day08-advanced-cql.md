# Day 08 - Advanced CQL Features in Apache Cassandra

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)

Welcome to **Day 8** of our Apache Cassandra learning journey! 🎉 Today, we’re diving into **Advanced CQL Features** that empower you to build real-time, flexible, and fault-tolerant applications. These tools are powerful but require a deep understanding to avoid common pitfalls, such as misusing `BATCH` statements or over-relying on secondary indexes. By mastering these features, you’ll unlock Cassandra’s full potential for handling complex use cases with precision and efficiency. 🚀

In this guide, we’ll:

- Explore advanced CQL features like `BATCH` statements, Lightweight Transactions (LWT), Materialized Views, User-Defined Types (UDTs), User-Defined Functions (UDFs), and JSON support 📝
- Provide practical examples tailored to a social media application 📱
- Highlight best practices and common mistakes to avoid ⚠️
- Include a bonus tip on safely dropping tables and keyspaces 🛠️

> **Note**: I forgot to mention earlier, but dropping tables and keyspaces is a critical operation that requires caution. To drop a table, use `DROP TABLE keyspace_name.table_name;`. To drop an entire keyspace, use `DROP KEYSPACE keyspace_name;`. Always ensure you have backups or are certain the data is no longer needed, as these operations are irreversible!

## Batch Statements – Executing Multiple Operations Together

### What Are Batch Statements?

Batch statements allow you to execute multiple `INSERT`, `UPDATE`, or `DELETE` operations as a single logical unit. They are useful for grouping related write operations, especially when they target the same partition key.

### Types of Batches

- **Logged Batch**: Ensures atomicity (all operations succeed or fail together) but is slower due to coordination overhead. Use for operations requiring consistency.
- **Unlogged Batch**: Faster, as it skips the coordination step, but does not guarantee atomicity. Ideal for performance-critical operations.
- **Counter Batch**: Specifically for tables with counter columns, used to update counters atomically.

### Example: Social Media Application

Imagine a social media app where a user posts a status update, and we need to update both the user’s profile and their timeline simultaneously.

```sql
BEGIN UNLOGGED BATCH
  INSERT INTO posts (user_id, post_id, content, created_at) 
  VALUES (1, uuid(), 'Hello, world!', '2025-08-05T12:00:00');
  UPDATE users SET post_count = post_count + 1 WHERE user_id = 1;
APPLY BATCH;
```

### Best Practices

- **Use for Same Partition Key**: Batches are most efficient when operations target the same partition key to avoid overloading the coordinator node.
- **Avoid Large Batches**: Large batches can strain the cluster. Limit to a few operations (e.g., &lt;10).
- **Choose Unlogged for Performance**: Use unlogged batches unless atomicity is critical.

### Common Mistake

Using a batch for operations across different partition keys can overload the coordinator node, leading to performance degradation. For example:

```sql
BEGIN BATCH
  INSERT INTO posts (user_id, post_id, content) VALUES (1, uuid(), 'Post 1');
  INSERT INTO posts (user_id, post_id, content) VALUES (2, uuid(), 'Post 2');
APPLY BATCH;
```

This is inefficient as it spans multiple partitions. Instead, execute these as separate statements.

## Lightweight Transactions (LWT) – Conditional Operations

### What Are Lightweight Transactions?

LWTs allow conditional operations, ensuring changes are applied only if a specific condition is met (e.g., "compare and set"). They use the Paxos protocol for consistency, making them slower but atomic and linearizable.

### Example: Preventing Duplicate Usernames

In our social media app, we want to ensure usernames are unique during registration.

```sql
INSERT INTO users (user_id, username, email) 
VALUES (1, 'ali123', 'ali@example.com') 
IF NOT EXISTS;
```

If a user with `user_id = 1` already exists, the operation fails, and the result might look like:

```
[applied] | user_id | username | email
----------|---------|----------|----------------
 false    | 1       | ali123   | ali@example.com
```

### Example: Conditional Update

Suppose we only want to update a user’s profile if their email hasn’t changed.

```sql
UPDATE users 
SET username = 'ali_new' 
WHERE user_id = 1 
IF email = 'ali@example.com';
```

If the email doesn’t match, the update fails:

```
[applied] | email
----------|----------------
 false    | wrong@email.com
```

### Key Characteristics

| Feature | Regular Write | LWT |
| --- | --- | --- |
| **Condition** | None, always executes | Executes only if condition is met |
| **Atomicity** | No | Yes (uses Paxos protocol) |
| **Speed** | Fast ⚡ | Slower 🐢 due to coordination |
| **Use Case** | Simple updates | Concurrency control, unique constraints |
| **Cluster Load** | Low | High |

### When to Use LWT

- **Use When**: Preventing race conditions, ensuring unique data (e.g., usernames), or implementing optimistic locking.
- **Avoid When**: Performance is critical, or conditions aren’t necessary.

## Materialized Views – Secondary Data Representation

### What Are Materialized Views?

Materialized Views create a secondary table with a different primary key or sort order, automatically populated from the base table. They’re useful for supporting additional query patterns without duplicating data manually.

### Example: Querying Users by Email

In our social media app, we want to query users by email instead of `user_id`.

```sql
CREATE MATERIALIZED VIEW users_by_email AS
  SELECT user_id, username, email 
  FROM users
  WHERE email IS NOT NULL AND user_id IS NOT NULL
  PRIMARY KEY (email, user_id);
```

Now, we can query:

```sql
SELECT * FROM users_by_email WHERE email = 'ali@example.com';
```

### Warning

In Cassandra 4.0 and later, Materialized Views are deprecated due to consistency issues. Consider alternatives:

- **Denormalization**: Create separate tables with different primary keys.
- **Secondary Indexes**: For low-cardinality columns.
- **External Search Engines**: Use Elasticsearch or Solr for complex queries.
- **Application-Level Logic**: Cache query results in the application layer.

## User-Defined Types (UDTs) – Complex Data Structures

### What Are UDTs?

UDTs allow you to define complex data structures (like structs) within a single column, reducing the need for separate tables.

### Example: Storing User Addresses

In our social media app, we want to store a user’s address as a structured object.

```sql
CREATE TYPE address (
  street text,
  city text,
  zip int
);

CREATE TABLE users (
  user_id int PRIMARY KEY,
  username text,
  addr address
);

INSERT INTO users (user_id, username, addr)
VALUES (1, 'ali123', {street: 'Valiasr', city: 'Tehran', zip: 12345});
```

### Best Practices

- Use UDTs for logically related data (e.g., addresses, coordinates).
- Avoid deeply nested UDTs, as they can complicate queries.

## User-Defined Functions (UDFs) and Aggregates – Custom Functions

### What Are UDFs?

UDFs let you define custom functions in Java or JavaScript to extend CQL’s functionality. Aggregates are custom aggregation functions.

### Example: Checking Even Numbers

In our social media app, we want to check if a user’s post count is even.

```sql
CREATE FUNCTION is_even(n int)
RETURNS NULL ON NULL INPUT
RETURNS boolean
LANGUAGE java
AS 'return (n % 2) == 0;';

SELECT user_id, post_count, is_even(post_count) AS is_even 
FROM users;
```

### Example: String Length

Calculate the length of a username.

```sql
CREATE FUNCTION string_length(s text)
RETURNS NULL ON NULL INPUT
RETURNS int
LANGUAGE java
AS 'return s.length();';

SELECT username, string_length(username) AS name_length 
FROM users;
```

### Warning

UDFs can impact performance and are best used in local or test environments. Avoid in production unless necessary.

## JSON Support – JSON Input/Output

### What Is JSON Support?

Cassandra allows inserting and retrieving data in JSON format, simplifying integration with modern applications.

### Example: Inserting a Post

In our social media app, insert a post using JSON.

```sql
INSERT INTO posts JSON 
'{"user_id": 2, "post_id": "uuid()", "content": "Loving Cassandra!", "created_at": "2025-08-05T12:00:00"}';
```

### Example: Retrieving as JSON

Fetch a user’s data as JSON.

```sql
SELECT JSON user_id, username, email FROM users WHERE user_id = 1;
```

### Best Practices

- Use JSON for quick prototyping or when integrating with JSON-based APIs.
- Avoid for high-performance queries, as JSON parsing adds overhead.

## Best Practices and Common Pitfalls

### Best Practices

- **Batch Wisely** 📦: Use unlogged batches for performance and limit to same-partition operations.
- **Use LWT Sparingly** 🔒: Reserve for scenarios requiring concurrency control or uniqueness.
- **Avoid Overusing Materialized Views** 📊: Consider denormalization or external search engines for complex queries.
- **Leverage UDTs for Structure** 🛠️: Use UDTs to model complex but related data.
- **Test UDFs Locally** 🧪: Deploy UDFs cautiously due to performance implications.
- **Validate JSON Data** 📜: Ensure JSON inputs match the table schema to avoid errors.

### Common Pitfalls

- **Overloading with Batches** ⚠️: Avoid batches across multiple partitions.
- **Overusing LWT** 🚨: Don’t use LWT for simple updates, as it’s resource-intensive.
- **Ignoring Materialized View Limitations** 🚫: Be aware of consistency issues in newer Cassandra versions.
- **Complex UDFs in Production** 🛑: UDFs can degrade performance; test thoroughly.

## Conclusion

Today, we explored advanced CQL features that make Apache Cassandra a powerful tool for building scalable, real-time applications. By understanding `BATCH` statements, Lightweight Transactions, Materialized Views, UDTs, UDFs, and JSON support, you’re equipped to handle complex use cases in your social media app or any other Cassandra-powered project. Always balance performance and consistency, and avoid common pitfalls to keep your cluster running smoothly. 🚀