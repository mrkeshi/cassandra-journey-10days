# Installing Apache Cassandra on Windows and Linux

This guide provides step-by-step instructions to install Apache Cassandra on **Linux (Debian/Ubuntu and CentOS/RHEL)** systems. Installation resources for Windows are also included at the end.

---

## Installation on Ubuntu / Debian

### Step 1: Install Java

Apache Cassandra requires Java (either OpenJDK 8 or 11). To install Java 11:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version ```

### Step 2: Add the Apache Cassandra repository and GPG key