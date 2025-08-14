# aws-portfolio-project-4
Capstone Project -SAA C03 Final Work (Hands-On)

# AWS Capstone Project – LAMP App with RDS, ALB, and Auto Scaling

This project provisions a multi-tier AWS architecture for a PHP/MySQL inventory application, with automation and scaling for production-style deployment.

## Architecture Overview

- VPC with Public and Private subnets across two AZs.
- RDS MySQL Multi-AZ database in private subnets.
- Cloud9 for development and AMI creation.
- LAMP stack web server running PHP application.
- Systems Manager Parameter Store for DB credentials.
- Application Load Balancerrouting traffic to private instances.
- Auto Scaling Group for high availability.

## Prerequisites

- AWS account with sufficient permissions.
- Key pair for SSH access (optional if using Cloud9 direct access).
- AWS CLI installed (optional but recommended).


## Task 0 – Environment Inspection

### Subnets
- Public Subnet 1: `10.0.0.0/24` (us-east-1a)
- Public Subnet 2: `10.0.1.0/24` (us-east-1b)
- Private Subnet 1: `10.0.2.0/23` (us-east-1a)
- Private Subnet 2: `10.0.4.0/23` (us-east-1b)

### Security Groups
- `ALBSG` – ALB inbound 80/443 from 0.0.0.0/0
- `Bastion-SG` – SSH inbound from your IP
- `Example-DB` – MySQL inbound from app SG
- `Inventory-App` – HTTP inbound from ALB SG

---

## Task 1 – Create MySQL RDS Instance

1. **Create Subnet Group**: `Example-DB-subnet` with private subnets in 1a and 1b.
2. **Create DB**:
   - Engine: MySQL
   - Multi-AZ: Yes
   - Instance: `t3.micro`
   - Allocated storage: 20GB, autoscaling enabled
   - Public access: No
   - VPC SG: `Example-DB`
   - Initial DB: `exampledb`
   - Credentials: `admin` / `password` (lab use only)

---

## Task 2 – Cloud9 Environment

- Name: `capstone project`
- Instance: `t2.micro`, Amazon Linux 2
- Network: Example VPC, Public Subnet 2
- Direct access (no SSH key required)

---

## Task 3 – Install LAMP and App Code

```bash
sudo yum update -y
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo yum install -y httpd mariadb-server unzip
sudo systemctl start httpd
sudo systemctl enable httpd









