# Lab 1 – Creating Ubuntu VM on Azure via Portal (DevOps)

## Objective
Create an Ubuntu 22.04 LTS VM on Azure via portal with configurations focused on DevOps.

---

## Step by Step

### 1. Access Azure Portal
```plaintext
1. Access https://portal.azure.com
2. Click "Create a resource"
```

### 2. Select Ubuntu Image
```plaintext
1. Search for "Virtual Machine"
2. Select:
   - Image: Ubuntu Server 22.04 LTS - Gen2
```

### 3. Basic Configuration
```plaintext
Resource group: devops-ubuntu-rg (new)
VM name: devops-ubuntu-vm
Region: Brazil South
Size: Standard_B2s (2 vCPUs, 4 GiB RAM)
Authentication: Password
User: devopsadmin
Password: [Set a strong password!]
```

### 4. Network Configuration
```plaintext
Virtual network: devops-ubuntu-vnet (new)
Public IP: devops-ubuntu-ip (new)
Open ports: SSH (22)
```

### 5. Deployment
```plaintext
Click "Review + create"
After validation, click "Create"
```

---

## SSH Connection

### Using Azure CLI to get the IP
```bash
ssh devopsadmin@$(az vm show -d -g devops-ubuntu-rg -n devops-ubuntu-vm --query publicIps -o tsv)
```

### Manual Alternative
```bash
ssh devopsadmin@<PUBLIC_IP>
```

Get the VM's public IP directly from the portal if needed.

---

## Useful Azure CLI Commands

### List VMs
```bash
az vm list -g devops-ubuntu-rg -o table
```

### Stop the VM
```bash
az vm stop -g devops-ubuntu-rg -n devops-ubuntu-vm
```

### Delete all resources
```bash
az group delete -n devops-ubuntu-rg --yes
```

---

## Security Tips

- Use `ssh-keygen` and public key authentication whenever possible
- Restrict the NSG to accept connections only from your IP
- Always stop the VM when not in use to avoid charges

---

This lab helps you understand how to provision and manage a VM on Azure with a focus on DevOps. We can continue with automation labs using CLI, Bicep or Terraform if you want.

