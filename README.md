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
### 1. Create Ubuntu VM 
- Update system packages
  ```bash
  sudo apt update -y
  sudo apt upgrade -y
  ```

- Install packages and dependenciesز 
```bash
sudo apt install git make vim -y
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
  <img width="1847" height="377" alt="Screenshot 2025-09-18 015009" src="https://github.com/user-attachments/assets/c134545b-141d-418b-9f18-b20f5e90251f" />


### 4. Create an AWX instance manifest
- Create AWX deployment (instance) file.
- This YAML file represents the request to the AWX Operator to deploy and configure all the AWX components (PostgreSQL database, web UI, task workers, etc.)
  ```bash
  vim awx-deploy.yaml
  ```

- Edit 'YAML' file to expose AWX UI via NodePort. 
  ```bash
  apiVersion: awx.ansible.com/v1beta1
  kind: AWX
  metadata:
    name: awx
    namespace: awx
  spec:
    service_type: NodePort
  ```

- Apply the file to start the deployment.
  ```bash
  sudo kubectl apply -f awx-deploy.yaml
  ```

### 5. Verify the AWX Deployment
- Wait for the AWX deployment to complete, then check the `awx` namespace.
  ```bash
  sudo kubectl -n awx get pods
  sudo kubectl -n awx get all 
  ```
  <img width="1918" height="867" alt="Screenshot 2025-09-18 022411" src="https://github.com/user-attachments/assets/238925eb-d4cc-4cb0-9f59-aa59d30ac9b4" />


### 6. Access AWX dashboard
- Retrieve the dashboard password from the Secret.
  ```bash
  sudo kubectl -n awx get secret awx-admin-password -o jsonpath="{.data.password}" | base64 --decode && echo
  ```
- Use the IP address of Ubuntu VM and the port number of the 'awx-service' to access the dashboard UI.
- The username is 'admin' and the password is the retrived one from the Secret.
  
  <img width="3824" height="1977" alt="Screenshot 2025-09-18 022849" src="https://github.com/user-attachments/assets/a4c6c34a-98b4-456e-83df-549f4e53be25" />




