# Lab 1 – Creating a Resource Group with Terraform (Azure)

## 🎯 Objective
Create a Resource Group in Azure using Terraform, step by step.

## 📁 Directory Structure
```bash
mkdir -p ~/labs/terraform/lab1-resource-group
cd ~/labs/terraform/lab1-resource-group
```

## 📄 Step 1 – Create the provider file
```bash
export ARM_SUBSCRIPTION_ID="YOUR_SUBSCRIPTION_HERE"
vi provider.tf
```
Content:
```t
provider "azurerm" {
  features {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.0"
}
```

## 📄 Step 2 – Create the main configuration file
```bash
vi main.tf
```
Content:
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "devops-rg"
  location = "East US"
}
```

## ✅ Step 3 – Initialize Terraform
```bash
terraform init
```

## 🔍 Step 4 – Validate and review the plan
```bash
terraform validate
terraform plan
```

## 🚀 Step 5 – Apply the code and provision
```bash
terraform apply
```
Confirm with `yes` when prompted.

## 🧼 Step 6 – Destroy resources (optional)
```bash
terraform destroy
```

---

# Lab 2 – Creating an Ubuntu VM with Variables and Data Source (Azure)

## 🎯 Objective
Create an Ubuntu virtual machine in Azure using Terraform with variables and fetching the existing resource group via `data`.

## 📁 Directory Structure
```bash
mkdir -p ~/labs/terraform/lab2-vm-variaveis
cd ~/labs/terraform/lab2-vm-variaveis
```

## 📄 Step 1 – Create the provider file
```bash
vi provider.tf
```
Content:
```t
provider "azurerm" {
  features {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.0"
}
```

## 📄 Step 2 – Define variables
```bash
vi variables.tf
```

Content:
```t
variable "location" {
  default = "East US"
}

variable "resource_group_name" {
  default = "devops-rg"
}

variable "admin_username" {
  default = "azureuser"
}

variable "admin_password" {
  default = "SenhaForte123!@#"
}
```

## 📄 Step 3 – Create the main configuration file
```bash
vi main.tf
```
Content:
```t
data "azurerm_resource_group" "rg" {
  name = var.resource_group_name
}

resource "azurerm_public_ip" "public_ip" {
  name                = "devops-public-ip"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Basic"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "devops-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "devops-subnet"
  resource_group_name  = data.azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "nic" {
  name                = "devops-nic"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.public_ip.id
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "devops-vm"
  resource_group_name = data.azurerm_resource_group.rg.name
  location            = var.location
  size                = "Standard_B1s"
  admin_username      = var.admin_username
  admin_password      = var.admin_password
  disable_password_authentication = false
  network_interface_ids = [
    azurerm_network_interface.nic.id
  ]
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts"
    version   = "latest"
  }
}

resource "azurerm_network_security_group" "nsg" {
  name                = "devops-nsg"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "nic_nsg" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}
```

## ✅ Passo 4 – Inicializar o Terraform
```bash
terraform init
```

## 🔍 Passo 5 – Validar e revisar o plano
```bash
terraform validate
terraform plan
```

## 🚀 Passo 6 – Aplicar o código e provisionar
```bash
terraform apply
```
Confirm with `yes` when prompted.

## 🧼 Step 7 – Destroy resources (optional)
```bash
terraform destroy
```

---

# Lab 01 – Creating an AKS with Terraform (Free Tier)

## What We Will Do

- Create an AKS cluster using Terraform
- Use Azure Free Tier
- Authenticate via Azure CLI
- Apply infrastructure with step-by-step commands
- Validate and access the cluster with `kubectl`

---

## Directory Structure

```bash
aks-free-tier/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── keda.tf
├── helm-ingress.tf
```

```bash
mkdir -p ~/labs/terraform/aks-free-tier
cd ~/labs/terraform/aks-free-tier

touch main.tf variables.tf outputs.tf provider.tf keda.tf helm-ingress.tf
```

---

1. Use the curl command to download the kubectl binary:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
This command fetches the latest stable version of kubectl and downloads the appropriate binary for the AMD64 architecture.

Give execute permission to the binary:
```bash
chmod +x kubectl
```
Move the kubectl binary to a directory in your PATH. For example, you can move it to /usr/local/bin/:
```bash
sudo mv kubectl /usr/local/bin/
```
Verify that the installation was successful by running the following command:
```bash
kubectl version --client
```

### Download the Helm installation script:

1. Download a file
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

2. Make the script executable:

```bash
chmod +x get_helm.sh
```
3. Run the installation script to install Helm:

```bash
./get_helm.sh
```
4. Verify the installation by checking the Helm version:

```bash
helm version
```

### How to install AzCli

```bash
sudo apt install azure-cli
```

### Update packages
```bash
sudo apt update && sudo apt upgrade -y
```
###  Install dependencies
```bash
sudo apt install -y gnupg software-properties-common curl
```

###  Add HashiCorp GPG key
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

###  Add official repository
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

###  Update repositories and install Terraform
```bash
sudo apt update
sudo apt install -y terraform
```

# Verify installation
```bash
terraform -v
```

## provider.tf

```t
provider "azurerm" {
  features {}
}

terraform {
  required_version = ">= 1.0.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    kubernetes = {
      source = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
    helm = {
      source = "hashicorp/helm"
      version = "~> 2.0"
    }
  }
}

provider "kubernetes" {
  host                   = azurerm_kubernetes_cluster.aks.kube_config[0].host
  client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_certificate)
  client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_key)
  cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].cluster_ca_certificate)
}

provider "helm" {
  kubernetes {
    host                   = azurerm_kubernetes_cluster.aks.kube_config[0].host
    client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_certificate)
    client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_key)
    cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].cluster_ca_certificate)
  }
}
```

---

## variables.tf

```hcl
variable "resource_group_name" {
  default = "rg-aks-free"
}

variable "location" {
  default = "eastus"
}

variable "aks_cluster_name" {
  default = "aks-free-tier"
}
```

---

## main.tf

```t
data "azurerm_kubernetes_cluster" "aks_data" {
  name                = azurerm_kubernetes_cluster.aks.name
  resource_group_name = azurerm_kubernetes_cluster.aks.resource_group_name
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = var.aks_cluster_name
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "aksfree"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_B2s"  # ideal for free tier
  }

  identity {
    type = "SystemAssigned"
  }

  tags = {
    Environment = "Dev"
  }
}

resource "azurerm_public_ip" "ingress_ip" {
  name                = "myAKSPublicIPForIngress"
  location            = azurerm_resource_group.rg.location
  resource_group_name = data.azurerm_kubernetes_cluster.aks_data.node_resource_group
  allocation_method   = "Static"
  sku                 = "Standard"
}
```

---


## helm_ingress.tf

```t
resource "kubernetes_namespace" "ingress" {
  metadata {
    name = "ingress-nginx"
  }
}

resource "helm_release" "nginx_ingress" {
  name       = "ingress-nginx"
  namespace  = kubernetes_namespace.ingress.metadata[0].name
  repository = "https://kubernetes.github.io/ingress-nginx"
  chart      = "ingress-nginx"
  version    = "4.9.1"

  values = [
    <<EOF
controller:
  replicaCount: 2
  nodeSelector:
    beta.kubernetes.io/os: linux
  service:
    type: LoadBalancer
    externalTrafficPolicy: Local
    loadBalancerIP: "${azurerm_public_ip.ingress_ip.ip_address}"

defaultBackend:
  nodeSelector:
    beta.kubernetes.io/os: linux
EOF
  ]

  depends_on = [azurerm_public_ip.ingress_ip]
}

```

## helm_keda.tf

```t
resource "kubernetes_namespace" "keda" {
  metadata {
    name = "keda"
  }
}

resource "helm_release" "keda" {
  name       = "keda"
  namespace  = kubernetes_namespace.keda.metadata[0].name
  repository = "https://kedacore.github.io/charts"
  chart      = "keda"
  version    = "2.13.2"

  values = [
    <<EOF
replicaCount: 1
EOF
  ]

  depends_on = [kubernetes_namespace.keda]
}

```

## outputs.tf

```t

output "next_steps" {
  value = <<EOT
🎉 AKS Cluster created successfully!

👉 To access your cluster, run the following command:

az aks get-credentials --resource-group ${var.resource_group_name} --name ${var.aks_cluster_name}

🚀 Ingress Controller installed successfully!

✅ Access the fixed IP created:
http://${azurerm_public_ip.ingress_ip.ip_address}

📄 You will see a "404 Not Found" message from NGINX — this is expected!

➡️ Proceed to Lab 03 to create an application + Ingress Resource.

EOT
}

output "resource_group_name" {
  value = var.resource_group_name
}

output "aks_cluster_name" {
  value = var.aks_cluster_name
}
```

---

## Step by Step to Run

### Prerequisites

- Install [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- Install [Terraform](https://developer.hashicorp.com/terraform/downloads)
- Have an Azure account with credits or Free Tier

---

### 1. Login to Azure

```bash
az login
```

---

### 2. Initialize Terraform

```bash
terraform init
```

---

### 3. View the execution plan

```bash
terraform plan
```

---

### 4. Apply the infrastructure

```bash
terraform apply -auto-approve
```

---

### 5. Access the AKS cluster

```bash
az aks get-credentials --resource-group rg-aks-free --name aks-free-tier
kubectl get nodes
```

# Lab 2 - Creating your first Pod in Kubernetes (port 8081)

## Objective

Manually create a Pod with the image `iesodias/java-api:latest`, running on port `8081`, and interact with it via `kubectl`.

---
