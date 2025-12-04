# aws-portfolio-project-4
AWS Capstone Project -SAA C03 Final Work (Hands-On)

# AWS Capstone Project – LAMP App with RDS, ALB, and Auto Scaling

# AWS Portfolio Project #4  

# Author - Andrew Kremmidas - AWS Solutions Architect Associate

## Capstone LAMP Web App — RDS + ALB + Auto Scaling (SAA-C03 Final Build)

### What This Project Demonstrates
A production-style, **highly available** and **secure** multi-tier web application environment on AWS using a classic LAMP stack.

This build mirrors real enterprise workloads — with **private networking**, **scaling**, **load balancing**, and **managed database durability**.

| Capability | Feature Enabled |
|----------|----------------|
| High Availability | Multi-AZ RDS + Multi-AZ load balancing |
| Scalability | EC2 Auto Scaling Group (ASG) self-healing instances |
| Security | Private DB tier + least-privilege Security Groups |
| Traffic Management | Application Load Balancer (ALB) |
| Automation | Cloud9 + AMI bake flow for repeatable deployments |

---

### Architecture Overview

VPC (Custom)  
→ Public Subnets (ALB + Bastion/Cloud9)  
→ Private Subnets (EC2 LAMP app + RDS MySQL)

Users
│
▼
Application Load Balancer (Public Subnets)
│ HTTPS/HTTP routing
▼
EC2 Auto Scaling Group (Private Subnets)
│ Secure DB connection
▼
Amazon RDS MySQL (Multi-AZ Private)

- **Cloud9** used to install app code and create a reusable **golden AMI**
- **Parameter Store** stores DB credentials (no plaintext exposure)

> _Architecture diagram placeholder_  
> `images/architecture.png`

---

### AWS Services Used

| Service | Purpose |
|--------|---------|
| Amazon VPC | Secure network segmentation |
| Amazon EC2 | Web app compute (LAMP stack) |
| Auto Scaling Group | Scale out / self-healing |
| Application Load Balancer | Public entry point + routing |
| Amazon RDS (MySQL Multi-AZ) | Durable backend database |
| AWS Systems Manager Parameter Store | Secure secrets |
| AWS Cloud9 | Dev environment + AMI build |
| Amazon S3 (optional logging) | Central log storage |

---

### Security Features

- **No database exposure to the internet**
- Private subnets for web + DB tiers
- IAM roles instead of stored credentials
- SG inbound rules locked to exact tiers (ALB → EC2 → RDS)
- **HTTPS-only** enforcement via ALB (optional ACM TLS certificate)

---

### Network Configuration

| Component | CIDR | AZ |
|---------|------|----|
| Public Subnet 1 | 10.0.0.0/24 | us-east-1a |
| Public Subnet 2 | 10.0.1.0/24 | us-east-1b |
| Private Subnet 1 | 10.0.2.0/23 | us-east-1a |
| Private Subnet 2 | 10.0.4.0/23 | us-east-1b |

### 🔒 Security Groups Overview

| SG Name | Allows | From |
|--------|--------|------|
| **ALB-SG** | 80/443 | Internet (0.0.0.0/0) |
| **Inventory-App-SG** | 80 | ALB-SG |
| **DB-SG** | 3306 | Inventory-App-SG |
| **Bastion/Cloud9-SG** | 22 | Admin IP |

---

## Step-by-Step Deployment

### Task 1 — Create RDS MySQL Database

- Engine: **MySQL**
- Instance: **t3.micro**
- Multi-AZ: **Enabled**
- Storage: 20 GB + autoscaling
- Public Access: **No**
- DB Name: `exampledb`
- Credentials: `admin / password` (lab only)
- Attach **DB-SG**
- Subnet group: private subnets (`1a` + `1b`)

---

### Task 2 — Cloud9 Development Environment

- Name: `capstone-project`
- Instance: `t2.micro` (Amazon Linux 2)
- VPC: Example VPC → **Public Subnet 2**
- IAM role with SSM access recommended

> No SSH key needed (SSM channel)

---

### Task 3 — EC2 LAMP Stack + App Code Installation

```bash
sudo yum update -y
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo yum install -y httpd mariadb-server unzip
sudo systemctl start httpd
sudo systemctl enable httpd
Install app code + configure DB connection (detailed app steps to be added)
Then:
Bake AMI from configured instance
Deploy Auto Scaling Group using that AMI
Attach ALB Target Group to ASG
Test failover & auto-scaling behavior
