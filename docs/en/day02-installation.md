
# Installing Apache Cassandra on Linux and Windows

This guide provides complete step-by-step instructions to install Apache Cassandra on **Linux (Debian/Ubuntu and CentOS/RHEL)** and **Windows** using WSL or Docker.

---

## Installing Apache Cassandra on Linux

### Prerequisites

- Linux machine (Debian/Ubuntu or CentOS/RHEL)
- Java 8 or 11 installed
- curl or wget installed
- sudo privileges
- Internet connection

### Installation on Ubuntu/Debian

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version

curl https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor -o /usr/share/keyrings/cassandra.gpg

echo "deb [signed-by=/usr/share/keyrings/cassandra.gpg] https://downloads.apache.org/cassandra/debian 40x main" | sudo tee /etc/apt/sources.list.d/cassandra.list

sudo apt update
sudo apt install cassandra -y

sudo systemctl enable cassandra
sudo systemctl start cassandra
sudo systemctl status cassandra

cqlsh
```

### Installation on CentOS/RHEL

```bash
sudo yum install java-11-openjdk -y
java -version

sudo tee /etc/yum.repos.d/cassandra.repo <<EOF
[cassandra]
name=Apache Cassandra
baseurl=https://downloads.apache.org/cassandra/redhat/40x/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://downloads.apache.org/cassandra/KEYS
EOF

sudo yum install cassandra -y

sudo systemctl enable cassandra
sudo systemctl start cassandra
sudo systemctl status cassandra

cqlsh
```

---

## Installing Apache Cassandra on Windows (via WSL and Docker)

This guide explains how to install Apache Cassandra on **Windows** using two recommended methods:

- [1] Using **WSL (Windows Subsystem for Linux)**
- [2] Using **Docker**

---

### Prerequisites

- Windows 10 or 11
- WSL enabled (for Method 1)
- Docker Desktop installed (for Method 2)
- A stable internet connection

---

### Method 1: Installing Cassandra using WSL

This method runs a Linux-based Cassandra inside a WSL environment on Windows.

#### Step 1: Install WSL and Ubuntu

Open PowerShell as Administrator and run:

```powershell
wsl --install
```

This installs WSL and the default Ubuntu distribution. If already installed, update it with:

```powershell
wsl --update
```

Then restart your PC if prompted.

---

#### Step 2: Launch Ubuntu and install Cassandra

Open the Ubuntu terminal from the Start menu and run:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Verify Java:

```bash
java -version
```

Add the Cassandra repository and install:

```bash
curl https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor -o /usr/share/keyrings/cassandra.gpg

echo "deb [signed-by=/usr/share/keyrings/cassandra.gpg] https://downloads.apache.org/cassandra/debian 40x main" | sudo tee /etc/apt/sources.list.d/cassandra.list

sudo apt update
sudo apt install cassandra -y
```

---

#### Step 3: Start Cassandra and test

```bash
sudo systemctl start cassandra
cqlsh
```

If the `cqlsh` prompt appears, Cassandra is installed successfully inside WSL.

---

### Method 2: Installing Cassandra using Docker

This is the easiest and most portable way to run Cassandra on Windows.

#### Step 1: Install Docker Desktop

Download and install Docker Desktop for Windows:  
👉 https://www.docker.com/products/docker-desktop/

Make sure Docker is running before continuing.

---

#### Step 2: Run Cassandra container

In a PowerShell or CMD window, run:

```bash
docker pull cassandra
docker run --name cassandra -p 9042:9042 -d cassandra
```

This starts a Cassandra container and maps the default CQL port (9042) to your host machine.

---

#### Step 3: Connect to Cassandra

Use the CQL shell inside the container:

```bash
docker exec -it cassandra cqlsh
```

You should see the `cqlsh>` prompt. You're now connected to your Cassandra instance running in Docker.

---

### Tips and Notes

- Cassandra data in Docker is **ephemeral** unless volumes are configured.
- In WSL, you can manage Cassandra just like in a native Ubuntu system.
- You can install GUI tools like [DataStax Studio](https://www.datastax.com/studio) to explore data visually.

---

### References

- [Apache Cassandra Official Docs](https://cassandra.apache.org/doc/latest/)
- [Docker Hub: cassandra image](https://hub.docker.com/_/cassandra)
- [Microsoft WSL Guide](https://learn.microsoft.com/en-us/windows/wsl/)
