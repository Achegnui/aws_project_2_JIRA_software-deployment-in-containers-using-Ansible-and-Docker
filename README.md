# Containerized JIRA Deployment with Ansible and Docker

This project automates the deployment of **JIRA Software** using **Ansible** and **Docker**, ensuring efficiency, scalability, and consistent deployments. Follow this step-by-step guide to set up a fully containerized JIRA environment with centralized management and orchestration.

---

## Prerequisites

Before starting, ensure the following:

1. **AWS Account**: Access to create EC2 instances.
2. **Git Bash/Terminal**: Installed on your local machine.
3. **Basic Knowledge**: Familiarity with Linux commands, Ansible, and Docker.

---

## Deployment Steps

### Step 1: Launch EC2 Instances

1. Launch **three Amazon Linux 2 EC2 instances**:
   - `Ansible_Control` (Control Node)
   - `Worker_Node_1` and `Worker_Node_2` (Worker Nodes)
2. Configure **Security Groups**:
   - `Ansible_Control`: Allow **SSH** from your IP.
   - `Worker_Nodes`: Allow **SSH** from `Ansible_Control` and your IP.

**Image Suggestion**: Diagram showing the `Ansible_Control` node connected to `Worker_Node_1` and `Worker_Node_2`, with security group rules labeled.

---

### Step 2: Set Hostnames

1. Login to each instance using Git Bash:
   - Go to the AWS console, select the instance, and click on connect and select SSH and follow the steps.
2. Switch to the root user:
   ```bash
   sudo su -
   ```
3. Edit the hostname file:
   ```bash
   nano /etc/hostname
   ```
4. Set hostnames as:
   - Ansible_Control
   - Worker_Node_1
   - Worker_Node_2
5. Save changes and reboot the instance:
   ```bash
   reboot
   ```

---

## Step 3: Create the Ansible User

1. On all instances:
   ```bash
   sudo su -
   useradd ansible
   passwd ansible
   ```
2. Enable password-based SSH authentication:
   - Edit the SSH configuration file:
     ```bash
     nano /etc/ssh/sshd_config
     ```
   - Update these lines:
     PasswordAuthentication yes
     PermitRootLogin yes
   - Restart the SSH service:
   ```bash
   sudo systemctl restart sshd
   ```
3. Add the ansible user to the sudoers group:
   ```bash
   nano /etc/sudoers
   ```
   - Add the following line to the file positions respectively:
   ```
   ansible ALL=(ALL)
   ansible ALL=(ALL)
   ansible ALL=(ALL) NOPASSWD:ALL
   ```

---

## Step 4: Configure Passwordless SSH

1. On Ansible_Control, generate an SSH keypair:
   ```bash
   sudo su ansible
   ssh-keygen -t rsa
   ```
   Press Enter for all prompts
2. Copy the key to both worker nodes:
   ```bash
   ssh-copy-id ansible@<worker_node_1_private_ip>
   ssh-copy-id ansible@<worker_node_2_private_ip>
   ```
3. Verify passwordless SSH by trying to access each worker node from the control node using each woker node's private IP address:
   ```bash
   ssh ansible@<worker_node_private_ip>
   ```

---

## Step 5: Install Ansible and Docker

1. Install Ansible on Ansible_Control:
   ```bash
   sudo amazon-linux-extras install ansible2 -y
   ansible --version
   ```
2. SSH into Worker_Node_1 and Worker_Node_2 from Ansible_Control and install Docker:
   ```bash
   sudo amazon-linux-extras install docker -y
   sudo service docker start
   sudo systemctl enable docker
   sudo docker run hello-world
   ```

---

## Step 6: Configure the Ansible Inventory

1. On Ansible_Control, edit the Ansible inventory file:
   ```bash
   cd /etc/ansible/
   sudo nano hosts
   ```
2. Remove the "**#**" from [webservers] and add the following under:
   ```
   [webservers]
   <worker_node_1_private_ip>
   <worker_node_2_private_ip>
   ```

---

## Step 7: Create Deployment Files

1. Create a directory for the Docker Compose file:
   ```bash
   mkdir ~/jira-docker
   cd ~/jira-docker
   sudo nano docker-compose.yml
   ```
   Paste the Docker Compose configuration into this file.
2. Create a directory for the Ansible Playbook:
   ```bash
   mkdir ~/ansible-playbooks
   cd ~/ansible-playbooks
   sudo nano deploy_jira.yml
   ```
   Paste the Ansible playbook configuration into this file.

---

## Step 8: Run the Ansible Playbook

1. Navigate to the playbook directory:
   ```bash
   cd ~/ansible-playbooks
   ```
2. Run the playbook:
   ```bash
   ansible-playbook deploy_jira.yml
   ```

---

## Step 9: Configure the Load Balancer

1. Create a Target Group:
   - Protocol: HTTP
   - Port: 8080
   - Register Worker_Node_1 and Worker_Node_2.
2. Create an Application Load Balancer:
   - Protocol: HTTP
   - Listener Port: 80
   - Attach the target group
   - Configure the security group to allow HTTP/HTTPS traffic(Create a security group for the load balancer).
   - It should have a scheme of internet-facing

---

## Step 10: Test the Setup

1. Update Security Group for Worker Nodes:
   - Go to the Security Groups associated with Worker_Node_1 and Worker_Node_2 and ensure that inbound rules allow HTTP traffic (Port 8080) from the Load Balancer's security group.
2. Access the JIRA Application:
   - Copy the DNS of the load balancer, paste in your web browser and navigate to it.
   - You should see the JIRA Software setup page, indicating that the application is successfully running and accessible.
