# AWS Lab – Creating and Manipulating S3 with AWS CLI

This lab teaches how to create and manipulate Amazon S3 buckets using only the AWS CLI, without graphical interface. Ideal for those learning automation and cloud command line.

---

## Prerequisites

* AWS CLI installed and configured (`aws configure`)
* S3 access permissions

---

## 1. Create an S3 bucket

```bash
aws s3api create-bucket --bucket meu-lab-s3 --region us-east-1
```

> Creates a bucket called `meu-lab-s3` in the `us-east-1` region.
> Warning: the bucket name must be globally unique.

---

## 2. List all S3 buckets in the account

```bash
aws s3api list-buckets --query "Buckets[].Name"
```

> Lists the names of all buckets created in your AWS account.

---

## 3. Send a file to the bucket

```bash
echo "test" > arquivo.txt
aws s3 cp arquivo.txt s3://meu-lab-s3/
```

> Creates a local file and sends it to the bucket.

---

## 4. List files inside the bucket

```bash
aws s3 ls s3://meu-lab-s3/
```

> Shows all objects saved in the bucket.

---

## 5. Download a file from the bucket

```bash
aws s3 cp s3://meu-lab-s3/arquivo.txt arquivo-download.txt
```

> Downloads a specific object from the bucket to your computer.

---

## 6. Create a folder (prefix) in the bucket

```bash
aws s3api put-object --bucket meu-lab-s3 --key pasta1/
```

> Creates a logical folder in the bucket.
> In S3,
## Lab 10 – Creating and Manipulating VMs on AWS via CLI

### 🎙️ Initial introduction

In this lab we will learn to create a virtual machine on AWS using only the command line
We will use the default VPC and variables to automate each step of the process
This type of automation is essential for those who want to integrate infrastructure creation in DevOps pipelines

---

### 1. Objective

Create and manage EC2 instances on AWS using the command line, leveraging default VPC and subnets, with variable usage for reusability.

---

### 2. Prerequisites

* AWS CLI installed and configured (`aws configure`)
* Permissions for EC2, VPC and IAM

---

### 3. Initial environment variables

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text)
```

> `--filters Name=isDefault,Values=true`: filters VPCs marked as default in the account
> `--query "Vpcs[0].VpcId"`: returns the ID of the first default VPC
> `--output text`: removes JSON formatting, returning only pure text

```bash
SUBNET_ID=$(aws ec2 describe-subnets \
  --filters Name=default-for-az,Values=true \
  --query "Subnets[0].SubnetId" \
  --output text)
```

> `--filters Name=default-for-az,Values=true`: filters default subnets of the availability zone
> `--query "Subnets[0].SubnetId"`: returns the ID of the first available subnet

```bash
KEY_NAME="devops-keypair"
SG_NAME="devops-sg-ie"
```

> Defines reference names for SSH key and security group

---

### 4. Create key pair for access

```bash
aws ec2 create-key-pair \
  --key-name "$KEY_NAME" \
  --query 'KeyMaterial' \
  --output text > "$KEY_NAME.pem"
```

> `--key-name`: name of the key to be created
> `--query 'KeyMaterial'`: extracts only the private key content
> `--output text`: avoids JSON
> `> "$KEY_NAME.pem"`: saves the content to local PEM file

```bash
chmod 400 "$KEY_NAME.pem"
```

> Sets secure permission for the file (read only by owner)

---

### 5. Create security group

```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name "$SG_NAME" \
  --description "SSH Access" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' \
  --output text)
```

> Creates a security group and captures its ID with the `--query 'GroupId'` option

---

### 6. Enable SSH access

```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

> `--group-id`: ID of the group to be modified
> `--protocol tcp`: defines the protocol
> `--port 22`: opens port 22 (SSH)
> `--cidr 0.0.0.0/0`: allows access from any IP (not recommended for production)

---

### 7. Get ID of an Amazon Linux 2 AMI

```bash
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text)
```

> `--owners amazon`: filters only official Amazon AMIs
> `--filters`: filters available Amazon Linux 2 AMIs
> `sort_by(...) | [-1]`: sorts by creation date and picks the latest one
> `--output text`: returns only the ID

---

### 8. Create EC2 instance

```bash
INSTANCE_INFO=$(aws ec2 run-instances \
  --image-id "$AMI_ID" \
  --count 1 \
  --instance-type t2.micro \
  --key-name "$KEY_NAME" \
  --security-group-ids "$SG_ID" \
  --subnet-id "$SUBNET_ID" \
  --associate-public-ip-address \
  --query 'Instances[0].[InstanceId, PublicIpAddress]' \
  --output text)
```

> Creates the instance with public IP and captures ID and IP via query

```bash
INSTANCE_ID=$(echo "$INSTANCE_INFO" | awk '{print $1}')
PUBLIC_IP=$(echo "$INSTANCE_INFO" | awk '{print $2}')
```

> Extracts separated values from the response

```bash
echo "Instance created: $INSTANCE_ID"
echo "Public IP: $PUBLIC_IP"
```

> Displays the data in the terminal

---

### 9. Connect via SSH

```bash
ssh -i "$KEY_NAME.pem" ec2-user@"$PUBLIC_IP"
```

> Connects to the EC2 instance using the private key

---

### 10. Stop the instance

```bash
aws ec2 stop-instances --instance-ids "$INSTANCE_ID"
```

> Sends stop command to the instance

---

### 11. Start again

```bash
aws ec2 start-instances --instance-ids "$INSTANCE_ID"
```

> Starts the instance again

---

### 12. Terminate and clean up resources

```bash
aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
```

> Deletes the EC2 instance

```bash
aws ec2 delete-security-group --group-id "$SG_ID"
```

> Removes the created security group

```bash
rm "$KEY_NAME.pem"
```

> Removes the local key from the system

---


## Lab 11 – Creating VMs in Batch with Bash Script on AWS CLI

### 1. Objective

Automate the creation of multiple EC2 instances using a shell script with loops and variables.

---

### 2. Prerequisites

* AWS CLI configured (`aws configure`)
* EC2, VPC, subnets and security groups permissions

---

### 3. Create project directory

```bash
mkdir -p ~/labs/aws/vm-lote # IF YOU HAVE DELETED
cd ~/labs/aws/vm-lote
touch criar-vms-aws.sh
chmod +x criar-vms-aws.sh
```

---

### 4. Content of `criar-vms.sh` script

```bash
## Lab 11 – Criando VMs em Lote com Script Bash na AWS CLI

### 1. Objetivo

Automatizar a criação de múltiplas instâncias EC2 usando um script em shell com loops e variáveis.

---

### 2. Pré-requisitos

* AWS CLI configurada (`aws configure`)
* Permissões de EC2, VPC, subnets e security groups

---

### 3. Criar diretório do projeto

```bash
mkdir -p ~/labs/aws/vm-lote
cd ~/labs/aws/vm-lote
touch criar-vms.sh
chmod +x criar-vms.sh
```

---

### 4. Conteúdo do script `criar-vms.sh`

```bash
#!/bin/bash

KEY_NAME="devops-keypair"
SG_NAME="devops-sg-ie"

VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" --output text)

SUBNET_ID=$(aws ec2 describe-subnets \
  --filters Name=default-for-az,Values=true \
  --query "Subnets[0].SubnetId" --output text)

AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text)

# Check if keypair already exists in AWS
aws ec2 describe-key-pairs --key-names "$KEY_NAME" >/dev/null 2>&1

if [ $? -ne 0 ]; then
  echo "Creating keypair $KEY_NAME"
  aws ec2 create-key-pair --key-name "$KEY_NAME" \
    --query 'KeyMaterial' --output text > "$KEY_NAME.pem"
  chmod 400 "$KEY_NAME.pem"
else
  echo "Keypair $KEY_NAME already exists in AWS. Skipping creation."
  if [ ! -f "$KEY_NAME.pem" ]; then
    echo "⚠️ Local file $KEY_NAME.pem does not exist. Create manually or download from original creation."
    exit 1
  fi
fi

# Create security group
SG_ID=$(aws ec2 create-security-group \
  --group-name "$SG_NAME" \
  --description "Access via loop" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)

# Open port 22
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" --protocol tcp --port 22 --cidr 0.0.0.0/0

# List of names
VMS=(vm01 vm02 vm03)

for NAME in "${VMS[@]}"; do
  echo "Creating instance: $NAME"

  INSTANCE_ID=$(aws ec2 run-instances \
    --image-id "$AMI_ID" \
    --count 1 \
    --instance-type t2.micro \
    --key-name "$KEY_NAME" \
    --security-group-ids "$SG_ID" \
    --subnet-id "$SUBNET_ID" \
    --associate-public-ip-address \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$NAME}]" \
    --query 'Instances[0].InstanceId' --output text)

  echo "$NAME created with ID: $INSTANCE_ID"

  # Wait for instance to be ready
  aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"
  echo "$NAME is running"
done
```

---

### 5. Execute the script

```bash
./criar-vms.sh
```

---

### 6. Validate the instances

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=vm01,vm02,vm03" \
  --query "Reservations[].Instances[].PublicIpAddress" --output text
```

---

### 7. Final words

Very good, with this script we can create multiple VMs with customized names
This shows how loops and variables help in large-scale automation
See you in the next class

```

---

### 5. Execute the script

```bash
./criar-vms.sh
```

---

### 6. Validate the instances

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=vm01,vm02,vm03" \
  --query "Reservations[].Instances[].PublicIpAddress" --output text
```

---

## Lab 12 – Script to Destroy VMs and Resources on AWS via CLI

### 1. Objective

Remove EC2 instances created in batch as well as associated resources such as security group and access key

---

### 2. Expected structure

This script assumes that VMs were created with names like `vm01`, `vm02`, `vm03`, and that resources were named as follows:

```bash
KEY_NAME="devops-keypair"
SG_NAME="devops-sg-ie"
```

---

### 3. Create the destruction script

```bash
mkdir -p ~/labs/aws/vm-lote
cd ~/labs/aws/vm-lote
touch destruir-vms.sh
chmod +x destruir-vms.sh
```

---

### 4. Content of `destruir-vms.sh`

```bash
#!/bin/bash

export AWS_PAGER=""

KEY_NAME="devops-keypair-01"
SG_NAME="devops-sg-ie"

# List of created names
VMS=(vm01 vm02 vm03)

# Identify the Instance IDs
INSTANCE_IDS=()

for NAME in "${VMS[@]}"; do
  INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$NAME" "Name=instance-state-name,Values=running,stopped" \
    --query "Reservations[*].Instances[*].InstanceId" --output text)

  if [ -n "$INSTANCE_ID" ]; then
    echo "Terminating $NAME with ID $INSTANCE_ID"
    aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
    INSTANCE_IDS+=($INSTANCE_ID)
  else
    echo "Instance $NAME not found or already terminated"
  fi
done

# Wait for termination of ALL collected instances
if [ ${#INSTANCE_IDS[@]} -gt 0 ]; then
  echo "Waiting for instances to terminate..."
  aws ec2 wait instance-terminated --instance-ids "${INSTANCE_IDS[@]}"
  echo "Instances terminated successfully"
else
  echo "No instances to wait for termination"
fi

# Delete security group with verification
SG_ID=$(aws ec2 describe-security-groups \
  --group-names "$SG_NAME" \
  --query 'SecurityGroups[0].GroupId' --output text 2>/dev/null)

if [ -n "$SG_ID" ]; then
  echo "Waiting for Security Group release..."
  sleep 10  # small pause to ensure release
  aws ec2 delete-security-group --group-id "$SG_ID"
  echo "Security group removed successfully"
else
  echo "Security group $SG_NAME not found or already removed"
fi

# Delete remote and local key
aws ec2 delete-key-pair --key-name "$KEY_NAME"
rm -f "$KEY_NAME.pem"
echo "Keypair and local file removed"

```

---

### 5. Execute the script

```bash
./destruir-vms.sh
```

---

### 6. Final words

Done successfully, we destroyed all VMs in batch and cleaned up AWS resources
This type of script is essential to avoid costs and keep the environment clean
See you in the next class
