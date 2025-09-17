# Deploying Ansible AWX on K3s Cluster
This project demonstrates how to deploy and configure AWX open-source version of Ansible Tower on a lightweight Kubernetes distribution (K3s).

## System Requirements:
  - Linux Ubuntu VM with 2 vCPUs and 8 GB RAM.  
  - Docker/Containerd installed.
  - K3s installed.
  - kubectl configured to manage the K3s cluster.  

## Tools and Technologies:
- AWX: Open-source Ansible Tower GUI.  
- K3s: Lightweight Kubernetes distribution.
- Ansible: Automation engine for configuration management and orchestration.

## Project Setup
### 1. Create Ubuntu VM and Update System Packages
```bash
sudo apt update -y
```

### 2. Install K3s
- Run the official install script from Rancher [https://k3s.io/]
```bash
curl -sfL https://get.k3s.io | sh -
```

-  Check for Ready node
```bash 
sudo k3s kubectl get node
```

- Check K3s cluster status
```bash
sudo systemctl status k3s
```

- K3s installs kubectl automatically to alias it for convenience
```bash
alias kubectl='k3s kubectl'
kubectl get nodes
```

- K3s stores its kubeconfig file at `/etc/rancher/k3s/k3s.yaml`. Copy this file to `~/.kube/config` to use the standard `kubectl` command:
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

- Get the status of the nodes (by default, K3s creates one node acting as both control-plane and worker).
```bash
kubectl get nodes
```

### 3.


### 4.

