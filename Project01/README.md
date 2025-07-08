# 📦 Project 01: Kubernetes Fundamentals & On-Prem Deployment

## 🧠 Learning Objectives

- Understand the core concepts and architecture of Kubernetes.
- Learn how to set up a Kubernetes cluster in an on-premises environment.
- Gain hands-on experience with cluster initialization, node joining, and network setup.
- Practice basic cluster management using `kubectl`.

---

## 📋 Prerequisites

- Basic knowledge of Linux command-line (Ubuntu recommended).
- Understanding of virtualization (used Proxmox for VM setup).
- 3 VMs with Ubuntu 24.04+ installed on Proxmox (1 master-node, 2 worker-node)
- Internet connection for downloading packages.

---

## 🛠️ 1. Kubernetes Fundamentals

### 🔸 What is Kubernetes?
Kubernetes is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications.

### 🔸 Kubernetes Architecture
![image](https://github.com/Cong-SonDao/UIT-TTDN/issues/1#issue-3210946906)

### 🔸 Core Components
- **Pod**: The smallest deployable unit.
- **Node**: A worker machine (physical or virtual).
- **Cluster**: A group of nodes managed by Kubernetes.
- **Deployment**: Describes the desired state for pods and manages updates.
- **Service**: Exposes pods to internal or external traffic.

### 🔸 Basic kubectl Commands
```bash
kubectl get nodes
kubectl get pods --all-namespaces
kubectl describe pod <pod-name>
kubectl apply -f <manifest.yaml>
