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
