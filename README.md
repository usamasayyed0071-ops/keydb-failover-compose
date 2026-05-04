# KeyDB Failover Setup using Terraform, Docker Compose, and Azure VM

## Overview
This project demonstrates how to create a KeyDB failover scenario using Terraform to provision an Azure Ubuntu virtual machine (8GB RAM), Docker to run containers, and Docker Compose to manage KeyDB master-replica setup. The goal is to achieve high availability where data remains accessible even if the master node fails.

## Step 1: Create Infrastructure using Terraform

### provider.tf
provider "azurerm" {
  features {}
}

### variables.tf
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

### outputs.tf
output "public_ip" {
  value = azurerm_public_ip.public_ip.ip_address
}

### main.tf (basic structure)
Includes:
- Resource Group
- Virtual Network
- Subnet
- Public IP
- Network Interface
- Linux Virtual Machine

Run commands:
terraform init
terraform apply

## Step 2: Connect to VM
ssh -i vm-codoagent_key.pem azureuser@<public-ip>

## Step 3: Install Docker
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl start docker
sudo systemctl enable docker

(Optional)
sudo usermod -aG docker $USER
newgrp docker

## Step 4: Create Project Directory
mkdir keydb-setup
cd keydb-setup

## Step 5: Create .env File
nano .env

Add:
MASTER_PORT=6379
REPLICA_PORT=6380
KEYDB_PASSWORD=12345

## Step 6: Create docker-compose.yml
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

## Step 7: Start Containers
docker compose up -d

Check:
docker ps

## Step 8: Test Replication

Enter master:
docker exec -it master keydb-cli

Run:
SET name "usama"
GET name

Enter replica:
docker exec -it replica keydb-cli

Run:
GET name

Expected output:
"usama"

## Step 9: Test Failover Scenario

Stop master:
docker stop master

Check containers:
docker ps

Enter replica:
docker exec -it replica keydb-cli

Run:
GET name

Expected:
"usama"

## Step 10: Validate Behavior
- Master handles writes
- Replica copies data
- Replica serves data when master is down
- Replica is read-only

## Architecture Flow
Client → Master → Replica → (Master fails) → Replica serves data

## Conclusion
This project shows how to build a fault-tolerant KeyDB system using Terraform, Docker, and Docker Compose. It ensures high availability and data safety in real-world DevOps environments.
