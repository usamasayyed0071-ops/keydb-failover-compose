Here’s your project formatted as a clean **GitHub-ready `README.md` file**:

```markdown
# 🚀 KeyDB Failover Setup using Terraform, Docker Compose, and Azure VM

## 📌 Overview
This project demonstrates how to create a **KeyDB failover setup** using:

- Terraform (Infrastructure as Code)
- Azure Virtual Machine (Ubuntu, 8GB RAM)
- Docker & Docker Compose (Container orchestration)

The goal is to achieve **high availability**, ensuring data remains accessible even if the master node fails.

---

## 🏗️ Architecture

```

Client → Master → Replica → (Master fails) → Replica serves data

````

- **Master**: Handles writes
- **Replica**: Syncs data and serves reads
- **Failover**: Replica continues serving data if master goes down

---

## ⚙️ Step 1: Provision Infrastructure using Terraform

### 📄 provider.tf
```hcl
provider "azurerm" {
  features {}
}
````

### 📄 variables.tf

```hcl
variable "location" {
  default = "Central India"
}

variable "vm_size" {
  default = "Standard_D2s_v3"
}

variable "admin_username" {
  default = "azureuser"
}

variable "public_key_path" {
  default = "vm-codoagent_key.pub"
}
```

### 📄 outputs.tf

```hcl
output "public_ip" {
  value = azurerm_public_ip.public_ip.ip_address
}
```

### 📄 main.tf

Includes:

* Resource Group
* Virtual Network
* Subnet
* Public IP
* Network Interface
* Linux Virtual Machine

### ▶️ Run Terraform

```bash
terraform init
terraform apply
```

---

## 🔐 Step 2: Connect to VM

```bash
ssh -i vm-codoagent_key.pem azureuser@<public-ip>
```

---

## 🐳 Step 3: Install Docker

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

## 📁 Step 4: Create Project Directory

```bash
mkdir keydb-setup
cd keydb-setup
```

---

## 🔑 Step 5: Create `.env` File

```bash
nano .env
```

Add:

```env
MASTER_PORT=6379
REPLICA_PORT=6380
KEYDB_PASSWORD=12345
```

---

## 🧩 Step 6: Create `docker-compose.yml`

```yaml
services:
  master:
    image: eqalpha/keydb
    container_name: master
    ports:
      - "${MASTER_PORT}:6379"

  replica:
    image: eqalpha/keydb
    container_name: replica
    ports:
      - "${REPLICA_PORT}:6379"
    command: keydb-server --replicaof master 6379
```

---

## ▶️ Step 7: Start Containers

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

---

## 🔄 Step 8: Test Replication

### 🔹 On Master

```bash
docker exec -it master keydb-cli
```

```bash
SET name "usama"
GET name
```

### 🔹 On Replica

```bash
docker exec -it replica keydb-cli
```

```bash
GET name
```

✅ Expected Output:

```
"usama"
```

---

## ⚠️ Step 9: Test Failover Scenario

### Stop Master

```bash
docker stop master
```

### Check Containers

```bash
docker ps
```

### Access Replica

```bash
docker exec -it replica keydb-cli
```

```bash
GET name
```

✅ Expected Output:

```
"usama"
```

---

## ✅ Step 10: Validate Behavior

* ✔️ Master handles writes
* ✔️ Replica syncs data
* ✔️ Replica serves data when master is down
* ✔️ Replica is read-only

---

## 🎯 Conclusion

This project demonstrates how to build a **fault-tolerant KeyDB system** using:

* Terraform for infrastructure
* Docker for containerization
* Docker Compose for orchestration

It ensures:

* High availability
* Data reliability
* Real-world DevOps readiness

---

## 📌 Future Improvements

* Add KeyDB Sentinel for automatic failover
* Use Load Balancer for traffic routing
* Secure setup with authentication & firewall rules
* Monitoring with Prometheus + Grafana

---

## 🤝 Contributing

Feel free to fork this repo and submit pull requests!

---

## 📜 License

This project is licensed under the MIT License.

```

---

If you want, I can also:
- Split Terraform into production-grade modules  
- Add Sentinel-based automatic failover  
- Convert this into a **real DevOps portfolio project (resume-ready)**
```
