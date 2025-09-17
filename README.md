# Deploying Ansible AWX on K3s Cluster
This project demonstrates how to deploy and configure AWX open-source version of Ansible Tower on a lightweight Kubernetes distribution (K3s).

## System Requirements:
  - Linux Ubuntu VM with 2 vCPUs and 8 GB RAM.  
  - Docker/Containerd installed.
  - K3s installed.
  - Kubectl configured to manage the K3s cluster.  

## Tools and Technologies:
- AWX: Open-source Ansible Tower GUI.  
- K3s: Lightweight Kubernetes distribution.
- Ansible: Automation engine for configuration management and orchestration.

## Project Setup
### 1. Create Ubuntu VM and Update System Packages
```bash
sudo apt update -y
sudo apt install git make -y

```

### 2. Install K3s
- Run the official install script from Rancher [https://k3s.io/]
```bash
sudo apt install curl -y
curl -sfL https://get.k3s.io | sh -
```

- Check K3s service status
```bash
sudo systemctl status k3s
```

- Check K3s cluster status. K3s creates one node acting as both control-plane and worker.
```bash
sudo kubectl get nodes
```
- K3s bundles its own container runtime by default. It uses containerd (lighter and faster).

### 3. Install AWX Operator
- Create a namespace for AWX and install the operator.
```bash
sudo kubectl create namespace awx
```

- Clone AWX operator repo
```bash
git clone https://github.com/ansible/awx-operator.git
```

- Install a specific version of the AWX operator and build its Kubernetes manifest. 
```bash
cd awx-operator/
git checkout 2.19.0
sudo make deploy
```

- Check the deployment status of AWX operator.
```bash
sudo kubectl -n awx get all
```


### 4.

