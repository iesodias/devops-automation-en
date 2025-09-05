## Lab 1: Basic Pipeline with GitHub Actions

### Objective

Create a simple workflow to validate the functioning of GitHub Actions in a new repository.

---

### 1. Create local structure

```bash
mkdir workspace-devops-automation
cd workspace-devops-automation
```

---

### 2. Create GitHub repository

1. Go to [https://github.com](https://github.com)
2. Click "New Repository"
3. Name: `workspace-devops-automation`
4. Leave without README, without .gitignore (we'll configure locally)

---

### 🔗 3. Initialize Git and connect to remote repository

```bash
git init
git remote add origin https://github.com/SEU_USUARIO/workspace-devops-automation.git
echo "# DevOps Automation Project" > README.md
git add .
git commit -m "first commit"
git push -u origin main
```

---

### 4. Create GitHub Actions structure

```bash
mkdir -p .github/workflows
```

---

### 5. Create basic workflow

Create the file `.github/workflows/hello.yml`:

```yaml
name: Hello GitHub Actions

on:
  push:
    branches:
      - main

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Display message in log
        run: echo "✅ GitHub Actions working successfully!"
```

---

### 🚀 6. Push to GitHub

```bash
git add .
git commit -m "add basic GitHub Actions workflow"
git push
```

---

### 7. Check result

1. Go to the repository on GitHub
2. Click on the **Actions** tab
3. Check if the "Hello GitHub Actions" pipeline was executed successfully

---

Done! Now your environment is configured and GitHub Actions is working correctly ✨

# Lab 1: CI/CD for Java Application with Push to Azure Container Registry (ACR)

This lab aims to clone an existing Java project, configure sensitive variables, and run a CI/CD pipeline in GitHub Actions that:

* Compiles the application with Maven
* Runs tests
* Builds the Docker image
* Performs security scan with Trivy
* Performs Azure login
* Pushes the image to Azure Container Registry (ACR)

---

## ✨ Requirements

* Account on [Azure](https://portal.azure.com)
* Azure CLI installed and authenticated
* Permissions to create an **Azure Container Registry**
* Account on [GitHub](https://github.com)
* GitHub project with GitHub Actions pipeline enabled

---

## 🏗️ Create the Azure Container Registry (ACR)

Run the commands below to create a basic ACR:

```bash
# Variables
RESOURCE_GROUP="rg-devops-lab"
REGISTRY_NAME="devopsautomationid"
LOCATION="eastus"

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create the ACR
az acr create --resource-group $RESOURCE_GROUP \
  --name $REGISTRY_NAME \
  --sku Basic \
  --admin-enabled false
```

> 📝 The ACR name must be globally unique and contain only lowercase letters and numbers. It will be accessed as `devopsautomationid.azurecr.io`.

---

## ♻️ Lab Steps

### 1. Clone the original Java project

```bash
# Clone the Java application repository to a temporary folder
git clone https://github.com/iesodias/devops-automation-api.git temp-folder
cd temp-folder

# Copy essential files to the pipeline repository
cp -r pom.xml src Dockerfile ../workspace-devops-automation/
```

### 2. Create GitHub secrets

In the `workspace-devops-automation` repository, go to:

**Settings > Secrets and variables > Actions > New repository secret**

Create the following secrets:

| Name                  | Value                                                                 |
|-----------------------|-----------------------------------------------------------------------|
| `AZURE_CREDENTIALS`   | JSON gerado com `az ad sp create-for-rbac --name "mdcgithubactions" --role contributor --scopes /subscriptions/SUA_SUBSCRIPTION --sdk-auth`                |
| `AZURE_REGISTRY_NAME` | Full name of your ACR (e.g.: `devopsautomationid.azurecr.io`)       |

> ✅ The `AZURE_CREDENTIALS` JSON must include `clientId`, `clientSecret`, `tenantId`, etc.

---

### 3. CI/CD Workflow (.github/workflows/pipeline.yml)

Create or replace the `pipeline.yml` file in the `.github/workflows/` directory with the content below:

```yaml
name: 🚀 Java API CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

env:
  ARTIFACT_NAME: "api-${{ github.sha }}.jar"
  IMAGE_NAME: "java-api:${{ github.sha }}"
  REGISTRY_NAME: ${{ secrets.AZURE_REGISTRY_NAME }}
  REGISTRY_IMAGE: "${{ secrets.AZURE_REGISTRY_NAME }}/java-api:${{ github.sha }}"

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: 🔍 Checkout Code
        uses: actions/checkout@v3

      - name: ☕ Setup Java
        uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: 📦 Build with Maven
        run: mvn clean package -DskipTests

      - name: 🧪 Run Tests
        run: mvn test

      - name: 🛠 Build Docker Image
        run: docker build -t ${{ env.IMAGE_NAME }} .

      - name: 🔐 Trivy Scan (Image)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.IMAGE_NAME }}
          format: table
          exit-code: 0
          severity: HIGH,CRITICAL

      - name: 🔐 Azure Login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: 🔐 Docker Login to ACR
        run: az acr login --name ${{ env.REGISTRY_NAME }}

      - name: 🏷️ Tag Docker Image
        run: docker tag ${{ env.IMAGE_NAME }} ${{ env.REGISTRY_IMAGE }}

      - name: 📤 Push to Azure Container Registry
        run: docker push ${{ env.REGISTRY_IMAGE }}

      - name: 📦 Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: java-api-artifact
          path: target/*.jar

  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: 📥 Download JAR
        uses: actions/download-artifact@v4
        with:
          name: java-api-artifact

      - name: 🚀 Fake Deploy
        run: |
          echo "✅ Build: ${{ env.ARTIFACT_NAME }}"
          echo "📡 Fake deploy to: https://staging.example.com"
````

> ✅ This workflow compiles, tests, scans, pushes to ACR and simulates a deploy to staging.

---

### 4. Execute the pipeline

Given that the project is updated and with the secrets created:

```bash
git add .
git commit -m "setup pipeline ci/cd"
git push origin main
```

GitHub Actions should automatically start the workflow execution.

---

### ⚡ Validation

* Go to GitHub > Actions and check the pipeline execution
* The image should be published in your ACR
* Tests should run successfully
* The Trivy scan should show any vulnerabilities, but without failing
* The "fake" deploy should appear in the log


# Lab – Infrastructure with Terraform + Ansible + GitHub Actions (Optional Destroy)

This lab teaches how to create a VM on Azure with Terraform, configure with Ansible and orchestrate everything with GitHub Actions — including an optional destruction button.

---

## 🔧 What you will learn

- Create infrastructure with Terraform
- Configure VM with Ansible
- Use GitHub Actions with `workflow_dispatch`
- Run `destroy` manually with button

---

## 📁 Project Structure

```
azure-vm-terraform-ansible/
├── infra/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── backend.tf (optional)
├── ansible/
│   └── playbook.yml
├── .github/
│   └── workflows/
│       └── deploy.yml
└── README.md
```

---

## ✅ Prerequisites

- Azure CLI installed and authenticated
- Azure account with permissions
- GitHub repository with Actions enabled
- Secrets created:
  - `AZURE_CREDENTIALS`
  - `ARM_SUBSCRIPTION_ID`
  - `ADMIN_PASSWORD`

---

## 🚀 Lab Steps

### 1. Create the project structure

Run the command below to create the necessary folders and files:

```bash
mkdir -p .github/workflows infra ansible && touch .github/workflows/deploy.yml infra/{main.tf,variables.tf,outputs.tf,terraform.tfvars} ansible/playbook.yml README.md
```

### 2. Access the project folder

```bash
cd workspace-devops-automation
```

---

### 3. Configure remote backend (optional)

```bash
RESOURCE_GROUP="rg-tfstate"
STORAGE_ACCOUNT="tfstatecurso$RANDOM"
CONTAINER_NAME="tfstate"
LOCATION="eastus"

az group create --name $RESOURCE_GROUP --location $LOCATION

az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --encryption-services blob

az storage container create \
  --name $CONTAINER_NAME \
  --account-name $STORAGE_ACCOUNT \
  --auth-mode login
```

---

### 4. Write the Terraform code

Use the `main.tf`, `variables.tf`, `outputs.tf` files in the `infra` folder to define the VM, NSG, IP, etc.

main.tf
```t
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-automation"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "subnet-automation"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "public_ip" {
  name                = "public-ip-vm"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Basic"
}


resource "azurerm_network_interface" "nic" {
  name                = "nic-vm"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.public_ip.id
  }
}

resource "azurerm_network_security_group" "nsg" {
  name                = "nsg-vm"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  security_rule {
    name                       = "SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Swagger"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "8081"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}


resource "azurerm_network_interface_security_group_association" "nsg_assoc" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                            = "vm-automation"
  resource_group_name             = azurerm_resource_group.rg.name
  location                        = azurerm_resource_group.rg.location
  size                            = "Standard_B1s"
  admin_username                  = "azureuser"
  admin_password                  = var.admin_password
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

```

variables.tf
```t
variable "resource_group_name" {
  description = "Resource Group name"
  type        = string
  default     = "rg-vm-automation"
}

variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

variable "admin_password" {
  description = "Administrator user password"
  type        = string
  sensitive   = true
  default     = "Torresmo!@123!(*@)"
}

```

backend.tf
```t
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "tfstatecurso10998"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}

```

outputs.tf
```t
output "public_ip_address" {
  value       = azurerm_public_ip.public_ip.ip_address
  description = "VM public IP address"
}


output "nsg_name" {
  value = azurerm_network_security_group.nsg.name
}

output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}

```

---

### 5. Create the Ansible playbook

In `ansible/playbook.yml`, put what you want to install (e.g.: Docker).


```yaml
---
- name: Install Docker on Ubuntu
  hosts: vm
  become: yes
  tasks:
    - name: Install dependencies
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present
        update_cache: yes

    - name: Add official Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
        state: present

    - name: Update apt cache and install Docker
      apt:
        name: docker-ce
        state: present
        update_cache: yes

    - name: Add user to Docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Start and enable Docker service
      systemd:
        name: docker
        state: started
        enabled: yes
        
    - name: Run Java application container
      docker_container:
        name: java-api
        image: iesodias/java-api:latest
        state: started
        restart_policy: always
        published_ports:
          - "8081:8081"
```

---

### 6. Configure GitHub Actions

In `.github/workflows/deploy.yml`, insert the workflow with `workflow_dispatch` and `destroy` input.

You can run:

- `destroy: false` → creates infra + configures + tests
- `destroy: true` → only destroys infra


```yaml
name: Deploy Terraform to Azure

on:
  workflow_dispatch:
    inputs:
      destroy:
        description: 'Destruir infraestrutura?'
        required: true
        default: 'false'

permissions:
  id-token: write
  contents: read

jobs:
  validate:
    if: ${{ github.event.inputs.destroy == 'false' }}
    name: 🔎 Validar Terraform
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Instalar Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Validar Sintaxe e Plano
        working-directory: infra
        run: |
          export ARM_SUBSCRIPTION_ID=${{ secrets.ARM_SUBSCRIPTION_ID }}
          terraform init
          terraform fmt -check
          terraform validate
          terraform plan -out=tfplan

  deploy:
    if: ${{ github.event.inputs.destroy == 'false' }}
    name: 🚀 Deploy e Configuração
    needs: validate
    runs-on: ubuntu-latest
    outputs:
      vmName: ${{ steps.output_vm.outputs.vm_name }}
      adminUsername: ${{ steps.output_vm.outputs.admin_username }}
      publicIP: ${{ steps.output_vm.outputs.public_ip }}
      nsgName: ${{ steps.output_vm.outputs.nsg_name }}
      resourceGroup: ${{ steps.output_vm.outputs.resource_group }}
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Instalar Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Aplicar Terraform
        working-directory: infra
        run: |
          export ARM_SUBSCRIPTION_ID=${{ secrets.ARM_SUBSCRIPTION_ID }}
          terraform init
          terraform apply -auto-approve

      - name: Capturar Outputs
        id: output_vm
        working-directory: infra
        run: |
          echo "vm_name=vm-automation" >> $GITHUB_OUTPUT
          echo "admin_username=azureuser" >> $GITHUB_OUTPUT
          echo "public_ip=$(terraform output -raw public_ip_address)" >> $GITHUB_OUTPUT
          echo "nsg_name=$(terraform output -raw nsg_name)" >> $GITHUB_OUTPUT
          echo "resource_group=$(terraform output -raw resource_group_name)" >> $GITHUB_OUTPUT

      - name: Definir Variáveis de Ambiente
        run: |
          echo "VM_NAME=vm-automation" >> $GITHUB_ENV
          echo "ADMIN_USERNAME=azureuser" >> $GITHUB_ENV
          echo "PUBLIC_IP=${{ steps.output_vm.outputs.public_ip }}" >> $GITHUB_ENV
          echo "SSH_COMMAND=ssh azureuser@${{ steps.output_vm.outputs.public_ip }}" >> $GITHUB_ENV

      - name: Instalar Ansible e sshpass
        run: |
          sudo apt-get update
          sudo apt-get install -y ansible sshpass

      - name: Criar Inventário Ansible
        run: |
          echo "[vm]" > inventory
          echo "${{ env.PUBLIC_IP }} ansible_user=${{ env.ADMIN_USERNAME }} ansible_password=${{ secrets.ADMIN_PASSWORD }} ansible_ssh_common_args='-o StrictHostKeyChecking=no'" >> inventory

      - name: Executar Playbook Ansible
        run: |
          ansible-playbook -i inventory ansible/playbook.yml --extra-vars "ansible_sudo_pass=${{ secrets.ADMIN_PASSWORD }}"

  post-tests:
    if: ${{ github.event.inputs.destroy == 'false' }}
    name: ✅ Pós-Testes de Infra
    needs: deploy
    runs-on: ubuntu-latest
    env:
      PUBLIC_IP: ${{ needs.deploy.outputs.publicIP }}
      VM_NAME: ${{ needs.deploy.outputs.vmName }}
      NSG_NAME: ${{ needs.deploy.outputs.nsgName }}
      RESOURCE_GROUP: ${{ needs.deploy.outputs.resourceGroup }}
    steps:
      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Testar Swagger na Porta 8081
        run: |
          echo "Aguardando aplicação subir com Swagger..."
          sleep 30
          response=$(curl -s -o /dev/null -w "%{http_code}" http://$PUBLIC_IP:8081/swagger-ui/index.html)
          if [ "$response" != "200" ]; then
            echo "❌ Swagger não respondeu como esperado. Status HTTP: $response"
            exit 1
          else
            echo "✅ Swagger disponível em /swagger-ui/index.html na porta 8081!"
          fi

      - name: Verificar status da VM
        run: |
          status=$(az vm get-instance-view \
            --name "$VM_NAME" \
            --resource-group "$RESOURCE_GROUP" \
            --query "instanceView.statuses[?code=='PowerState/running'].displayStatus" \
            --output tsv)

          echo "Status da VM: $status"

          if [ "$status" != "VM running" ]; then
              echo "❌ A VM não está em execução!"
              exit 1
          else
              echo "✅ VM está rodando com sucesso!"
          fi

      - name: Verificar regra da NSG para porta 8081
        run: |
          result=$(az network nsg rule list \
            --nsg-name "$NSG_NAME" \
            --resource-group "$RESOURCE_GROUP" \
            --query "[?destinationPortRange=='8081' && access=='Allow']")

          if [ "$result" = "[]" ]; then
            echo "❌ Porta 8081 não está liberada na NSG!"
            exit 1
          else
            echo "✅ Porta 8081 está liberada corretamente na NSG!"
          fi

  destroy:
    if: ${{ github.event.inputs.destroy == 'true' }}
    name: 🧨 Destruir Infraestrutura
    runs-on: ubuntu-latest
    env:
      ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Instalar Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Executar Terraform Destroy
        working-directory: infra
        run: |
          terraform init
          terraform destroy -auto-approve

```

### 7. Make commit and push

```bash
git add .
git commit -m "Lab - Terraform + Ansible with Destroy button"
git push origin main
```

## ☁️ Running the Lab

1. Go to the GitHub **Actions** tab
2. Choose the `Deploy Terraform to Azure` workflow
3. Click **Run workflow**
4. Select `destroy = false` to create or `true` to destroy

## ✅ Expected Result

- Infrastructure created automatically
- VM configuration with Ansible
- Swagger available on port 8081
- Simple and secure destruction via GitHub button

## 🧹 Final Tip

Leave the default value of `destroy` as `false` and guide your team to change to `true` **only if you want to destroy everything.**

---
