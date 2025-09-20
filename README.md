# Deploying and Configuring Ansible AWX on K3s Cluster
This project demonstrates how to deploy and configure AWX open-source version of Ansible on a lightweight Kubernetes distribution (K3s) to provision an operational K8s cluster.

## System Requirements:
  - Linux Ubuntu VM with 2 vCPUs and 8 GB RAM.  
  - Docker/Containerd installed.
  - K3s installed.
    
## Tools and Technologies:
- AWX: Open-source Ansible UI.  
- K3s: Lightweight Kubernetes distribution.
- Ansible: Automation engine for configuration management and orchestration.

  
## Project Setup
## A) Deploy AWX on K3s Cluster
### 1. Create Ubuntu VM 
- Update system packages
  ```bash
  sudo apt update -y
  sudo apt upgrade -y
  ```

- Install packages and dependencies
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


### 4. Create an AWX Instance Manifest
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

### 5. Verify AWX Deployment
- Wait for the AWX deployment to complete, then check the `awx` namespace.
  ```bash
  sudo kubectl -n awx get pods
  sudo kubectl -n awx get all 
  ```
  <img width="1918" height="867" alt="Screenshot 2025-09-18 022411" src="https://github.com/user-attachments/assets/238925eb-d4cc-4cb0-9f59-aa59d30ac9b4" />


### 6. Access AWX Dashboard
- Retrieve the dashboard password from the Secret.
  ```bash
  sudo kubectl -n awx get secret awx-admin-password -o jsonpath="{.data.password}" | base64 --decode && echo
  ```
- Use the IP address of Ubuntu VM and the port number of the 'awx-service' to access the dashboard UI.
- The username is 'admin' and the password is the retrived one from the Secret.
  
  <img width="3824" height="1977" alt="Screenshot 2025-09-18 022849" src="https://github.com/user-attachments/assets/a4c6c34a-98b4-456e-83df-549f4e53be25" />

## B) Configure AWX to Provision K8s Cluster
- The deployed AWX instance on K3s cluster will be used to monitor an operatonal K8s cluster composed of 5 nodes. They are 3 control plane nodes and 2 worker nodes.

### 1. Verify AWX Prerequisites on Managed Nodes  
  - **Network configuration**: Ensure network connectivity between the AWX controller and all managed nodes.
  
  - **SSH service**: Activate or install the SSH service on all managed nodes.
  
  - **Create user for AWX**: AWX will use this user to manage the node.
    ```bash
    sudo useradd ansible
    sudo passwd ansible
    sudo mkdir -p /home/ansible
    sudo chown ansible:ansible /home/ansible
    sudo chmod 700 /home/ansible
    ```
  - **Add ansible user to the sudoers file**.
    ```bash
    sudo vim /etc/sudoers
    ansible ALL=(ALL) NOPASSWD: ALL
     ```

  - **Install Python 3**: Ansible requires Python on the managed nodes:
    ```bash
    sudo apt install python3
    ```
- **Generate access keypair**:
  - Create public and private keys to allow AWX controller to access the managed nodes without password each time it connects to a host.
  - On AWX VM, generate the keypair and copy the public key to each managed node.
  - Replace <Node.IP> with the actual IP address of the managed node.
    ```bash
    ssh-keygen
    ssh-copy-id ansible@<Node.IP>
    ```
  
### 2. Create Inventroy and Add Hosts
  - From AWX UI dashboard, choose Resources --> Inventories and create 'k8s-servers' Inventory.
  - After creating the inventory, create the hosts (managed nodes of the K8s cluster) and assign them to the created inventory. Grouping them is an option.
    
    <img width="3811" height="1966" alt="Screenshot 2025-09-20 133600" src="https://github.com/user-attachments/assets/5f8dd14a-d4cb-472b-82bf-c1d09b02511c" />

### 3. Create Credential
  - From AWX UI dashboard, choose Resources --> Credentials to add new credential, i.e., the "ansible" user created on the managed nodes.
  - For "Credential Type", choose "Machine". This is the credentail type used to SSH nodes.
  - For the username, write "ansible" and upload the private key of the keypair generated in Step 1.
    
    <img width="3750" height="1964" alt="Screenshot 2025-09-20 152810" src="https://github.com/user-attachments/assets/8105bbff-1a77-45be-95d1-18907a034df0" />

### 4. Create AWX Project and Assign Playbook
  - Create YAML file for the required playbook. In this project "ping_playbook.yml" is used to test the connectivity with the managed nodes.
  - From AWX UI dashboard, choose Resources --> Projects to add new project.
  - Select the "Source Control Type" to be Git and indicate the Source Control URL of repo.
  - The AWX server will clone the remote repo under "/var/lib/awx/projects". This path is located on the AWX VM (server), or inside "awx-task" Pod in case AWX is deployed on K8s/K3s cluster.
    
    <img width="3811" height="1878" alt="Screenshot 2025-09-20 162421" src="https://github.com/user-attachments/assets/5f5dc2b1-94be-4a2f-8f5a-c1bcb8169295" />

### 5. Create Template
- AWX template combines inventory, credential, project, and the playbook into a single definition.
- From AWX UI dashboard, choose Resources --> Templates to create new job template.
- Select the associated inventory, project, playbook and credentials.
- Choose "Privilege Escalation" option to run this playbook as an administrator.

  <img width="3824" height="1980" alt="Screenshot 2025-09-20 171705" src="https://github.com/user-attachments/assets/ba54e2f6-2aa1-445c-8609-540039f0d0e4" />

### 6. Launch Job and Check Execution Output
- From AWX UI dashboard, choose Templates and luanch the required templates.
  <img width="3808" height="1974" alt="Screenshot 2025-09-20 173141" src="https://github.com/user-attachments/assets/115ecbda-96e7-4765-8100-fc65da250444" />

- Once the template is launched, the AWX UI will be directed automatically to Jobs --> Output section to visualize the results of the execution.
- 
