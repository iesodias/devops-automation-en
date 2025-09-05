# 🧪 Lab - Installing Datadog Agent on an EC2 Instance in AWS

## 🎯 Objective
Provision an EC2 instance in AWS and install the Datadog agent to collect basic infrastructure metrics.

---

## ✅ Prerequisites
- AWS account with EC2 permissions
- AWS CLI configured (`aws configure`)
- Key pair created (`devops-automation.pem`)
- Datadog account with active trial plan
- AWS integration connected in Datadog portal (Integrations > AWS)

---

## 🚀 Step 1 - Create Security Group with necessary ports

```bash
aws ec2 create-security-group \
  --group-name datadog-sg \
  --description "Security group for Datadog agent"

aws ec2 authorize-security-group-ingress \
  --group-name datadog-sg \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-name datadog-sg \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

---

## 🖥️ Step 2 - Create an EC2 Amazon Linux 2 instance

```bash
aws ec2 run-instances \
  --image-id ami-0c101f26f147fa7fd \
  --instance-type t2.micro \
  --key-name devops-automation \
  --security-groups datadog-sg \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datadog-ec2}]' \
  --count 1
```

---

## 🔍 Step 3 - Get the instance public IP

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=datadog-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]" \
--output table
```

---

## 🔐 Step 4 - Access the instance via SSH

```bash
chmod 400 devops-automation.pem
ssh -i "devops-automation.pem" ec2-user@SEU_IP_PUBLICO
```

---

## 🐶 Step 5 - Install Datadog agent

1. Copy your API Key from the portal: `Integrations > APIs`

2. Run on the EC2 instance:

```bash
DD_API_KEY="SUA_API_KEY" \
DD_SITE="datadoghq.com" \
bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

---

## 📈 Step 6 - Validate agent status

```bash
sudo datadog-agent status
```

You should see various metrics collected, such as CPU, memory, disk, etc.

---

## 🧭 Step 7 - Check in Datadog portal

1. Access: https://app.datadoghq.com/
2. Go to **Infrastructure > Host Map**
3. Confirm that the host appears by name or ID
4. View ready-made dashboards like **EC2 Overview** or **Host Overview**

---

## 🧹 Step 8 - Clean up environment (optional)

```bash
# List instances and capture Instance ID
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datadog-ec2" \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text

# Terminate the instance
aws ec2 terminate-instances --instance-ids i-xxxxxxxxxxxxxxxxx

# Delete the Security Group
aws ec2 delete-security-group --group-name datadog-sg
```

---

## ✅ Done
Your EC2 has been configured with the Datadog agent and AWS integration is working.
You can now create dashboards, monitors and explore observability resources comprehensively.


