# Day 05 - Mastering Cassandra Collections: List, Set, and Map with Practical Examples

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)

Welcome to **Day 5** of our Apache Cassandra learning journey! 🎉 Today, we dive into the exciting world of **Cassandra Collections**—List, Set, and Map. These powerful data types allow you to store and manage structured data within a single column, making your tables flexible and efficient. 🚀

By now, you’ve mastered Cassandra basics, installation, CQL queries, and data modeling. In this guide, we’ll explore how to use Collections to handle complex data, perform operations like INSERT, UPDATE, SELECT, and DELETE, and query them effectively. Let’s make Collections fun and practical with a real-world example! 💪

---

## 📖 Introduction to Cassandra Collections

Cassandra Collections (List, Set, and Map) allow you to store multiple values in a single column, reducing the need for multiple tables or complex joins. They’re perfect for scenarios where you need to group related data, like user emails, roles, or contact numbers. Here’s a quick overview:

- **List**: Stores an ordered collection of elements, allowing duplicates (e.g., `['email1', 'email2']`).
- **Set**: Stores an unordered collection of unique elements (e.g., `{'admin', 'editor'}`).
- **Map**: Stores key-value pairs, like a dictionary (e.g., `{'home': '123456', 'mobile': '09120000000'}`).

In this guide, we’ll:
- Create a sample `users` table with all three Collection types 📝
- Perform core operations: INSERT, SELECT, UPDATE, and DELETE 🛠️
- Explore advanced querying with Collections (e.g., accessing specific elements) 🔎
- Highlight best practices and limitations ⚠️
- Provide exercises to solidify your understanding 🧠

Let’s get started! 🚀

---

## 🔑 Creating a Sample Table with Collections

We’ll create a `users` table to store user information, including a List of emails, a Set of roles, and a Map of phone numbers. This table will serve as our playground for all operations.

### 📦 Table Definition

```sql
CREATE TABLE users (
  username TEXT PRIMARY KEY,
  emails LIST<TEXT>,
  roles SET<TEXT>,
  phones MAP<TEXT, TEXT>
);
```

- **username**: The primary key, uniquely identifying each user.
- **emails**: A List to store multiple email addresses (ordered, allows duplicates).
- **roles**: A Set to store user roles (unordered, no duplicates).
- **phones**: A Map to store phone numbers with labels (e.g., 'home', 'mobile').

> **Tip**: Collections are ideal for small, bounded datasets. Avoid storing thousands of elements in a Collection, as this can impact performance. 📉

---

## 🛠️ Core Operations on Collections

Let’s perform the four main operations (INSERT, SELECT, UPDATE, DELETE) on our `users` table, with practical examples and expected outputs.

### ✅ 1. INSERT — Adding a New User

Insert a user named 'javad' with sample data for emails, roles, and phones.

```sql
INSERT INTO users (username, emails, roles, phones)
VALUES (
  'javad',
  ['javad@mail.com', 'me@example.com'],
  {'admin', 'editor'},
  {'home': '021123456', 'mobile': '09120000000'}
);
```

**What’s Happening?**
- We’re adding a user with two emails, two roles, and two phone numbers.
- The List (`emails`) preserves order and allows duplicates.
- The Set (`roles`) ensures no duplicates.
- The Map (`phones`) associates keys (e.g., 'home') with values (e.g., '021123456').

---

### 🔍 2. SELECT — Querying Collection Data

Let’s read the data we inserted and explore ways to query specific parts of Collections.

#### 📥 2.1. Fetch All Data for a User

```sql
SELECT * FROM users WHERE username = 'javad';
```

**Output**:

```
 username | emails                               | phones                                         | roles
----------+--------------------------------------+------------------------------------------------+---------------------
 javad    | ['javad@mail.com', 'me@example.com'] | {'home': '021123456', 'mobile': '09120000000'} | {'admin', 'editor'}
```

#### 📥 2.2. Fetch the First Email from the List

Access the first email using index `[0]`:

```sql
SELECT emails[0] FROM users WHERE username = 'javad';
```

**Output**:

```
 emails[0]
------------
 javad@mail.com
```

> **Note**: Lists are zero-indexed, and you can access elements directly with `[index]`.

#### 📥 2.3. Fetch a Specific Phone Number from the Map

Access the 'home' phone number using the Map key:

```sql
SELECT phones['home'] FROM users WHERE username = 'javad';
```

**Output**:

```
 phones['home']
---------------
 021123456
```

#### 📥 2.4. Fetch All Keys from the Map

Use the `keys()` function to get all keys in the `phones` Map:

```sql
SELECT keys(phones) FROM users WHERE username = 'javad';
```

**Output**:

```
 keys(phones)
---------------
 ['home', 'mobile']
```

#### 📥 2.5. Fetch All Values from the Map

Use the `values()` function to get all values in the `phones` Map:

```sql
SELECT values(phones) FROM users WHERE username = 'javad';
```

**Output**:

```
 values(phones)
-------------------
 ['021123456', '09120000000']
```

#### 📥 2.6. Fetch the Entire Set

Since Sets are unordered, you can’t access elements by index. Instead, fetch the entire Set:

```sql
SELECT roles FROM users WHERE username = 'javad';
```

**Output**:

```
 roles
---------------
 {'admin', 'editor'}
```

> **Important**: Sets don’t support indexing (e.g., `roles[0]`) because they’re unordered. Use `CONTAINS` (covered later) for filtering.

---

### 🔄 3. UPDATE — Modifying Collections

Collections support flexible updates, such as appending, removing, or replacing elements.

#### ▶️ 3.1. Append to a List

Add a new email to the `emails` List:

```sql
UPDATE users SET emails = emails + ['new@mail.com'] WHERE username = 'javad';
```

**After Update** (fetch to verify):

```sql
SELECT emails FROM users WHERE username = 'javad';
```

**Output**:

```
 emails
--------------------------------------
 ['javad@mail.com', 'me@example.com', 'new@mail.com']
```

#### ▶️ 3.2. Add to a Set

Add a new role to the `roles` Set:

```sql
UPDATE users SET roles = roles + {'viewer'} WHERE username = 'javad';
```

**After Update**:

```sql
SELECT roles FROM users WHERE username = 'javad';
```

**Output**:

```
 roles
----------------------
 {'admin', 'editor', 'viewer'}
```

> **Note**: If 'viewer' already exists, Cassandra ignores it since Sets don’t allow duplicates.

#### ▶️ 3.3. Add or Update a Map Entry

Add a new phone number with the key 'work':

```sql
UPDATE users SET phones['work'] = '026123456' WHERE username = 'javad';
```

**After Update**:

```sql
SELECT phones FROM users WHERE username = 'javad';
```

**Output**:

```
 phones
------------------------------------------------
 {'home': '021123456', 'mobile': '09120000000', 'work': '026123456'}
```

---

### 🗑️ 4. DELETE — Removing from Collections

You can remove specific elements from Collections or overwrite them entirely.

#### 🗑 4.1. Remove an Element from a List

Delete the second email (index 1):

```sql
DELETE emails[1] FROM users WHERE username = 'javad';
```

**After Delete**:

```sql
SELECT emails FROM users WHERE username = 'javad';
```

**Output**:

```
 emails
-----------------------------
 ['javad@mail.com', 'new@mail.com']
```

#### 🗑 4.2. Remove an Element from a Set

Remove the 'editor' role:

```sql
UPDATE users SET roles = roles - {'editor'} WHERE username = 'javad';
```

**After Delete**:

```sql
SELECT roles FROM users WHERE username = 'javad';
```

**Output**:

```
 roles
---------------
 {'admin', 'viewer'}
```

#### 🗑 4.3. Remove a Key from a Map

Delete the 'mobile' phone number:

```sql
DELETE phones['mobile'] FROM users WHERE username = 'javad';
```

**After Delete**:

```sql
SELECT phones FROM users WHERE username = 'javad';
```

**Output**:

```
 phones
--------------------------------------
 {'home': '021123456', 'work': '026123456'}
```

---

### 🔁 5. Replacing an Entire Collection

You can overwrite a Collection entirely, replacing all its elements.

#### ⛔️ Replace the Entire List

Set a new List of emails:

```sql
UPDATE users SET emails = ['only@mail.com'] WHERE username = 'javad';
```

**After Replace**:

```sql
SELECT emails FROM users WHERE username = 'javad';
```

**Output**:

```
 emails
---------------
 ['only@mail.com']
```

> **Warning**: Replacing a Collection deletes all previous data in that column. Use with caution! ⚠️

---

### 🧪 6. Advanced Querying with CONTAINS

To filter rows based on Collection values, use the `CONTAINS` keyword. However, this requires an index on the Collection column.

#### Create an Index on the `roles` Set

```sql
CREATE INDEX ON users(roles);
```

> **Note**: Don’t worry about indexes yet—we’ll cover them in detail in Day 6! For now, know that an index enables `CONTAINS` queries.

#### Query Users with a Specific Role

Find users with the 'admin' role:

```sql
SELECT * FROM users WHERE roles CONTAINS 'admin';
```

**Output** (assuming only 'javad' has the 'admin' role):

```
 username | emails              | phones                                  | roles
----------+--------------------+-----------------------------------------+---------------
 javad    | ['only@mail.com'] | {'home': '021123456', 'work': '026123456'} | {'admin', 'viewer'}
```

#### Querying Maps with `CONTAINS KEY` or `CONTAINS`

- **CONTAINS KEY**: Filter by Map keys.
- **CONTAINS**: Filter by Map values (requires an index on values).

Example (requires index on `phones` values):

```sql
CREATE INDEX ON users(values(phones));
SELECT * FROM users WHERE phones CONTAINS '021123456';
```

**Output**:

```
 username | emails              | phones                                  | roles
----------+--------------------+-----------------------------------------+---------------
 javad    | ['only@mail.com'] | {'home': '021123456', 'work': '026123456'} | {'admin', 'viewer'}
```

---

## 📊 Summary of Collection Features

Here’s a quick comparison of List, Set, and Map:

| **Collection** | **Update Operations**         | **Filterable?**              | **Key Notes**                              |
|----------------|-------------------------------|------------------------------|--------------------------------------------|
| **List**       | Append, index-based, replace   | No (index access only)       | Ordered, allows duplicates                 |
| **Set**        | Add/remove, replace           | Yes (with `CONTAINS` + index)| Unordered, no duplicates                   |
| **Map**        | Add/remove by key, replace    | Yes (with `CONTAINS`/`KEY` + index) | Key-value pairs, like a dictionary |

**Best Practices**:
- Use Lists for ordered data where duplicates are okay (e.g., event logs).
- Use Sets for unique, unordered data (e.g., tags or roles).
- Use Maps for key-value data (e.g., settings or contact details).
- Keep Collections small (<100 elements) to avoid performance issues.
- Use indexes sparingly, as they increase write overhead.

**Limitations**:
- Collections can’t be part of the primary key.
- `CONTAINS` queries require indexes, which we’ll explore in Day 6.
- Large Collections can slow down reads and writes—design with care.

---

## 🧠 Exercises to Test Your Skills

1. **Create a Table**: Design a `products` table with a List of categories, a Set of tags, and a Map of attributes (e.g., color, size). Insert a sample product.
2. **Query Practice**: Write a query to fetch the second category from the `products` table’s List.
3. **Update Challenge**: Add a new tag to the `tags` Set and remove an attribute from the `attributes` Map.
4. **Advanced Query**: Create an index on the `tags` Set and query products with a specific tag using `CONTAINS`.

