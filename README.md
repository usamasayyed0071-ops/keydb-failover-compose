#  KeyDB Failover Setup using Docker Compose

##  Overview

This project demonstrates a **KeyDB master–replica setup** using:

* Docker
* Docker Compose

The goal is to simulate **high availability**, ensuring data remains accessible even if the master node goes down.

---

##  Architecture

```
Client → Master → Replica
              ↓
        (Master fails)
              ↓
        Replica serves data
```

### Components

* **Master**

  * Handles write operations
* **Replica**

  * Syncs data from master
  * Serves read requests
* **Failover Behavior**

  * Replica continues serving **read-only data** if master fails

>  Note: This is **not automatic failover** (no promotion to master).

---

##  Prerequisites

* Linux system (Ubuntu recommended)
* Docker & Docker Compose installed

---

##  Step 1: Install Docker

```bash
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl start docker
sudo systemctl enable docker
```

### (Optional: Run Docker without sudo)

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## Step 2: Setup Environment File

Copy the example file:

```bash
cp .env.example .env
```

Edit the file:

```bash
nano .env
```

Update values if needed:

```env
MASTER_PORT=6379
REPLICA_PORT=6380
KEYDB_PASSWORD=12345
```

---

##  Step 3: Start Containers

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

---

##  Step 4: Test Replication

### On Master

```bash
docker exec -it master keydb-cli -a $KEYDB_PASSWORD
```

```bash
SET name "usama"
GET name
```

---

### On Replica

```bash
docker exec -it replica keydb-cli -a $KEYDB_PASSWORD
```

```bash
GET name
```

Expected Output:

```
"usama"
```

---

##  Step 5: Test Failover Scenario

### Stop Master

```bash
docker stop master
```

### Access Replica

```bash
docker exec -it replica keydb-cli -a $KEYDB_PASSWORD
```

```bash
GET name
```

 Expected Output:

```
"usama"
```

---

##  Step 6: Validate Behavior

*  Master handles writes
*  Replica syncs data
*  Replica serves data when master is down
*  Replica is **read-only**

---

##  Limitations

* No automatic failover
* No master election
* Manual recovery required
* Single replica (not production-ready)

---



It provides a strong foundation for learning **DevOps and distributed systems**.
