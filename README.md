# Deploying Ansible AWX on K3s Cluster
This project demonstrates how to deploy and configure AWX open-source version of Ansible Tower on a lightweight Kubernetes distribution (K3s).

## System Requirements:
  - Linux VM (ubuntu) with 2 vCPUs and 8 GB RAM.  
  - Docker/Containerd installed.
  - K3s installed.
  - kubectl configured to manage the K3s cluster.  

## Tools and Technologies:
- AWX: Open-source Ansible Tower GUI.  
- K3s: Lightweight Kubernetes distribution.
- Ansible: Automation engine for configuration management and orchestration.

## Project Setup
### 1. Create Ubuntu VM and update System Packages
```bash
sudo apt update -y
sudo apt upgrade -y
```

### 2. Install K3s
- Run the official install script from Rancher [https://k3s.io/]
```bash
curl -sfL https://get.k3s.io | sh -
```
-  Check for Ready node, takes ~30 seconds
```bash 
sudo k3s kubectl get node
```
