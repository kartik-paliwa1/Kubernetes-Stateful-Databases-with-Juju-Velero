# Kubernetes-Stateful-Databases-with-Juju-Velero (Single EC2 VM)

## Video Explaination: 
<a href="https://www.tella.tv/video/vid_cmk4ewuy600dv04laa9acbxmx">
  <img src="assets/video-thumbnail.png" alt="Watch the video — Kubernetes Stateful Databases with Juju & Velero" width="480" height="270" style="max-width:100%; height:auto;">
</a>

## Project Overview

This project demonstrates how to run **stateful databases reliably on Kubernetes** using **Juju Kubernetes operators** and implement **backup & disaster recovery** using **Velero**, all on a **single AWS EC2 instance**.

The setup simulates a **real-world production scenario** including:

* Multi-node Kubernetes cluster
* Stateful databases with persistent storage
* Application lifecycle management
* Failure simulation
* Backup and restore

---

##  What This Project Solves

Kubernetes is excellent for stateless workloads, but managing **stateful applications like databases** requires additional tooling.

This project addresses:

* Database deployment and lifecycle management
* Persistent storage handling
* Safe scaling and recovery
* Backup and disaster recovery

---

##  Architecture Overview

* **Platform**: AWS EC2 (single VM)
* **Kubernetes Cluster**: 6 nodes (3 control-plane + 3 workers)
* **Cluster Tool**: KIND (Kubernetes in Docker)
* **Application Manager**: Juju
* **Databases**:

  * MySQL
  * PostgreSQL
  * MongoDB
* **Storage**: Kubernetes Persistent Volumes (PVCs)
* **Backup & DR**: Velero with AWS S3

---

##  Tools Used & Why

| Tool           | Why It Is Used                                                |
| -------------- | ------------------------------------------------------------- |
| Docker         | Container runtime, required by KIND                           |
| KIND           | Create multi-node Kubernetes cluster on a single VM           |
| kubectl        | Interact with Kubernetes cluster                              |
| Juju           | Manage application lifecycle (deploy, scale, relate, recover) |
| Juju Operators | Database-specific operational logic                           |
| Velero         | Backup and disaster recovery for Kubernetes                   |
| AWS S3         | Backup storage backend                                        |

---

##  Step-by-Step Setup

### 1️⃣ Update System & Install Essentials

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl wget snapd unzip
```

---

### 2️⃣ Install Docker

```bash
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
newgrp docker
```

---

### 3️⃣ Install kubectl

```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

---

### 4️⃣ Install KIND

```bash
curl -Lo kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
```

---

### 5️⃣ Create 6-Node Kubernetes Cluster

```bash
kind create cluster --name k8s-6node --config kind-6node.yaml
kubectl get nodes
```

---

### 6️⃣ Install Juju

```bash
sudo snap install juju --classic
juju version
```

---

### 7️⃣ Bootstrap Juju on Kubernetes

```bash
juju add-k8s kind-k8s
juju bootstrap kind-k8s juju-controller
juju add-model k8s-model
juju switch k8s-model
```

---

##  Deploy Stateful Databases

### Deploy Databases Using Juju Operators

```bash
juju deploy mysql-k8s mysql
juju deploy postgresql-k8s postgresql
juju deploy mongodb-k8s mongodb
```

Check status:

```bash
juju status
```

---

##  Persistent Storage Verification

```bash
kubectl get pvc -n k8s-model
```

✔ PVCs automatically created
✔ Data persists across pod restarts

---

##  Application Relations (Example)

```bash
juju deploy wordpress-k8s wordpress
juju relate wordpress mysql
```

Juju automatically:

* Creates database
* Generates credentials
* Shares secrets securely

---

##  Scaling & Self-Healing

```bash
juju scale-application mysql 3
juju scale-application postgresql 3
```

Failure simulation:

```bash
kubectl delete pod mysql-0 -n k8s-model
```

✔ Pod recreated
✔ Data preserved
✔ Cluster remains healthy

---

##  Backup & Disaster Recovery with Velero

### Install Velero CLI

```bash
curl -LO https://github.com/vmware-tanzu/velero/releases/latest/download/velero-linux-amd64.tar.gz
tar -xvf velero-linux-amd64.tar.gz
sudo mv velero-v*/velero /usr/local/bin/
```

---

### Install Velero in Kubernetes

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0 \
  --bucket juju-velero-backup \
  --backup-location-config region=ap-south-1 \
  --secret-file ./credentials-velero \
  --use-volume-snapshots=false
```

Verify:

```bash
velero backup-location get
```

---

### Create Backup

```bash
velero backup create db-backup --include-namespaces k8s-model
velero backup get
```

---

### Restore Test (Disaster Recovery)

```bash
kubectl delete pod mysql-0 -n k8s-model
velero restore create --from-backup db-backup
velero restore get
```

✔ Restore completed
✔ Databases healthy
✔ No data loss

---

## Key Learnings

* Managing stateful apps on Kubernetes
* Juju operators simplify complex operations
* Persistent storage is critical for databases
* Velero enables real disaster recovery
* Failure simulation builds production confidence

---

## Final Outcome

✔ Multi-node Kubernetes cluster on a single VM
✔ Stateful databases running safely
✔ Automatic recovery and scaling
✔ Backup & restore verified
✔ Production-style architecture

---
