# AWS EC2 Complete Launch Guide

Complete guide for launching, configuring, and managing AWS EC2 instances from scratch.

---

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI installed (optional but recommended)
- SSH client (Terminal, PuTTY)

---

## 1. Choose an AMI (Amazon Machine Image)

### Recommended AMIs

| AMI | Use Case | Username |
|-----|----------|----------|
| Ubuntu 24.04 LTS | General purpose, web servers | `ubuntu` |
| Ubuntu 22.04 LTS | Stable, wide support | `ubuntu` |
| Amazon Linux 2023 | AWS optimized, lightweight | `ec2-user` |
| Debian 12 | Stable, minimal | `admin` |
| CentOS Stream 9 | Enterprise-like | `centos` |

---

## 2. Launch EC2 via AWS Console

### Step 1: Navigate to EC2

1. Log into AWS Console
2. Go to **EC2** service
3. Click **Launch Instance**

### Step 2: Configure Instance

**Name and Tags:**
```
Name: my-web-server
Environment: production
Project: my-app
```

**Application and OS Images:**
- Select **Ubuntu Server 24.04 LTS (HVM)**
- Architecture: 64-bit (x86)

**Instance Type:**

| Type | vCPUs | Memory | Use Case |
|------|-------|--------|----------|
| t2.micro | 1 | 1 GB | Testing, free tier |
| t3.small | 2 | 2 GB | Light workloads |
| t3.medium | 2 | 4 GB | Web servers |
| t3.large | 2 | 8 GB | Production apps |
| c5.xlarge | 4 | 8 GB | CPU intensive |
| r5.large | 2 | 16 GB | Memory intensive |

**Key Pair:**
- Create new key pair
- Name: `my-server-key`
- Key pair type: RSA
- Format: `.pem` (Linux/Mac) or `.ppk` (PuTTY)
- Download and save securely!

**Network Settings:**

```
VPC: default or custom
Subnet: Public subnet
Auto-assign public IP: Enable
```

**Security Group (Firewall):**

Create new security group:

| Type | Protocol | Port | Source | Description |
|------|----------|------|--------|-------------|
| SSH | TCP | 22 | My IP | SSH access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure web |
| Custom TCP | TCP | 3000 | 0.0.0.0/0 | Node.js (optional) |

**Storage:**

```
Size: 20 GB (minimum for Ubuntu)
Volume type: gp3 (SSD)
IOPS: 3000
Throughput: 125 MB/s
Delete on termination: Yes (or No for persistence)
```

### Step 3: Launch

Click **Launch Instance** and wait for it to start.

---

## 3. Connect to Your Instance

### Get Connection Info

1. Select your instance
2. Click **Connect**
3. Note the Public IP or DNS

### Connect via SSH

```bash
# Set key permissions
chmod 400 my-server-key.pem

# Connect
ssh -i my-server-key.pem ubuntu@<public-ip>

# Or using public DNS
ssh -i my-server-key.pem ubuntu@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

### Add to SSH Config

```bash
nano ~/.ssh/config
```

```
Host aws-server
    HostName <public-ip>
    User ubuntu
    IdentityFile ~/.ssh/my-server-key.pem
```

Now connect with:
```bash
ssh aws-server
```

---

## 4. Initial Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Set hostname
sudo hostnamectl set-hostname my-server

# Set timezone
sudo timedatectl set-timezone Asia/Dhaka

# Install essentials
sudo apt install -y curl wget git htop vim unzip

# Create deploy user (optional)
sudo adduser deployer
sudo usermod -aG sudo deployer
```

---

## 5. Elastic IP (Static IP)

### Allocate Elastic IP

1. Go to EC2 → **Elastic IPs**
2. Click **Allocate Elastic IP address**
3. Select **Amazon's pool of IPv4 addresses**
4. Click **Allocate**

### Associate with Instance

1. Select the Elastic IP
2. Click **Actions** → **Associate Elastic IP address**
3. Choose your instance
4. Click **Associate**

### AWS CLI

```bash
# Allocate
aws ec2 allocate-address --domain vpc

# Associate
aws ec2 associate-address --instance-id i-xxxxxxxxx --allocation-id eipalloc-xxxxxxxxx
```

> **Note:** Elastic IPs are free when associated with a running instance.

---

## 6. Security Groups Best Practices

### Create Security Group

```bash
aws ec2 create-security-group \
    --group-name web-server-sg \
    --description "Web server security group" \
    --vpc-id vpc-xxxxxxxx
```

### Add Rules

```bash
# SSH from your IP
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxx \
    --protocol tcp \
    --port 22 \
    --cidr $(curl -s ifconfig.me)/32

# HTTP from anywhere
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxx \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0
```

### Recommended Security Groups

**Web Server SG:**
| Port | Source | Description |
|------|--------|-------------|
| 22 | My IP | SSH |
| 80 | 0.0.0.0/0 | HTTP |
| 443 | 0.0.0.0/0 | HTTPS |

**Database SG:**
| Port | Source | Description |
|------|--------|-------------|
| 3306 | Web Server SG | MySQL |
| 5432 | Web Server SG | PostgreSQL |
| 27017 | Web Server SG | MongoDB |

---

## 7. IAM Role for EC2

Create an IAM role to give EC2 access to AWS services.

### Create Role (Console)

1. Go to **IAM** → **Roles** → **Create role**
2. Select **AWS service** → **EC2**
3. Attach policies:
   - `AmazonS3ReadOnlyAccess` (for S3 access)
   - `CloudWatchAgentServerPolicy` (for monitoring)
4. Name: `EC2-WebServer-Role`
5. Create role

### Attach to Instance

1. Select EC2 instance
2. **Actions** → **Security** → **Modify IAM role**
3. Select your role
4. **Update IAM role**

---

## 8. User Data (Auto-Configuration)

Run scripts on launch:

```bash
#!/bin/bash
apt update
apt upgrade -y
apt install -y nginx
systemctl enable nginx
systemctl start nginx

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
npm install -g pm2
```

Add in **Advanced Details** → **User data** when launching.

---

## 9. Connect Domain to EC2

### With Route 53

1. Go to **Route 53** → **Hosted zones**
2. Select your domain
3. Create record:
   - Record name: `@` or `www`
   - Type: `A`
   - Value: Your Elastic IP
   - TTL: 300

### With External DNS

Add A record pointing to your Elastic IP:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ | 52.xx.xx.xx | 300 |
| A | www | 52.xx.xx.xx | 300 |

---

## 10. AWS CLI Commands

### Install AWS CLI

```bash
# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure
aws configure
```

### Common Commands

```bash
# List instances
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' --output table

# Start/Stop/Reboot
aws ec2 start-instances --instance-ids i-xxxxxxxxx
aws ec2 stop-instances --instance-ids i-xxxxxxxxx
aws ec2 reboot-instances --instance-ids i-xxxxxxxxx

# Terminate (delete)
aws ec2 terminate-instances --instance-ids i-xxxxxxxxx

# Get instance status
aws ec2 describe-instance-status --instance-id i-xxxxxxxxx
```

---

## 11. Monitoring with CloudWatch

### Enable Detailed Monitoring

```bash
aws ec2 monitor-instances --instance-ids i-xxxxxxxxx
```

### Install CloudWatch Agent

```bash
# Download agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

# Install
sudo dpkg -i amazon-cloudwatch-agent.deb

# Configure
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Start
sudo systemctl start amazon-cloudwatch-agent
sudo systemctl enable amazon-cloudwatch-agent
```

### Key Metrics to Monitor

- CPUUtilization
- NetworkIn/NetworkOut
- DiskReadOps/DiskWriteOps
- StatusCheckFailed

---

## 12. Create AMI (Backup/Template)

### Create AMI from Instance

1. Select instance
2. **Actions** → **Image and templates** → **Create image**
3. Configure:
   - Name: `my-server-backup-20241225`
   - Description: Production server backup
   - No reboot: Optional
4. **Create image**

### AWS CLI

```bash
aws ec2 create-image \
    --instance-id i-xxxxxxxxx \
    --name "my-server-backup-$(date +%Y%m%d)" \
    --description "Automated backup" \
    --no-reboot
```

---

## 13. Auto Scaling (Production)

### Launch Template

```bash
aws ec2 create-launch-template \
    --launch-template-name my-web-server-template \
    --version-description "v1" \
    --launch-template-data '{
        "ImageId": "ami-xxxxxxxxx",
        "InstanceType": "t3.medium",
        "KeyName": "my-server-key",
        "SecurityGroupIds": ["sg-xxxxxxxxx"]
    }'
```

### Auto Scaling Group

Set up via console:
1. Go to **EC2** → **Auto Scaling Groups**
2. Create Auto Scaling group
3. Use launch template
4. Configure:
   - Min: 1
   - Max: 4
   - Desired: 2

---

## 14. Cost Optimization

### Right-Sizing

```bash
# Get instance recommendations
aws compute-optimizer get-ec2-instance-recommendations
```

### Reserved Instances / Savings Plans

- 1-year commitment: ~30% savings
- 3-year commitment: ~50% savings

### Spot Instances

For non-critical, interruptible workloads:
- Up to 90% discount
- Can be terminated with 2-minute warning

```bash
aws ec2 request-spot-instances \
    --instance-count 1 \
    --type "one-time" \
    --launch-specification file://spot-spec.json
```

---

## 15. Troubleshooting

### Can't Connect via SSH

```bash
# Check security group allows SSH from your IP
# Check instance is in "running" state
# Check correct key pair
# Check correct username (ubuntu, ec2-user, admin)
# Check key permissions: chmod 400 key.pem
```

### Instance Won't Start

```bash
# Check instance state
aws ec2 describe-instance-status --instance-id i-xxxxxxxxx

# Check system/instance status checks
# Review CloudWatch logs
```

### Out of Memory

```bash
# Add swap space
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Quick Reference

| Task | Command/Action |
|------|----------------|
| Connect | `ssh -i key.pem ubuntu@ip` |
| Start instance | Console or `aws ec2 start-instances` |
| Stop instance | Console or `aws ec2 stop-instances` |
| Get public IP | `curl http://169.254.169.254/latest/meta-data/public-ipv4` |
| Get instance ID | `curl http://169.254.169.254/latest/meta-data/instance-id` |

---

## Next Steps

1. **[Elastic IP Setup](./aws-elastic-ip-domain.md)** - Static IP configuration
2. **[Security Groups](./aws-security-groups.md)** - Advanced firewall
3. **[RDS Setup](./aws-rds-setup.md)** - Managed database
4. **[S3 Integration](./aws-s3-integration.md)** - Storage setup

---

✅ Your EC2 instance is ready for deployment!
