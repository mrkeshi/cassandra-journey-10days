# Day 09 - Mini Project: Building a Simple Library App with Apache Cassandra

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)

Welcome to **Day 9** of our Apache Cassandra learning journey! 🎉 Today, we embark on a **practical mini-project** to build a simple backend application for managing a digital library using Cassandra and CQL. This project follows a **Query-Driven** approach and leverages advanced CQL features like collections (SET, LIST, MAP), TTL, and secondary indexes. The goal is to apply your knowledge in a real-world scenario and prepare for future extensions like adding an API or UI. 🚀

In this guide, we will:

- Design tables based on query requirements 📝
- Use SET, LIST, and MAP collections for complex data modeling 📚
- Implement TTL for automatic deletion of expired data 🕒
- Create secondary indexes for efficient searching 🔍
- Explore practical queries and data analysis 📊
- Provide exercises to solidify your skills 💪

---

## 🎯 Project Goal

Build a simple backend application for managing a digital library, focusing on Cassandra’s query-driven design principles and advanced CQL features. This project will help you:

- Design tables tailored to application queries.
- Utilize collections and TTL for efficient data management.
- Implement secondary indexes for enhanced search capabilities.
- Prepare for scalability and future enhancements.

## 📖 Project Scenario

Imagine a digital library with the following features:

- **User Registration**: Store user details including name, email, registration date, and preferences (as a MAP).
- **Book Catalog**: Store book details including title, author, categories (SET), publication year, and availability status.
- **Borrowing**: Allow users to borrow multiple books and track borrowing history.
- **History**: Maintain borrowing history for users and books.
- **Reporting**: Generate reports on active books, unreturned books, and active users.
- **Automation**: Automatically delete expired data (e.g., old borrowing records) using TTL.

---

## 🧱 Step 1: Database Design (Query-Driven)

In Cassandra, tables are designed based on the queries the application will run, not traditional normalization. Below are the tables designed to meet the project’s query requirements.

### 🗃️ Table 1: users – User Registration

This table stores user information, using a MAP to store preferences for book categories.

```sql
CREATE TABLE library.users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    registration_date TIMESTAMP,
    preferences MAP<TEXT, INT>  -- Preferences for categories with scores
);
```

### 📚 Table 2: books – Book Catalog

This table stores book information, using a SET for categories.

```sql
CREATE TABLE library.books (
    book_id UUID PRIMARY KEY,
    title TEXT,
    author TEXT,
    categories SET<TEXT>,
    published_year INT,
    available BOOLEAN
);
```

### 📦 Table 3: borrows_by_user – User Borrowing History

This table tracks each user’s borrowing history, using TTL to automatically delete records after 30 days.

```sql
CREATE TABLE library.borrows_by_user (
    user_id UUID,
    borrow_id TIMEUUID,
    book_id UUID,
    borrow_date TIMESTAMP,
    return_date TIMESTAMP,
    notes TEXT,
    PRIMARY KEY (user_id, borrow_id)
) WITH CLUSTERING ORDER BY (borrow_id DESC)
  AND default_time_to_live = 2592000;  -- Auto-delete after 30 days
```

### 📦 Table 4: borrows_by_book – Book Borrowing History

This table tracks borrowing history from the perspective of books.

```sql
CREATE TABLE library.borrows_by_book (
    book_id UUID,
    borrow_id TIMEUUID,
    user_id UUID,
    borrow_date TIMESTAMP,
    return_date TIMESTAMP,
    status TEXT,
    PRIMARY KEY (book_id, borrow_id)
) WITH CLUSTERING ORDER BY (borrow_id DESC);
```

### 🔍 Secondary Index: Search by Book Title

To support searching books by title, we create a secondary index.

```sql
CREATE INDEX books_title_idx ON library.books (title);
```

---

## 🧪 Step 2: Insert Initial Data

To test the system, we insert initial data for users and books.

### ➕ Insert Users

```sql
INSERT INTO library.users (user_id, name, email, registration_date, preferences)
VALUES (uuid(), 'Ali Rezaei', 'ali@example.com', toTimestamp(now()), {'Fantasy': 5, 'Science': 3});

INSERT INTO library.users (user_id, name, email, registration_date, preferences)
VALUES (uuid(), 'Sara Ahmadi', 'sara@example.com', toTimestamp(now()), {'Classic': 4, 'History': 2});
```

### ➕ Insert Books

```sql
INSERT INTO library.books (book_id, title, author, categories, published_year, available)
VALUES (uuid(), '1984', 'George Orwell', {'Dystopian', 'Classic'}, 1949, true);

INSERT INTO library.books (book_id, title, author, categories, published_year, available)
VALUES (uuid(), 'The Hobbit', 'J.R.R. Tolkien', {'Fantasy'}, 1937, true);

INSERT INTO library.books (book_id, title, author, categories, published_year, available)
VALUES (uuid(), 'Sapiens', 'Yuval Noah Harari', {'History', 'Non-fiction'}, 2011, true);
```

---

## 📥 Step 3: Record a Book Borrowing

To record a borrowing, we need to update both `borrows_by_user` and `borrows_by_book` tables and update the book’s availability status.

### ⛏️ Steps:

1. Obtain `user_id` and `book_id`.
2. Generate `borrow_id` using `now()` (TIMEUUID type).
3. Insert into `borrows_by_user`.
4. Insert into `borrows_by_book`.
5. Update the book’s availability in the `books` table.

### ✅ Borrowing Operations:

```sql
-- Insert into borrows_by_user
INSERT INTO library.borrows_by_user (user_id, borrow_id, book_id, borrow_date, return_date, notes)
VALUES (
  <user_id>,
  now(),
  <book_id>,
  toTimestamp(now()),
  null,
  'First borrowing'
);

-- Insert into borrows_by_book
INSERT INTO library.borrows_by_book (book_id, borrow_id, user_id, borrow_date, return_date, status)
VALUES (
  <book_id>,
  now(),
  <user_id>,
  toTimestamp(now()),
  null,
  'borrowed'
);

-- Update book availability
UPDATE library.books SET available = false WHERE book_id = <book_id>;
```

---

## 📤 Step 4: Return a Book

To record a book return, we update the borrowing tables and the book’s availability status.

### ✅ Return Operations:

```sql
-- Update borrows_by_user
UPDATE library.borrows_by_user 
SET return_date = toTimestamp(now()), notes = 'Book returned'
WHERE user_id = <user_id> AND borrow_id = <borrow_id>;

-- Update borrows_by_book
UPDATE library.borrows_by_book 
SET return_date = toTimestamp(now()), status = 'returned'
WHERE book_id = <book_id> AND borrow_id = <borrow_id>;

-- Update book availability
UPDATE library.books SET available = true WHERE book_id = <book_id>;
```

---

## 🔍 Step 5: Useful Queries

These queries are used to extract information and generate reports from the data.

### 🟢 1. List Available Books

```sql
SELECT * FROM library.books WHERE available = true ALLOW FILTERING;
```

> **Warning**: Using `ALLOW FILTERING` can reduce performance. For frequently used queries, consider using an index or a separate table.

### 🔎 2. Search Books by Title

```sql
SELECT * FROM library.books WHERE title = '1984';
```

### 👤 3. View a User’s Borrowing History

```sql
SELECT * FROM library.borrows_by_user WHERE user_id = <user_id>;
```

### 📚 4. View a Book’s Borrowing History

```sql
SELECT * FROM library.borrows_by_book WHERE book_id = <book_id>;
```

### 📊 5. Report Books Borrowed in the Last 7 Days

```sql
SELECT * FROM library.borrows_by_book 
WHERE borrow_date >= '2025-07-29T00:00:00' 
AND borrow_date <= toTimestamp(now()) 
ALLOW FILTERING;
```

---

## 🔄 Step 6: Automatic Cleanup with TTL

Using `default_time_to_live = 2592000` (30 days) in the `borrows_by_user` table ensures borrowing records are automatically deleted after 30 days.

### Benefits:

- **Keep Tables Lightweight**: Reduces storage of old data.
- **Automated Management**: No need for scripts to delete old records.
- **Efficient Resource Use**: Optimizes database memory usage.

---

## 📝 Exercises for Mastery

To strengthen your skills, complete these exercises:

### ✅ Exercise 1: Insert Data

Insert data for 5 different users and at least 10 books. Use varied categories in the SET and different scores in the MAP for preferences.

### ✅ Exercise 2: Record Borrowings

Have each user borrow at least 2 books. Ensure the book categories align with user preferences.

### ✅ Exercise 3: Search by Preferences

Write a query to find users interested in the "Fantasy" category. (Hint: Use `CONTAINS KEY` for the MAP in the `users` table.)

```sql
SELECT * FROM library.users WHERE preferences CONTAINS KEY 'Fantasy';
```

### ✅ Exercise 4: Report Recent Borrowings

Write a query to return all books borrowed in the last 7 days. (Example provided above.)

### ✅ Exercise 5: Python Script for Availability Check

Write a Python script to check book availability and display a warning if no copies are available.

#### Sample Python Script:

```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

# Connect to Cassandra
auth_provider = PlainTextAuthProvider(username='cassandra', password='cassandra')
cluster = Cluster(['127.0.0.1'], auth_provider=auth_provider)
session = cluster.connect('library')

# Check book availability
rows = session.execute("SELECT title, available FROM books")
for row in rows:
    if not row.available:
        print(f"Warning: Book '{row.title}' is not available!")

# Close connection
cluster.shutdown()
```

---

## 🛠️ Additional Tips and Best Practices

### Best Practices

- **Query-Driven Design** 📝: Always design tables based on application queries.
- **Use Collections** 🗂️: Leverage SET, LIST, and MAP for modeling complex but related data.
- **Manage TTL** 🕒: Use TTL for temporary data (e.g., borrowing records) to keep the cluster lightweight.
- **Cautious Use of Secondary Indexes** 🔍: Use indexes only for low-cardinality columns.
- **Test Queries** 🧪: Validate queries in a test environment before deploying to production.

### Common Pitfalls

- **Overusing ALLOW FILTERING** ⚠️: This can degrade performance. Redesign tables for frequently used queries.
- **Ignoring TTL** 🚫: Not using TTL for temporary data can bloat the cluster.
- **Misusing Secondary Indexes** 🚨: Avoid indexes on high-cardinality columns (e.g., email).

---

## 🚀 Conclusion

In this mini-project, we built a simple backend application for managing a digital library using Apache Cassandra. By applying query-driven design, collections (SET, LIST, MAP), TTL, and secondary indexes, we created a scalable and efficient system. This project provides a strong foundation for adding an API or UI in the future. Complete the suggested exercises to reinforce your CQL and Cassandra skills! 💪