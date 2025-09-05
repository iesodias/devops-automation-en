# AWS Lab - Creating S3 Bucket with Security Configurations

## Objective  
Create an S3 bucket with:  
- Versioning enabled  
- Public access blocking policy  
- Access logging  

---

## Prerequisites  
- AWS account with S3 permissions  
- [AWS CLI installed](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)  
- Optional: Terraform installed (version at the end)  

---

## Step by Step (AWS Console)  

### 1. Access S3 Console  
1. Log in to [AWS Console](https://console.aws.amazon.com)  
2. Search for **"S3"** and click on the service  

### 2. Create Bucket  
1. Click **"Create bucket"**  
2. Fill in:  
   ```plaintext
   Bucket name: devops-labs-bucket-<your-name> (must be globally unique)  
   Region: us-east-1 (N. Virginia) or choose your region  
   ```

### 3. Configure Options  
```plaintext
✅ Enable versioning  
✅ Block all public access (recommended)  
🔒 Encryption: AES-256 (SSE-S3) - Default  
```

### 4. Access Policy (Example)  
In the "Permissions" tab, paste this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/your-user"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::devops-labs-bucket-<your-name>/*"
    }
  ]
}
```

### 5. Enable Logging (Optional)  
In the "Properties" tab > "Server access logging"  
Specify another bucket for logs (or create a new one)  

### 6. Review and Create  
Click "Create bucket"  

---

## Validation via AWS CLI

### 1. List buckets
```bash
aws s3 ls
```

### 2. Upload test file
```bash
echo "DevOps Lab Test" > teste.txt
aws s3 cp teste.txt s3://devops-labs-bucket-<your-name>/
```

### 3. Check versioning
```bash
aws s3api list-object-versions --bucket devops-labs-bucket-<your-name>
```

---

## Terraform Version (optional)

### 1. Create `s3.tf`
```hcl
resource "aws_s3_bucket" "devops_lab" {
  bucket = "devops-labs-bucket-<your-name>"
  acl    = "private"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }

  tags = {
    Lab = "DevOps-S3"
  }
}
```

### 2. Apply
```bash
terraform init && terraform apply
```

---

## Best Practices

🔐 Always use granular IAM policies (avoid "Action": "s3:*")  
💸 Enable Lifecycle Rules for temporary files  
🛡️ Use Bucket Policies + IAM together for security  

---

## Extra Tip:  
For a **production** bucket, enable:  
- **Object Lock** (compliance)  
- **Cross-Region Replication**  
- **CloudTrail logging**  

---

# AWS S3 Lab with Terraform - Professional Structure

## File Structure
```plaintext
s3-lab/
├── main.tf          # Main resources
├── variables.tf     # Input variables
├── outputs.tf       # Module outputs
├── versions.tf      # Provider versions
└── terraform.tfvars # Variable values (optional)
```

### 1. versions.tf
```hcl
terraform {
  required_version = ">= 1.3.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 2. variables.tf
```hcl
variable "bucket_name" {
  description = "Global unique name for S3 bucket"
  type        = string
}

variable "environment" {
  description = "Environment (dev/stg/prod)"
  type        = string
  default     = "dev"
}

variable "enable_versioning" {
  description = "Enable object versioning"
  type        = bool
  default     = true
}

variable "allowed_ips" {
  description = "List of IPs with bucket access"
  type        = list(string)
  default     = []
}
```

### 3. terraform.tfvars
```hcl
bucket_name       = "devops-labs-bucket-12345" # Replace with your unique name
environment       = "dev"
enable_versioning = true
allowed_ips       = ["189.34.123.45/32"] # Replace with your IP
```

### 4. main.tf
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    Terraform   = "true"
  }
}

resource "aws_s3_bucket_versioning" "this" {
  count = var.enable_versioning ? 1 : 0

  bucket = aws_s3_bucket.this.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_public_access_block" "this" {
  bucket = aws_s3_bucket.this.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_policy" "this" {
  count = length(var.allowed_ips) > 0 ? 1 : 0

  bucket = aws_s3_bucket.this.id
  policy = data.aws_iam_policy_document.bucket_policy[0].json
}

data "aws_iam_policy_document" "bucket_policy" {
  count = length(var.allowed_ips) > 0 ? 1 : 0

  statement {
    principals {
      type        = "AWS"
      identifiers = ["*"]
    }

    effect = "Deny"

    actions = [
      "s3:*",
    ]

    resources = [
      aws_s3_bucket.this.arn,
      "${aws_s3_bucket.this.arn}/*"
    ]

    condition {
      test     = "NotIpAddress"
      variable = "aws:SourceIp"
      values   = var.allowed_ips
    }
  }
}
```

### 5. outputs.tf
```hcl
output "bucket_arn" {
  description = "ARN of the created bucket"
  value       = aws_s3_bucket.this.arn
}

output "bucket_domain_name" {
  description = "Bucket domain name"
  value       = aws_s3_bucket.this.bucket_domain_name
}

output "bucket_regional_domain_name" {
  description = "Regional bucket domain name"
  value       = aws_s3_bucket.this.bucket_regional_domain_name
}
```

---

## How to Execute
```bash
# Initialize
terraform init

# Check plan
terraform plan

# Apply configuration
terraform apply

# Destroy resources (when needed)
terraform destroy
```

---

## Implemented Improvements
- Clear separation of responsibilities by file
- Parameterizable variables with default values
- Condition-based security policy (IP)
- Useful outputs for integration with other systems
- Explicit provider versioning

---

### Usage Tip:
1. For different environments (dev/stg/prod), create separate `.tfvars` files:
   - `dev.tfvars`
   - `production.tfvars`
2. Apply with:
```bash
terraform apply -var-file="production.tfvars"
```

Would you like me to add any specific resource such as:



