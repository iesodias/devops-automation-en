# Prerequisites for DevOps Automation Lab

This document lists the prerequisites needed to follow all modules of the DevOps Automation course. Choose your operating system and follow the corresponding step-by-step instructions to ensure your environment is ready.

---

## 🔧 If you're using **Ubuntu (native or in VM)**

### 1. Update the system and install basic tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl unzip software-properties-common gnupg unzip
```

### 2. Install Git and configure:

```bash
sudo apt install git -y
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git --version
```

### 3. Install Docker:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
sudo docker run hello-world
```

### 4. Install AWS CLI:

To install AWS CLI, you'll need to download the correct package for your system architecture.

**For x86_64 systems (Intel/AMD):**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

**For ARM64 systems:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
```

After downloading the correct file for your architecture, run the following commands to unpack and install:
```bash
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### 5. Install Azure CLI:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
az version
```

### 7. Install Terraform:

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com noble main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform -y
terraform -version
```

### 8. Install Ansible:

```bash
sudo apt install ansible -y
ansible --version
```

### 9. Install kubectl and watch:

To install kubectl, you'll need to download the correct binary for your system architecture. First, get the latest stable version:

```bash
KUBECTL_VERSION=$(curl -Ls https://dl.k8s.io/release/stable.txt)
echo "kubectl stable version: $KUBECTL_VERSION"
```

**For x86_64 systems (Intel/AMD):**
```bash
curl -LO "https://dl.k8s.io/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl"
```

**For ARM64 systems:**
```bash
curl -LO "https://dl.k8s.io/release/$KUBECTL_VERSION/bin/linux/arm64/kubectl"
```

After downloading the correct binary for your architecture, run the following commands to install it and check the version:
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

To install `watch`:
```bash
sudo apt install -y watch
watch --version
```

### 11. Install VS Code:

Download from: [https://code.visualstudio.com/](https://code.visualstudio.com/)
And install the extensions: Docker, Terraform, YAML, GitHub Pull Requests

---

## 💻 If you're using **Windows with WSL 2 (recommended)**

### 1. Install WSL:

```powershell
wsl --install
```

> This automatically installs WSL 2 with Ubuntu (Windows 11).

### 2. If needed, install manually:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

* Restart the computer
* Download the kernel: [https://aka.ms/wsl2kernel](https://aka.ms/wsl2kernel)
* Set WSL 2 as default:

```powershell
wsl --set-default-version 2
```

* Install Ubuntu from Microsoft Store

### 3. Open Ubuntu and follow the same native Ubuntu steps above to install Git, Docker, AWS CLI, etc.

### 4. Install Docker Desktop on Windows:

[https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

> Check the option for WSL integration

### 5. Install Visual Studio Code and the **Remote - WSL** extension

[https://code.visualstudio.com/](https://code.visualstudio.com/)

---

## 🍎 If you're using **macOS**

### 1. Install Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. Install tools:

```bash
brew install git terraform ansible node docker awscli azure-cli
```

### 3. Check versions:

```bash
git --version
terraform -version
ansible --version
docker --version
aws --version
az version
```

### 4. Install Docker Desktop:

[https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

### 5. Install Visual Studio Code:

[https://code.visualstudio.com/](https://code.visualstudio.com/)

### 6. Install extensions:

* Docker
* Terraform
* YAML
* GitHub Pull Requests

---

## With everything ready...

You will be able to:

* Clone GitHub repositories
* Run containers
* Execute CLI commands in AWS and Azure
* Create resources with Terraform and Ansible

Everything is ready to start the DevOps Automation course labs!
