# Todo Application Infrastructure (IaC & CAC)

This repository contains the **Terraform** configurations and **Ansible** playbooks required to provision and configure a production-ready, highly available cloud environment on **AWS**.

## 🏗️ Architecture Overview
The infrastructure is designed for security and scalability:
* **VPC Networking**: A custom VPC with a multi-AZ architecture featuring public and private subnets across two Availability Zones.
* **EKS Cluster**: A managed Kubernetes environment (Amazon EKS) version 1.33, utilizing **Amazon Linux 2023** optimized AMIs for worker nodes.
* **Private Registry**: A dedicated **Sonatype Nexus** instance hosted on an EC2 instance within a private subnet.

---

## 🛠️ Infrastructure Components (Terraform)

### 1. Kubernetes Orchestration & Add-ons
* **EKS Cluster**: Provisioned with both private and public endpoint access and managed node groups.
* **Cluster Autoscaler**: Deployed via Helm to automatically scale worker nodes based on workload demands.
* **Nginx Ingress Controller**: Manages external traffic routing to internal Kubernetes services.
* **Metrics Server**: Provides necessary resource data for the Horizontal Pod Autoscaler (HPA).
* **Pod Identity Agent**: Simplifies IAM permission management for pods by using the EKS Pod Identity add-on.

### 2. Storage & Security
* **Persistent Storage**: A **40GB gp2 EBS volume** is dedicated to Nexus data with `prevent_destroy` enabled to ensure image persistence.
* **SSM Integration**: EC2 instances are configured with IAM roles for **AWS Systems Manager (SSM)**, allowing secure management and Ansible connectivity without public SSH exposure.
* **Security Groups**: Custom rules allow traffic for Nexus (8081), Docker Registry (5000), and standard web ports.

---

## ⚙️ Configuration Management (Ansible)

Once Terraform provisions the hardware, the included **Ansible Playbook** automates the setup of Sonatype Nexus:

* **Storage Setup**: Formats the attached EBS volume as `ext4` and mounts it to `/nexus-ebs` for persistent storage.
* **Environment Prep**: Installs **OpenJDK 17**, creates a dedicated `nexus` user, and configures passwordless sudo for administration.
* **Nexus Installation**: Downloads and installs Nexus 3.83.2, ensuring the application is correctly extracted to the persistent mount.
* **Performance Tuning**: Optimizes JVM settings, specifically setting `MaxDirectMemorySize` to **2703m** and preferring the IPv4 stack.
* **Persistence Configuration**: Reconfigures Nexus default properties to ensure all data and work directories reside on the persistent EBS mount.
* **Service Management**: Creates and enables a `systemd` unit file to ensure Nexus starts automatically on boot.
* **Firewall Configuration**: Opens port `8081` via `ufw` to allow internal traffic.

---
