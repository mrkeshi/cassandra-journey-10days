# Day 07 - Query-Driven Data Modeling in Apache Cassandra

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)

Welcome to **Day 7** of our Apache Cassandra learning journey! 🎉 Today, we dive deep into **Query-Driven Data Modeling**, a core principle that sets Cassandra apart from traditional relational databases. Unlike RDBMS, where you normalize data and design tables based on relationships, Cassandra’s design revolves around the queries you plan to run. This approach ensures blazing-fast read performance, but it requires careful planning. 🚀

In this guide, we’ll:
- Explain the principles of query-driven data modeling 📝
- Explore how to design tables based on your application’s queries 🛠️
- Provide practical, non-repetitive examples tailored to a social media application 📱
- Highlight best practices and pitfalls to avoid ⚠️
- Include exercises to test your skills 🧠
- Tie this to Day 6’s concepts (TTL, Secondary Indexes, Built-in Functions) for continuity 🔗

Let’s make it engaging, practical, and beginner-friendly! 💪

---

## 📖 Introduction to Query-Driven Data Modeling

In Apache Cassandra, **data modeling** is driven by the queries your application needs to perform. Unlike relational databases, where you design tables to avoid redundancy and rely on JOINs, Cassandra prioritizes **read performance** by denormalizing data and tailoring tables to specific query patterns. This means you start by defining your queries, then design your schema to make those queries as efficient as possible.

### 🔑 Key Principles
1. **Denormalization**:
   - Cassandra doesn’t support JOINs, so you often duplicate data across multiple tables to support different query patterns.
   - Goal: Optimize for **read speed**, even if it means more storage or write complexity.

2. **Design for Reads, Not Writes**:
   - Cassandra is optimized for fast reads. Your table structure should minimize disk seeks and ensure data is pre-organized for your queries.
   - Writes are cheap in Cassandra, so it’s okay to write the same data to multiple tables.

3. **Query First, Then Schema**:
   - Identify your application’s queries first (e.g., “Get all posts by a user” or “Find posts by hashtag”).
   - Design tables to make those queries efficient, using appropriate **Partition Keys** and **Clustering Keys**.

4. **Composite Keys**:
   - Use **Partition Keys** to distribute data across nodes.
   - Use **Clustering Keys** to sort data within a partition for efficient retrieval.

5. **TTL Integration** (from Day 6):
   - Use TTL for temporary data (e.g., expiring posts or notifications) to manage storage efficiently.
   - Example: Stories in a social media app that expire after 24 hours.

---

## 🛠️ Section 1: Understanding Query-Driven Design

Let’s break down the process of query-driven data modeling with a real-world scenario: a **social media platform**. Users can post content, follow others, and browse posts by hashtags or categories. Our goal is to design tables that support the most common queries efficiently.

### 📌 Step-by-Step Process
1. **Identify Queries**:
   - What data does your application need to retrieve?
   - Example: “Show all posts by a specific user, sorted by time” or “Show posts with a specific hashtag.”

2. **Design the Primary Key**:
   - Choose a **Partition Key** to distribute data (e.g., `user_id` for user-specific queries).
   - Add **Clustering Keys** to sort data within a partition (e.g., `created_at` for time-based ordering).

3. **Denormalize for Query Patterns**:
   - Create separate tables for each major query pattern, even if it means duplicating data.
   - Example: One table for posts by user, another for posts by hashtag.

4. **Optimize with TTL and Indexes** (Day 6 Connection):
   - Use TTL for ephemeral data (e.g., stories or notifications).
   - Use secondary indexes sparingly for queries that don’t fit the primary key structure (but avoid overusing them due to performance costs).

---

## 📚 Section 2: Practical Example – Social Media Platform

Let’s design tables for a social media app with the following queries:
1. **Get all posts by a specific user**, sorted by time (newest first).
2. **Get all posts with a specific hashtag** (e.g., #AI).
3. **Get posts from users someone follows** (a user’s feed).
4. **Get notifications for a user**, which expire after 7 days.

### 🎯 Query 1: Posts by User
**Query**:
```sql
SELECT * FROM posts_by_user WHERE user_id = 'ali123' ORDER BY created_at DESC;
```

**Table Design**:
```cql
CREATE TABLE posts_by_user (
    user_id uuid,
    post_id uuid,
    title text,
    content text,
    created_at timeuuid,
    PRIMARY KEY (user_id, created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);
```
- **Partition Key**: `user_id` (distributes posts by user across nodes).
- **Clustering Key**: `created_at` (sorts posts by time, newest first).
- **Why TimeUUID?** Ensures uniqueness for posts created at the same time (Day 6: Built-in Functions).

**Insert Example**:
```cql
INSERT INTO posts_by_user (user_id, post_id, title, content, created_at)
VALUES (123e4567-e89b-12d3-a456-426614174000, uuid(), 'My first post!', 'Hello world!', now());
```

**Retrieve Example**:
```cql
SELECT * FROM posts_by_user WHERE user_id = 123e4567-e89b-12d3-a456-426614174000;
```

### 🎯 Query 2: Posts by Hashtag
**Query**:
```sql
SELECT * FROM posts_by_hashtag WHERE hashtag = '#AI' ORDER BY created_at DESC;
```

**Table Design**:
```cql
CREATE TABLE posts_by_hashtag (
    hashtag text,
    created_at timeuuid,
    post_id uuid,
    user_id uuid,
    title text,
    content text,
    PRIMARY KEY (hashtag, created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);
```
- **Partition Key**: `hashtag` (groups posts by hashtag).
- **Clustering Key**: `created_at` (sorts posts chronologically).
- **Note**: Same post data is duplicated here to avoid JOINs.

**Insert Example** (same post as above, duplicated for hashtag):
```cql
INSERT INTO posts_by_hashtag (hashtag, created_at, post_id, user_id, title, content)
VALUES ('#AI', now(), uuid(), 123e4567-e89b-12d3-a456-426614174000, 'My first post!', 'Hello world!');
```

### 🎯 Query 3: User’s Feed (Posts from Followed Users)
**Query**:
```sql
SELECT * FROM user_feed WHERE user_id = 'ali123' ORDER BY created_at DESC;
```

**Table Design**:
```cql
CREATE TABLE user_feed (
    user_id uuid,
    created_at timeuuid,
    post_id uuid,
    poster_id uuid,
    title text,
    content text,
    PRIMARY KEY (user_id, created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);
```
- **Partition Key**: `user_id` (the user viewing their feed).
- **Clustering Key**: `created_at` (sorts posts by time).
- **Logic**: When a user posts, insert the post into the `user_feed` table for all their followers.

**Insert Example** (when a user posts, fan-out to followers):
```cql
INSERT INTO user_feed (user_id, created_at, post_id, poster_id, title, content)
VALUES (follower_uuid, now(), uuid(), 123e4567-e89b-12d3-a456-426614174000, 'My first post!', 'Hello world!');
```

### 🎯 Query 4: Notifications with TTL
**Query**:
```sql
SELECT * FROM notifications WHERE user_id = 'ali123';
```

**Table Design** (with TTL from Day 6):
```cql
CREATE TABLE notifications (
    user_id uuid,
    notification_id uuid,
    created_at timeuuid,
    message text,
    PRIMARY KEY (user_id, created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);
```
- **Partition Key**: `user_id`.
- **Clustering Key**: `created_at`.
- **TTL**: Notifications expire after 7 days (604800 seconds).

**Insert Example** (with TTL):
```cql
INSERT INTO notifications (user_id, notification_id, created_at, message)
VALUES (123e4567-e89b-12d3-a456-426614174000, uuid(), now(), 'You have a new follower!')
USING TTL 604800;
```

**Retrieve with TTL Check** (Day 6: Built-in Functions):
```cql
SELECT message, TTL(message) AS remaining_time FROM notifications WHERE user_id = 123e4567-e89b-12d3-a456-426614174000;
```

---

## ⚠️ Best Practices and Pitfalls

### ✅ Do’s
- **Start with Queries**: Always define your application’s queries before designing tables.
- **Use TimeUUID for Time-Based Keys**: Prevents conflicts when multiple records share the same timestamp (Day 6).
- **Leverage TTL for Temporary Data**: Perfect for notifications, stories, or session tokens.
- **Keep Partitions Manageable**: Avoid “hot partitions” by choosing partition keys that distribute data evenly (e.g., avoid using a single hashtag like `#trending` for all posts).

### ❌ Don’ts
- **Avoid Overusing Secondary Indexes** (Day 6): They’re slow for high-cardinality columns (e.g., `user_id`). Use dedicated tables instead.
- **Don’t Expect JOINs**: Duplicate data across tables to support different query patterns.
- **Don’t Ignore Partition Size**: Large partitions (e.g., millions of rows for one `user_id`) can degrade performance. Consider bucketing (e.g., `user_id, year_month` as partition key).

---

## 🧠 Exercises to Test Your Skills

1. **Design a Table**:
   - Query: “Get all comments on a specific post, sorted by time.”
   - Task: Write the `CREATE TABLE` statement and an example `INSERT` and `SELECT` query.

2. **Extend the Social Media App**:
   - Query: “Get all posts liked by a specific user.”
   - Task: Design a table, explain your primary key choice, and write an `INSERT` query.

3. **TTL Challenge**:
   - Scenario: Stories in the social media app expire after 24 hours.
   - Task: Modify the `posts_by_user` table to include a `story` boolean field and apply a 24-hour TTL only to stories.

4. **Advanced**:
   - Query: “Get the top 10 most recent posts in a category (e.g., ‘sports’) from the last 7 days.”
   - Task: Design a table and explain how you’d handle time-based filtering with `minTimeuuid` (Day 6).

Feel free to share your answers, and I’ll review them! 😊 Or let me know if you want hints or solutions.

---

## 🔗 Connecting to Day 6: TTL, Secondary Indexes, and Built-in Functions

- **TTL**: Used in the `notifications` table to auto-expire data after 7 days, reducing storage overhead.
- **Secondary Indexes**: If you need to query posts by a rare field (e.g., `content` contains a keyword), you could add a secondary index, but it’s better to create a new table for frequent queries.
- **Built-in Functions**:
   - Used `now()` and `uuid()` for generating unique IDs and timestamps.
   - Used `minTimeuuid` and `maxTimeuuid` for time-range queries (e.g., posts from a specific date range).

---

## ❓ FAQs

**Q: How do I know if I need multiple tables for the same data?**  
A: If your application has multiple query patterns (e.g., by user, by hashtag, by category), create a table for each pattern. Cassandra’s denormalization expects this.

**Q: Can I use secondary indexes instead of multiple tables?**  
A: Secondary indexes (Day 6) are useful for low-cardinality fields or infrequent queries, but they’re less efficient than dedicated tables for frequent or high-cardinality queries.

**Q: What happens if two posts have the same `created_at` timestamp?**  
A: Use `TimeUUID` instead of `timestamp` to ensure uniqueness, as it includes a unique identifier (Day 6).

---

## 🚀 What’s Next?

Tomorrow (Day 8), we’ll explore **Cassandra’s Advanced Features** like materialized views, batch operations, and lightweight transactions. These will build on today’s query-driven modeling to make your applications even more powerful! Stay tuned! 🌟

If you have questions, want feedback on your exercise answers, or need more examples, let me know! 😊

---
