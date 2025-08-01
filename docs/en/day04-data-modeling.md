# Day 04 - Cassandra Data Modeling Guide — Step-by-Step with Practical Examples 🛠️
![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white) 
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)


Welcome to **Day 4** of our Apache Cassandra learning journey! 🎉 In this guide, we dive deep into **data modeling**, a critical skill for building efficient, scalable applications with Cassandra. This is the fourth part of a four-day series:

- **Day 1**: Introduction to Cassandra ([day01-introduction.md](day01-introduction.md)) 📄
- **Day 2**: Installing Cassandra ([day02-installation.md](day02-installation.md)) 🖥️
- **Day 3**: Getting Started with CQL ([day03-cql-basics.md](day03-cql-basics.md)) 💻
- **Day 4**: Data Modeling Basics (this guide) 📊

By now, you’ve learned Cassandra’s basics, set up a cluster, and explored CQL. Today, we’ll master **query-driven data modeling**, creating tables that power fast, scalable applications. Let’s make data modeling fun and practical with real-world examples! 🚀

> **Note**: To explore your Cassandra setup, use these CQL commands:
> - `DESCRIBE KEYSPACES;` — Lists all keyspaces in your cluster.
> - `DESCRIBE TABLES;` — Lists all tables in the current keyspace.
> Run these in your CQL shell to get familiar with your environment! 🔍

---

## 📖 Introduction to Cassandra Data Modeling

Apache Cassandra is a distributed NoSQL database built for **high scalability** and **availability**. Unlike SQL databases, which rely on normalization and JOINs, Cassandra uses a **query-driven design**. This means you design tables based on your application’s queries, often duplicating data to ensure fast reads. 🏎️

In this guide, we’ll:
- Explain Cassandra’s core concepts (primary keys, denormalization, and more) 📝
- Walk through a step-by-step data modeling process 🗺️
- Build exciting examples: a **social media app** 📱 and an **online bookstore** 📚
- Share best practices and pitfalls to avoid ⚠️
- Provide exercises to test your skills 🧠

Let’s get modeling! 💪

---

## 🔑 Core Concepts of Cassandra Data Modeling

To design effective Cassandra tables, you need to understand these foundational concepts:

### 1. Primary Key 🗝️
The **primary key** uniquely identifies rows and consists of:
- **Partition Key**: Determines which node in the cluster stores the data. It’s hashed to distribute data evenly. For example, a user ID ensures data for each user is stored together. 📍
- **Clustering Columns** (optional): Sort data within a partition. For example, sorting posts by timestamp within a user’s partition. 📅

Example:
```sql
PRIMARY KEY ((partition_key), clustering_column1, clustering_column2)
```
- `partition_key` decides the node.
- `clustering_column1` and `clustering_column2` sort rows within the partition.

### 2. Denormalization 📚
In SQL, you normalize data to avoid duplication. In Cassandra, you **denormalize**, creating multiple tables tailored to specific queries. This means duplicating data to avoid JOINs. It’s like having pre-organized bookshelves for each reader’s favorite genres! 📚

### 3. No JOINs or Aggregations 🚫
Cassandra doesn’t support JOINs, GROUP BY, or on-the-fly aggregations (e.g., SUM, COUNT). All data for a query must be in one table. For aggregations, use **counter columns** or pre-compute results. Think of it as pre-baking your query results! 🍰

### 4. Data Distribution 🌐
The partition key hashes data across nodes. A good partition key (e.g., user ID or date) ensures even distribution, avoiding “hot spots” where one node gets overloaded. Imagine spreading books evenly across library branches! 🏛️

---

## 📋 Cassandra Table Syntax

Here’s the syntax for creating a table in Cassandra:
```sql
CREATE TABLE keyspace_name.table_name (
    column1 data_type,
    column2 data_type,
    PRIMARY KEY ((partition_key), clustering_column1)
) WITH CLUSTERING ORDER BY (clustering_column1 DESC);
```

- **Keyspace**: A namespace for tables, like a database in SQL.
- **Partition Key**: Enclosed in parentheses for data distribution.
- **Clustering Columns**: Sort data within a partition.
- **CLUSTERING ORDER BY**: Sets ascending (`ASC`) or descending (`DESC`) order.

Example:
```sql
CREATE TABLE bookstore.books_by_genre (
    genre text,
    published_date timestamp,
    book_id uuid,
    title text,
    author text,
    PRIMARY KEY (genre, published_date)
) WITH CLUSTERING ORDER BY (published_date DESC);
```
- **Partition Key**: `genre` (groups books by genre, e.g., “Fantasy”).
- **Clustering Column**: `published_date` (sorts books newest first within a genre).

Query:
```sql
SELECT * FROM books_by_genre WHERE genre = 'Fantasy';
```

---

## 🗺️ Step-by-Step Data Modeling Process

To create a Cassandra data model:
1. **Identify Queries** 📋: List all queries your app needs (e.g., “Get all books by genre”).
2. **Define Entities** 🧩: Identify entities (e.g., books, users) and their relationships.
3. **Design Tables** 🏗️: Create a table for each query, including all needed data.
4. **Choose Keys** 🔑: Select partition and clustering keys for distribution and sorting.
5. **Validate** ✅: Test with realistic data to ensure performance.

Let’s apply this to two fun examples! 🎉

---

## 📱 Practical Example 1: Social Media App

Imagine building **ConnectSphere**, a social media app where users post photos, comment, and like content. Let’s model its data.

### Entities
- **Users**: ID, username, email, join date.
- **Posts**: ID, user ID, caption, image URL, creation time.
- **Comments**: ID, post ID, user ID, text, comment time.
- **Likes**: User ID, post ID, like time.

### Query Requirements
1. Fetch user profile by ID. 🧑
2. Get a user’s posts, sorted by time (newest first). 📸
3. Get a post by ID. 📷
4. Get comments on a post, sorted by time (oldest first). 💬
5. Get posts a user liked. ❤️
6. Get users who liked a post. 👍

### Tables

#### 1. Users
```sql
CREATE TABLE connectsphere.users (
    user_id uuid PRIMARY KEY,
    username text,
    email text,
    joined_at timestamp
);
```
- **Purpose**: Fetch user details (e.g., “Show @TravelLover’s profile”).
- **Query**: `SELECT * FROM users WHERE user_id = ?;`

#### 2. Posts by User
```sql
CREATE TABLE connectsphere.posts_by_user (
    user_id uuid,
    created_at timestamp,
    post_id uuid,
    caption text,
    image_url text,
    PRIMARY KEY ((user_id), created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);
```
- **Purpose**: Show a user’s posts, like @TravelLover’s photo feed, newest first.
- **Query**: `SELECT * FROM posts_by_user WHERE user_id = ?;`

#### 3. Post by ID
```sql
CREATE TABLE connectsphere.post_by_id (
    post_id uuid PRIMARY KEY,
    user_id uuid,
    caption text,
    image_url text,
    created_at timestamp
);
```
- **Purpose**: Fetch a specific post, like a viral sunset photo. 🌅
- **Query**: `SELECT * FROM post_by_id WHERE post_id = ?;`

#### 4. Comments by Post
```sql
CREATE TABLE connectsphere.comments_by_post (
    post_id uuid,
    comment_time timestamp,
    comment_id uuid,
    user_id uuid,
    content text,
    PRIMARY KEY ((post_id), comment_time)
) WITH CLUSTERING ORDER BY (comment_time ASC);
```
- **Purpose**: Show comments on a post, like reactions to a travel photo.
- **Query**: `SELECT * FROM comments_by_post WHERE post_id = ?;`

#### 5. Likes by User
```sql
CREATE TABLE connectsphere.likes_by_user (
    user_id uuid,
    liked_at timestamp,
    post_id uuid,
    PRIMARY KEY ((user_id), liked_at)
) WITH CLUSTERING ORDER BY (liked_at DESC);
```
- **Purpose**: Show posts a user liked, like @TravelLover’s favorite photos.
- **Query**: `SELECT * FROM likes_by_user WHERE user_id = ?;`

#### 6. Likes by Post
```sql
CREATE TABLE connectsphere.likes_by_post (
    post_id uuid,
    liked_at timestamp,
    user_id uuid,
    PRIMARY KEY ((post_id), liked_at)
) WITH CLUSTERING ORDER BY (liked_at DESC);
```
- **Purpose**: Show who liked a post, like fans of a sunset photo. 🌄
- **Query**: `SELECT * FROM likes_by_post WHERE post_id = ?;`

### Example Queries
```sql
-- Fetch @TravelLover’s profile
SELECT * FROM users WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- Get @TravelLover’s posts
SELECT * FROM posts_by_user WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- Get a specific sunset photo
SELECT * FROM post_by_id WHERE post_id = 660e8400-e29b-41d4-a716-446655440000;

-- Get comments on a post
SELECT * FROM comments_by_post WHERE post_id = 660e8400-e29b-41d4-a716-446655440000;

-- Get posts @TravelLover liked
SELECT * FROM likes_by_user WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- Get users who liked a post
SELECT * FROM likes_by_post WHERE post_id = 660e8400-e29b-41d4-a716-446655440000;
```

---

## 📚 Practical Example 2: Online Bookstore

Let’s build **BookHaven**, an online bookstore where users buy books, and admins track sales. 📖

### Entities
- **Users**: ID, name, email, phone, join date.
- **Books**: ID, title, author, genre, published date.
- **Orders**: ID, user ID, order time, total price, books, status.
- **Monthly Sales**: Total sales, order count by month.

### Query Requirements
1. Fetch user by ID. 🧑
2. Get books by genre, sorted by publication date. 📚
3. Get orders by user, sorted by time. 📦
4. Get orders by day. 📅
5. Get monthly sales summary. 💰

### Tables

#### 1. Users
```sql
CREATE TABLE bookhaven.users (
    user_id uuid PRIMARY KEY,
    name text,
    email text,
    phone text,
    joined_at timestamp
);
```
- **Purpose**: Fetch user details, like “Who’s Alice Bookworm?”.
- **Query**: `SELECT * FROM users WHERE user_id = ?;`

#### 2. Books by Genre
```sql
CREATE TABLE bookhaven.books_by_genre (
    genre text,
    published_date timestamp,
    book_id uuid,
    title text,
    author text,
    price decimal,
    PRIMARY KEY ((genre), published_date)
) WITH CLUSTERING ORDER BY (published_date DESC);
```
- **Purpose**: Browse books by genre, like “Latest Fantasy novels”.
- **Query**: `SELECT * FROM books_by_genre WHERE genre = 'Fantasy';`

#### 3. Orders by User
```sql
CREATE TABLE bookhaven.orders_by_user (
    user_id uuid,
    order_time timestamp,
    order_id uuid,
    total_price decimal,
    books list<text>,
    status text,
    PRIMARY KEY ((user_id), order_time)
) WITH CLUSTERING ORDER BY (order_time DESC);
```
- **Purpose**: Show a user’s order history, like Alice’s book purchases.
- **Query**: `SELECT * FROM orders_by_user WHERE user_id = ?;`

#### 4. Orders by Day
```sql
CREATE TABLE bookhaven.orders_by_day (
    order_date date,
    order_time timestamp,
    order_id uuid,
    user_id uuid,
    total_price decimal,
    books list<text>,
    status text,
    PRIMARY KEY ((order_date), order_time)
) WITH CLUSTERING ORDER BY (order_time DESC);
```
- **Purpose**: View all orders on a specific day, like Black Friday sales.
- **Query**: `SELECT * FROM orders_by_day WHERE order_date = '2025-08-01';`

#### 5. Monthly Sales Summary
```sql
CREATE TABLE bookhaven.monthly_sales_summary (
    month text PRIMARY KEY,
    total_sales counter,
    order_count counter
);
```
- **Purpose**: Track monthly sales, like August’s revenue.
- **Query**: `SELECT * FROM monthly_sales_summary WHERE month = '2025-08';`

### Inserts
```sql
-- Add user Alice Bookworm
INSERT INTO users (user_id, name, email, phone, joined_at) VALUES (
    550e8400-e29b-41d4-a716-446655440000,
    'Alice Bookworm',
    'alice@example.com',
    '09123456789',
    toTimestamp(now())
);

-- Add a Fantasy book
INSERT INTO books_by_genre (
    genre, published_date, book_id, title, author, price
) VALUES (
    'Fantasy',
    '2025-01-01 00:00:00',
    770e8400-e29b-41d4-a716-446655440000,
    'Dragon’s Quest',
    'J.R.R. Tolkien',
    29.99
);

-- Add Alice’s order
INSERT INTO orders_by_user (
    user_id, order_time, order_id, total_price, books, status
) VALUES (
    550e8400-e29b-41d4-a716-446655440000,
    toTimestamp(now()),
    660e8400-e29b-41d4-a716-446655440000,
    29.99,
    ['Dragon’s Quest'],
    'delivered'
);

-- Add order to daily table
INSERT INTO orders_by_day (
    order_date, order_time, order_id, user_id, total_price, books, status
) VALUES (
    '2025-08-01',
    toTimestamp(now()),
    660e8400-e29b-41d4-a716-446655440000,
    550e8400-e29b-41d4-a716-446655440000,
    29.99,
    ['Dragon’s Quest'],
    'delivered'
);

-- Update sales summary
UPDATE monthly_sales_summary SET 
    total_sales = total_sales + 29.99,
    order_count = order_count + 1 
WHERE month = '2025-08';
```

---

## ✅ Best Practices

- **Query-First Design** 📋: Build tables for specific queries.
- **Balanced Partitions** ⚖️: Choose keys to distribute data evenly.
- **Small Partitions** 📏: Keep partitions under thousands of rows.
- **Smart Denormalization** 📚: Duplicate data strategically.
- **Pre-Aggregate** 📊: Use counters or tables for aggregations.
- **Test Thoroughly** 🧪: Validate with real-world data.

---

## ⚠️ Common Pitfalls

- **SQL Mindset** 🚫: Don’t normalize or expect JOINs.
- **Hot Spots** 🔥: Avoid keys that overload nodes.
- **Large Partitions** 📈: Use composite keys to split data.
- **Overusing Indexes** 📉: Prefer tables over secondary indexes.
- **Frequent Updates** 🔄: Design for append-heavy workloads.

---

## 🧠 Exercises

1. **Books by Author** 📖
   - Query: `SELECT * FROM books_by_author WHERE author = ?;`
   - Hint: Use `author` as partition key, `published_date` as clustering column.

2. **User Reviews** ⭐
   - Queries: Get reviews by book or by user.
   - Hint: Create `reviews_by_book` and `reviews_by_user` tables.

3. **Order Status Updates** 📦
   - Query: Track order status changes by order ID.
   - Hint: Use `order_id` as partition key, `update_time` as clustering column.
---
