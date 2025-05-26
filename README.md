# 🚀 Task 2: Redis Cluster Setup (3 Node Cluster on AWS EC2)

## 📌 Objective
Set up a Redis Cluster with 3 EC2 instances acting as Redis nodes using custom ports, with `cluster-enabled` mode. This guide assumes Ubuntu-based EC2 instances.

---

## 🛠️ Requirements

- 3 AWS EC2 Ubuntu instances
- Open ports: `7000-7002` and `17000-17002`
- Redis installed on each instance

---

## 🧩 Step-by-Step Setup

### 🖥️ 1. Install Redis on All EC2 Instances
```bash
sudo apt update
sudo apt install redis-server -y
```
### ⚙️ 2. Configure Redis for Clustering
Create Redis Config File for Each Node

For example, on instance 1 for port 7000:

```
sudo mkdir -p /etc/redis
sudo nano /etc/redis/7000.conf
```

Paste the following configuration:

```conf
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes
dir /var/lib/redis/7000
logfile "/var/log/redis_7000.log"
daemonize yes
```

Repeat for the other instances with ports 7001 and 7002.

### 📁 3. Create Redis Data Directories
On each instance:

```bash
sudo mkdir -p /var/lib/redis/7000
sudo chown redis:redis /var/lib/redis/7000
```
Repeat for 7001, 7002.

### 🚀 4. Start Redis Server on Each Node
```bash

sudo redis-server /etc/redis/7000.conf
sudo redis-server /etc/redis/7001.conf
sudo redis-server /etc/redis/7002.conf
```

Check if the servers are running:

```bash
sudo netstat -tulnp | grep redis
```

### 🔓 5. Open Required Ports
🔥 On UFW (Local Firewall)

```bash
sudo ufw allow 7000
sudo ufw allow 17000
```

### Repeat for 7001/17001 and 7002/17002
🌐 On AWS Security Group

Open inbound TCP ports 7000-7002 and 17000-17002 for each instance from other cluster members.

### 🧹 6. Clean Existing Data (Optional)
```bash

redis-cli -p 7000 FLUSHALL
redis-cli -p 7001 FLUSHALL
redis-cli -p 7002 FLUSHALL
```

### 🧱 7. Create the Redis Cluster
Run this from one instance only:

```bash
redis-cli --cluster create \
  172.31.1.193:7000 \
  172.31.0.164:7001 \
  172.31.7.201:7002 \
  --cluster-replicas 0
```

When prompted:

```pgsql
Can I set the above configuration? (type 'yes' to accept): yes
```

### ✅ 8. Verify the Cluster
Run:

```bash
redis-cli -c -p 7000 cluster nodes

```
You should see output showing all three nodes with proper slot allocation:

Node 1: Slots 0 - 5460

Node 2: Slots 5461 - 10922

Node 3: Slots 10923 - 16383

----------------------------------------------
### 📝Summary
✅Redis installed on 3 nodes
✅Configured with clustering enabled
✅Required ports opened
✅Redis Cluster created successfully
✅All 16384 hash slots assigned

