# DAY 01 - 🚀 Getting Started with Apache Cassandra – Basics & Architecture

Welcome! This is a beginner-friendly guide to understanding **Apache Cassandra** — a powerful, distributed NoSQL database designed for handling massive amounts of data with high availability and scalability.

If you’ve ever wondered how platforms like **Netflix**, **Reddit**, or **Instagram** manage so much data with zero downtime, Cassandra is one of the answers.

---

## 🎯 Goal

The goal is to build a **distributed and scalable system** that can store a huge amount of structured data. Each row is indexed by a unique key, and every row can hold **an unlimited number of columns**.

---

## 🧠 What is Apache Cassandra?

**Apache Cassandra** is an open-source NoSQL database built for performance and reliability. It was originally created at **Facebook in 2007** to power their inbox search, and later became an Apache project.

Cassandra is special because it mixes the best ideas from:

- **Amazon Dynamo** (for distributed key-value storage)  
- **Google BigTable** (for its column-based data model)

### 🏗️ In simple terms:
Cassandra is like a giant, decentralized spreadsheet where:

- Each row has a unique key  
- Each row can have different columns  
- Data is automatically distributed across many machines  
- There is no single point of failure

---

## ⚙️ How Does Cassandra Work?

- **Decentralized**: All nodes are equal. There’s no master or leader.  
- **Scalable**: You can add more machines (nodes), and performance increases.  
- **Available**: Even if some machines go down, the system keeps running.  
- **Tunable Consistency**: You choose between fast but eventual consistency or slower but stronger consistency.

> Cassandra is classified as an **AP system** (Available and Partition-tolerant) in the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem).

---

## 🧰 Common Cassandra Terms

| Term        | Meaning                                                                 |
|-------------|-------------------------------------------------------------------------|
| **Node**    | A single machine (physical, cloud, or Docker) running Cassandra         |
| **Cluster** | A group of connected Cassandra nodes                                    |
| **Keyspace**| Like a database – contains multiple tables                              |
| **Table**   | Like a table in SQL – contains rows                                     |
| **Row**     | Identified by a primary key; holds multiple columns                     |
| **Column**  | A single piece of data (key-value pair)                                 |

---

## 📦 Use Cases for Cassandra

Cassandra shines in real-time, write-heavy, distributed applications. Some real-world examples:

- **Reddit/Digg**: Use it to store user data with high availability  
- **IoT**: Collect and analyze sensor data (temperature, motion, etc.)  
- **Logging**: Store time-series data for monitoring and alerts  
- **Social apps**: Messaging, news feeds, and recommendation engines

---

## 🔍 Example: Why Use Cassandra?

Let’s say you're building a system that:

- Needs to store billions of sensor readings per day  
- Can’t afford downtime  
- Requires fast writes  
- Doesn’t always need real-time strong consistency

Cassandra is a perfect fit.

---

## 🧱 High-Level Architecture Overview

- **Data is split across nodes** using consistent hashing  
- **No central coordinator** – each node can handle reads/writes  
- **Replication** makes sure data is safe and available, even if nodes fail

We’ll explore more architecture details like **Memtables**, **SSTables**, **compaction**, and **hinted handoff** in the coming lessons.

---

## ⚠️ Disclaimer

All content here is written for educational purposes and focuses on general concepts, not on any specific Cassandra version. The goal is to help you understand **how Cassandra works**, not just how to use it.

---

## 🧠 Tip

If you're new to distributed systems, try to imagine **Cassandra as a smart library**:

- The books (data) are stored in many branches (nodes)  
- Every branch can serve you  
- Even if one branch is closed, others will help you  
- Some books are copied across branches to keep things safe

---


## 🗝️ Primary Key

A **Primary Key** in Cassandra uniquely identifies each row and has two parts:

- **Partition Key**: Determines which node stores the data.
- **Clustering Key(s)**: Defines how data is sorted within the node.

**Example:**

```sql
PRIMARY KEY (city_id, employee_id)
```
- `city_id` → Partition Key
- `employee_id` → Clustering Key

All rows with the same `city_id` go to the same node; within that node, rows are sorted by `employee_id`.

---

## 🔄 Partitioner

A **Partitioner** decides where data is stored using a hash function.

- Default: `Murmur3`
- Hash output determines placement on the **Consistent Hash Ring**
- Once a partitioner is chosen, **it cannot be changed**

**Example:**
```sql
INSERT INTO employees (city_id, employee_id, name)
VALUES ('nyc', 101, 'Alice');
```
- `'nyc'` is hashed using Murmur3
- Hash points to Node 2 → data goes to Node 2

---

## 👤 Coordinator Node

- The node that receives a client query.
- Forwards queries to correct nodes (based on hash & replication).
- Any node can act as the **Coordinator**.

---

## 📦 Replication

Cassandra replicates data across multiple nodes.

### 🔸 Terms

- **Replication Factor (RF):** Number of copies of each row.
- **Replication Strategy:** How to choose replica nodes.

**Example:**

If `RF = 3` → data is stored on 3 different nodes.

### 🔹 SimpleStrategy (Single DC)

- First replica → hashed node
- Next replicas → next 2 nodes clockwise on the ring

### 🔹 NetworkTopologyStrategy (Multi DC)

- Define different RFs per datacenter
- Avoid putting replicas on the same **rack**

---

## Consistency Levels

**Consistency Level** = Minimum number of nodes that must respond for read/write to be "successful".

Cassandra has **tunable consistency**.

### 🔸 Write Consistency Levels

| Level         | Description                                          |
|---------------|------------------------------------------------------|
| ONE, TWO, THREE | 1, 2, or 3 replicas must acknowledge                |
| QUORUM        | Majority (e.g., RF = 3 → 2 nodes)                    |
| ALL           | All replicas must respond (strongest)               |
| ANY           | 1 node or hint is enough (weakest)                  |
| LOCAL_QUORUM  | Quorum in same datacenter                           |
| EACH_QUORUM   | Quorum in each datacenter                           |

**Example:** (RF = 3, Consistency = QUORUM):

- 2 of 3 nodes must confirm
- If 2 nodes respond → ✅ success
- If only 1 node responds → ❌ fail

---

## 📝 Hinted Handoff

When a replica is **down**, Cassandra can **store a hint** on another node to replay later.

**Example:** (RF = 3, 1 node down, Consistency = QUORUM):

- Coordinator writes successfully to 2 available nodes ✅  
- Stores a **hint** for the 3rd (down) node  
- When the 3rd node comes back online → coordinator **replays** the stored hint to sync data  

### ⚠️ Caveats

- Hints are stored for **3 hours** by default.  
- After 3 hours, if the node is still down, it may miss updates.  
- Cassandra can fix this during reads via **Read Repair** (more on that later).

---

## ❌ Avoid Using ‘ANY’ Consistency Level for Writes

- Writes succeed even if **all replicas are down**, by storing a hint.  
- But data is **not yet fully written** to any replica.  
- If the coordinator crashes before replaying hints → **data loss** can occur.

### 🔺 Use `ANY` **only** if you accept this risk.

---

## 📖 Read Consistency Levels in Cassandra

Consistency levels for **read** queries specify how many replicas must respond before returning data.

**Example:**:

If you have `RF = 3` and use consistency `QUORUM` for reads:

- Coordinator waits for at least **2 nodes** (majority) to respond.  
- If 2 nodes respond, data is returned to client.  
- If fewer than 2 respond → read fails.

---

## 🔐 Achieving Strong Consistency

Cassandra guarantees strong consistency if:
`  R + W > RF `

where:

- **R** = number of nodes that respond to a read  
- **W** = number of nodes that acknowledge a write  
- **RF** = replication factor

**Example:** 1: Strong consistency with RF=3

- Write consistency = QUORUM (W = 2)  
- Read consistency = QUORUM (R = 2)  
- Since 2 + 2 > 3 → reads always reflect latest writes.

**Example:** 2: Weak consistency scenario

- Write consistency = ONE (W = 1)  
- Read consistency = ONE (R = 1)  
- 1 + 1 = 2 ≤ 3 → reads might return stale data.

---

## 🕵️‍♂️ Snitch: Optimizing Node Proximity & Speed

The **Snitch** tells Cassandra about network topology:

- Maps which nodes are in which data centers and racks.  
- Routes requests to the **nearest and fastest node**.  
- Helps place replicas across different racks to avoid single points of failure.

---

## ⚡ How Cassandra Performs a Read

1. Coordinator selects the **fastest replica** and sends a full read request.  
2. Requests **digests** (checksums) from other replicas to verify data integrity.  
3. If digests don’t match, reads full data from all replicas to find the latest version.  
4. Returns latest data to the client.  
5. Initiates **Read Repair** asynchronously to update stale replicas.

---

## 🔧 Read Repair: Keeping Data Consistent

- Fixes out-of-sync replicas detected during reads.  
- Uses **timestamps** to determine the newest data.  
- Runs probabilistically (default 10% of reads) to avoid overhead.  
- Runs asynchronously so it doesn't block client queries.

### Example

Assume a replication factor (RF) of 3 with nodes: **Node A**, **Node B**, and **Node C**.

1. An update happens:
   - Data for key `user123` is successfully written to Node A and Node B.
   - Node C misses the update (e.g., it was down).

2. A client performs a read request with consistency level **QUORUM**:
   - The coordinator requests full data from Node A (fastest node).
   - It requests digests (checksums) from Node B and Node C.

3. The coordinator detects inconsistency:
   - Node C's digest does not match Node A and B’s digest because Node C has stale data.

4. Read Repair runs:
   - The coordinator fetches the latest data from Node A or Node B.
   - It sends the updated data to Node C to repair the inconsistency.
   - The client receives the most recent data.

### Notes
- Read Repair only happens probabilistically (e.g., 10% of reads) to avoid performance overhead.
- It runs asynchronously, so it doesn’t block the client.

---

## 🌐 Gossip Protocol: Cluster State Synchronization

- Nodes exchange state info every second with 1–3 random nodes.  
- Uses **generation numbers** to detect restarts.  
- Seed nodes bootstrap gossip for new nodes joining.  
- Ensures all nodes have up-to-date cluster info.

---

## ⏰ Failure Detection with Phi Accrual Failure Detector

- Detects node failures based on adaptive suspicion levels instead of simple alive/dead.  
- Reduces false positives caused by network delays.  
- Gradually isolates slow or failing nodes.

---

## ✍️ Anatomy of Cassandra’s Write Operation

1. Write appended to **commit log** on disk (durability).  
2. Data added to **MemTable** (in-memory sorted structure).  
3. When MemTable fills, it flushes to disk as an immutable **SSTable**.  
4. Periodic **compaction** merges SSTables for efficient reads and storage.

---

## 📂 Commit Log

- Write-ahead log stored on disk.  
- Ensures data durability in case of crashes.  
- Write isn't acknowledged until commit log write completes.

---

## 🧠 MemTable

- In-memory store of recent writes.  
- Sorted by partition key and clustering columns.  
- Fast reads and writes supported.  
- Multiple MemTables exist when flushing.

---

## 💽 SSTables (Sorted String Tables)

- Immutable files stored on disk after MemTable flush.  
- Contain sorted partitions and rows.  
- Updates create new SSTables; old ones deleted after compaction.

---

## 🗑️ Tombstones: Soft Deletes

- Deletes mark data with a **tombstone**, not immediate removal.  
- Tombstones expire after ~10 days (configurable).  
- Removed during compaction to reclaim space.  
- Excess tombstones slow reads and consume storage.

---

## ⚡ Reading Data in Cassandra

1. Check **Row Cache** for hot rows.  
2. If miss, check **Bloom filters** to quickly rule out SSTables.  
3. Use **Key Cache** to locate partition offsets.  
4. Search **Partition Index Summary** and **Partition Index** for data offset.  
5. Read from MemTable and SSTables to get latest version.

---

## 🌀 Compaction

- Merges multiple SSTables into one.  
- Removes obsolete data and tombstones.  
- Reduces number of SSTables to scan.  
- Strategies:  
  - **SizeTiered** (default, write-heavy)  
  - **Leveled** (read-optimized)  
  - **TimeWindow** (time-series data)

---

## 🎯 Examples

### Example 1: Read with Quorum Consistency

- `RF = 3`, `W = 2`, `R = 2` (quorum)  
- Write to any 2 nodes, read from any 2 nodes.  
- Ensures `R + W > RF` → strong consistency.

### Example 2: Tombstone Scenario

- Delete data on Node A (which is down).  
- Node B still holds data.  
- Tombstone ensures when Node A recovers and syncs, deleted data isn’t resurrected.

---

        
