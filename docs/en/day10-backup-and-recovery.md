# Day 10 - Backup and Recovery Strategies in Apache Cassandra

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)

Welcome to **Day 10** of our Apache Cassandra learning journey! 🎉 Today, we dive into **Backup and Recovery Strategies** in Cassandra, a critical aspect of managing a distributed database system. This guide covers automatic and manual recovery methods, Snapshot management, and Docker-specific commands. By mastering these techniques, you can protect your data from failures and keep your system robust. 🚀

In this guide, we will:

- Explore automatic and manual recovery methods in Cassandra 📝
- Explain how to use Commit Log and Snapshots for backup and recovery 🛠️
- Provide Docker-specific commands for backup and recovery 🐳
- Review essential tools and best practices for recovery strategies 🔍
- Present a practical example of backup and recovery 📚

---

## 🎯 Goal of This Day

Our goal is to learn various methods for protecting and recovering data in Cassandra, including:

- Leveraging built-in mechanisms like Hinted Handoff, Read Repair, and Commit Log for automatic recovery.
- Creating and managing Snapshots for manual backups.
- Executing backup and recovery commands in Docker environments.
- Implementing robust strategies for production environments.

---

## 🛠️ Data Recovery Methods in Cassandra

### Automatic Recovery

Cassandra uses built-in mechanisms to recover data when nodes fail:

- **Hinted Handoff**: When a node is unavailable, other nodes temporarily store its data. Once the node is back online, the data is transferred to it.
- **Read Repair**: During data reads, Cassandra detects and corrects inconsistencies between replicas.
- **Anti-Entropy Repair**: Running `nodetool repair` synchronizes data across replicas to ensure consistency.

### Recovery with Commit Log

If a node crashes, Cassandra reads the Commit Log upon restart to apply changes that were not yet written to SSTables, ensuring no data is lost.

### Recovery with Snapshots

Cassandra does not provide automatic backups by default. For backups, **Snapshots** are used, which are point-in-time copies of SSTables. Snapshots can be used for full or partial data recovery.

---

## 📸 Creating and Restoring Snapshots in Cassandra

### Creating a Snapshot

To take a Snapshot of a keyspace, use the following command:

```bash
nodetool snapshot <keyspace_name> -t <snapshot_name>
```

Example:

```bash
nodetool snapshot my_keyspace -t snapshot_2025_08_05
```

This creates a Snapshot of all tables in the `my_keyspace` keyspace, named `snapshot_2025_08_05`.

### Listing Snapshots

To view available Snapshots:

```bash
nodetool listsnapshots
```

This displays details about Snapshots, including names, keyspaces, and sizes.

### Restoring Data from a Snapshot

To restore data from a Snapshot, follow these steps:

1. **Stop the Cassandra service**:

   ```bash
   sudo service cassandra stop
   ```

2. **Clear existing data**:

   Remove current data from the data directory (specified in `cassandra.yaml` under `data_file_directories`):

   ```bash
   rm -rf /var/lib/cassandra/data/my_keyspace/*
   ```

3. **Copy Snapshot files to the data directory**:

   Snapshot files are typically stored in:

   ```
   /var/lib/cassandra/data/<keyspace>/<table>/snapshots/<snapshot_name>/
   ```

   Copy them to the main data directory:

   ```bash
   cp -r /var/lib/cassandra/data/my_keyspace/mytable/snapshots/snapshot_2025_08_05/* /var/lib/cassandra/data/my_keyspace/mytable/
   ```

4. **Restart the Cassandra service**:

   ```bash
   sudo service cassandra start
   ```

---

## 🐳 Backup and Recovery Commands in Docker

When running Cassandra in Docker, commands are executed inside the container, and data paths are tied to Volumes or Bind Mounts.

### Taking a Snapshot in Docker

Assuming the container name is `cassandra-node1`:

```bash
docker exec -it cassandra-node1 nodetool snapshot my_keyspace -t snapshot_2025_08_05
```

### Listing Snapshots in Docker

```bash
docker exec -it cassandra-node1 nodetool listsnapshots
```

### Restoring from a Snapshot in Docker

1. **Stop the container**:

   ```bash
   docker stop cassandra-node1
   ```

2. **Clear existing data**:

   If using a Volume, inspect its path:

   ```bash
   docker volume inspect cassandra_data_volume
   ```

   If using a Bind Mount (e.g., `/path/on/host` mapped to `/var/lib/cassandra/data`), clear the data:

   ```bash
   rm -rf /path/on/host/my_keyspace/*
   ```

3. **Copy Snapshot files**:

   Copy Snapshot files from the backup location to the data directory:

   ```bash
   cp -r /backup/cassandra_snapshots/my_keyspace/mytable/snapshot_2025_08_05/* /path/on/host/my_keyspace/mytable/
   ```

4. **Restart the container**:

   ```bash
   docker start cassandra-node1
   ```

### Example Docker Compose for Cassandra

To simplify management, use Docker Compose to set up a Cassandra cluster:

```yaml
version: '3.8'
services:
  cassandra:
    image: cassandra:latest
    container_name: cassandra-node1
    volumes:
      - cassandra_data:/var/lib/cassandra
      - ./backups:/backup
    environment:
      - CASSANDRA_CLUSTER_NAME=MyCluster
    ports:
      - "9042:9042"
volumes:
  cassandra_data:
```

This configuration sets up a single Cassandra node with a Volume for data and a backup directory.

---

## 🔧 Essential Tools and Commands for Recovery and Maintenance

| Tool/Command | Purpose |
|--------------|---------|
| `nodetool repair` | Synchronizes data across replicas for consistency |
| `nodetool snapshot` | Takes a Snapshot for backup |
| `nodetool listsnapshots` | Lists available Snapshots |
| `nodetool clearsnapshot` | Deletes old Snapshots |
| `nodetool flush` | Forces Memtable data to SSTables |
| Commit Log | Recovers unwritten changes to SSTables |

---

## 🛡️ Key Tips for a Robust Recovery Strategy

- **Take Periodic Snapshots**: Regularly create Snapshots and store them in a secure location (e.g., separate server or Cloud storage).
- **Configure Consistency Levels**: Set consistency levels (e.g., QUORUM or ONE) based on project requirements.
- **Run Periodic Repairs**: Use `nodetool repair` regularly to keep data synchronized across the cluster.
- **Enable Monitoring**: Use monitoring tools (e.g., Prometheus or DataStax OpsCenter) to detect issues early.
- **Manage Commit Logs**: Configure Commit Log settings in `cassandra.yaml` to optimize resource usage.
- **Production Backup Strategy**: Implement advanced solutions like incremental backups or Cloud storage (e.g., AWS S3) for production environments.

---

## 📚 Practical Example: Backup and Recovery

### 1. Take a Snapshot of a Keyspace

```bash
nodetool snapshot my_keyspace -t mybackup_2025_08_05
```

In Docker:

```bash
docker exec -it cassandra-node1 nodetool snapshot my_keyspace -t mybackup_2025_08_05
```

### 2. Locate Snapshot Files

```bash
ls /var/lib/cassandra/data/my_keyspace/mytable/snapshots/mybackup_2025_08_05/
```

In Docker (on the host, if using a Bind Mount):

```bash
ls /path/on/host/my_keyspace/mytable/snapshots/mybackup_2025_08_05/
```

### 3. Stop Service and Clear Data

```bash
sudo service cassandra stop
rm -rf /var/lib/cassandra/data/my_keyspace/mytable/*
```

In Docker:

```bash
docker stop cassandra-node1
rm -rf /path/on/host/my_keyspace/mytable/*
```

### 4. Copy Snapshot Files

```bash
cp -r /var/lib/cassandra/data/my_keyspace/mytable/snapshots/mybackup_2025_08_05/* /var/lib/cassandra/data/my_keyspace/mytable/
```

In Docker:

```bash
cp -r /path/on/host/my_keyspace/mytable/snapshots/mybackup_2025_08_05/* /path/on/host/my_keyspace/mytable/
```

### 5. Restart Service

```bash
sudo service cassandra start
```

In Docker:

```bash
docker start cassandra-node1
```

---

## 📝 Practical Exercise

To master these concepts, try this exercise:

1. Create a sample keyspace with multiple tables and test data.
2. Take a Snapshot of the keyspace and store it in a secure location.
3. Delete the current data and restore it using the Snapshot.
4. Repeat the process in a Docker environment.
5. Write a simple script to automate the backup process (e.g., using Bash or Python).

### Sample Automated Backup Script:

```bash
#!/bin/bash
# Script to take a snapshot and move it to a backup directory
KEYSPACE="my_keyspace"
SNAPSHOT_NAME="backup_$(date +%Y_%m_%d)"
BACKUP_DIR="/backup/cassandra_snapshots"

docker exec -it cassandra-node1 nodetool snapshot $KEYSPACE -t $SNAPSHOT_NAME
mkdir -p $BACKUP_DIR/$KEYSPACE
cp -r /path/on/host/$KEYSPACE/*/snapshots/$SNAPSHOT_NAME/* $BACKUP_DIR/$KEYSPACE/
echo "Backup completed for $KEYSPACE: $SNAPSHOT_NAME"
```

---

## 🚀 Summary and Final Notes

Cassandra facilitates automatic and manual recovery through mechanisms like Hinted Handoff, Read Repair, Commit Log, and Snapshots. Snapshots are the primary tool for manual backups, and periodic `nodetool repair` runs are essential for data consistency. In Docker environments, commands are similar but executed inside the container. By configuring appropriate consistency levels and using monitoring tools, you can build a reliable recovery strategy.

