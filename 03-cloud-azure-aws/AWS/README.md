# AWS Lab - Creating EC2 Instance (Ubuntu) via Console

## Objective  
Create an EC2 instance with Ubuntu on AWS, configure SSH access and basic security.

 

## Prerequisites  
- Active AWS account  
- Access to [AWS Console](https://console.aws.amazon.com)  

 

## Step by Step

> 🆕 This lab includes: Fixed public IP (Elastic IP) and user-data script for VM initialization with Docker and Nginx pre-installed.  

### 1. Access AWS Console  
1. Log in to [AWS Console](https://console.aws.amazon.com)  
2. In the search bar, type `EC2` and select the service  

### 2. Start Instance Creation  
1. Click `"Launch instance"` (orange button)  

### 3. Basic Configuration  
```plaintext
Instance name: devops-aws-ec2  
Operating system: Ubuntu Server 22.04 LTS (HVM)  
Instance type: t2.micro (Free Tier eligible)  
Key pair: Select existing or create new (.pem)  
```

### 4. Network Configuration  
```plaintext
VPC network: vpc-default (or create new)  
Subnet: Choose an availability zone  
Security group:  
  - Name: devops-aws-sg  
  - Inbound rules:  
    * SSH (22) - My IP  
    * HTTP (80) - Optional  
```

### 5. Storage  
```plaintext
Volume type: gp3 (SSD)  
Size: 8 GB (Free Tier)  
```

### 6. Deployment
```plaintext
Click "Launch instance"
Wait for "Running" status (~2 minutes)
```

 

## 🔹 Configure Elastic IP (Fixed Public IP)

### 1. Allocate Elastic IP
1. In AWS Console, go to "Elastic IPs"
2. Click "Allocate Elastic IP"
3. Confirm allocation

### 2. Associate Elastic IP to instance
1. Select the created IP
2. Click "Actions > Associate"
3. Choose the created instance (devops-aws-ec2)

After this, the IP will be fixed even if the instance is restarted.

 

## 🔹 User Data Script (automatic installation)

During instance creation (in the "Configure instance" tab), add the following script in the **"User data script"** field:

```bash
#!/bin/bash
apt update -y
apt upgrade -y
apt install -y docker.io nginx
systemctl enable docker
systemctl start nginx
```

This script will be executed automatically on first boot, installing Docker, Nginx and activating the service.

 

## SSH Connection

### 1. Find Public IP  
In EC2 console > Instances > copy the Public IPv4

### 2. Connect (Linux/macOS)
```bash
chmod 400 chave-devops.pem  # Give permission to the key
ssh -i "chave-devops.pem" ubuntu@<PUBLIC_IP>
```

### 3. Connect (Windows)  
Use PuTTY or Windows Terminal  
Convert .pem key to .ppk (via PuTTYgen)  

Connect with:
```bash
ssh -i "chave-devops.ppk" ubuntu@<PUBLIC_IP>
```

 

## Useful Commands (Post-Installation)

### Update packages
```bash
sudo apt update && sudo apt upgrade -y
```

### Install basic tools
```bash
sudo apt install -y docker.io nginx
```

 

## Management via AWS CLI

### List instances
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId, State:State.Name, IP:PublicIpAddress}' --output table
```

### Stop instance
```bash
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

### Terminate instance
```bash
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

## Best Practices

Never use security rules with `0.0.0.0/0` for SSH  
💸 Configure billing alerts to avoid unexpected costs  

## Key features of this version:
- Focus on Ubuntu + Free Tier
- Commands for Windows/Linux/macOS
- Critical security explanation
- 100% Markdown compatible formatting

 