## Ansible Labs Organization – Unified Structure

### Objective:

Consolidate all Ansible labs into a single structure within the `~/ansible-lab/` directory, using a single inventory and optional subfolders for playbook organization.

---

### Suggested Directory Structure:

```bash
~/ansible-lab/
├── hosts                    # Single inventory
├── playbooks/              # Playbooks organized by lab
│   ├── setup-basico.yaml
│   └── other future ones...
```

---

## Lab Ansible 01 – Creating VMs on Azure

### Objective:

Create two Ubuntu 20.04 VMs on Azure to serve as controller and target for automation with Ansible

---

### Steps:

#### 1. Create Resource Group

```bash
az group create \
  --name ansible-lab-rg \
  --location eastus
```

---

#### 2. Create the controller VM

```bash
az vm create \
  --name ansible-controller \
  --resource-group ansible-lab-rg \
  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest \
  --admin-username azureuser \
  --admin-password 'SenhaForte123!@#' \
  --authentication-type password \
  --public-ip-sku Basic
```

---

#### 3. Create the target VM

```bash
az vm create \
  --name ansible-target \
  --resource-group ansible-lab-rg \
  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest \
  --admin-username azureuser \
  --admin-password 'SenhaForte123!@#' \
  --authentication-type password \
  --public-ip-sku Basic
```

---

#### 4. Open port 22 (SSH) on both VMs

```bash
az vm open-port --resource-group ansible-lab-rg --name ansible-controller --port 22
az vm open-port --resource-group ansible-lab-rg --name ansible-target --port 22
```

---

#### (Optional) Expose application port (e.g.: 8081)

```bash
az network nsg rule create \
  --resource-group ansible-lab-rg \
  --nsg-name ansible-targetNSG \
  --name Allow-8081 \
  --priority 1002 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 8081
```

---

### Expected result:

* Two VMs created on Azure (controller and target)
* SSH access enabled on both machines
* Environment ready for installation and testing with Ansible

---

## Lab Ansible 02 – Install Ansible and test connection

### Objective:

Install Ansible and sshpass on the controller, configure the inventory and test the connection with the target

---

### Steps:

#### 1. Connect to the controller VM and organize structure

```bash
az vm list-ip-addresses \
  --resource-group ansible-lab-rg \
  --name ansible-controller \
  --query "[].virtualMachine.network.publicIpAddresses[].ipAddress" \
  -o tsv

az vm list-ip-addresses \
  --resource-group ansible-lab-rg \
  --name ansible-target \
  --query "[].virtualMachine.network.publicIpAddresses[].ipAddress" \
  -o tsv

ssh azureuser@<IP_DA_CONTROLLER>
mkdir -p ~/ansible-lab/playbooks
cd ~/ansible-lab
```

---

#### 2. Install dependencies

```bash
sudo apt update
sudo apt install -y ansible sshpass
```

---

#### 3. Export temporary environment variable

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

> This variable disables SSH key checking
> Validity: only in the current session

---

#### 4. Create the inventory file

```bash
vi ~/ansible-lab/hosts
```

Content:

```ini
[targets]
vmtarget ansible_host=<IP_DA_TARGET> ansible_user=azureuser ansible_password="SenhaForte123!@#" ansible_python_interpreter=/usr/bin/python3
```

---

#### 5. Test connection with ping module

```bash
ansible -i hosts targets -m ping
```

---

### Expected result:

* Successful connection with the target
* `pong` response from Ansible

---

## Lab Ansible 03 – Creating and executing your first playbook

### Objective:

Create a playbook to update packages and install utilities on the target machine

---

### Steps:

#### 1. Create playbook file

```bash
cd ~/ansible-lab/playbooks
vi setup-basico.yaml
```

Content:

```yaml
---
- name: Configure VM with basic packages
  hosts: targets
  become: true

  tasks:
    - name: Update package list
      apt:
        update_cache: yes

    - name: Install useful packages
      apt:
        name:
          - htop
          - curl
          - git
        state: present
```

---

#### 2. Check environment variable

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

---

#### 3. Execute the playbook

```bash
cd ~/ansible-lab
ansible-playbook -i hosts playbooks/setup-basico.yaml
```

---

#### 4. Validate results

```bash
ansible -i hosts targets -m shell -a "which htop"
ansible -i hosts targets -m shell -a "htop --version"
```

---

### Expected result:

* Packages updated and installed successfully
* First execution: `changed: true`
* Second execution: `ok: true`

Functional environment for automation with Ansible

## Lab Ansible 04 – Installing Docker and running a Java container

### Objective:

Install Docker and run the `iesodias/java-api:latest` application on the target machine using Ansible

---

### Steps:

#### 1. Create the playbook file

```bash
cd ~/ansible-lab/playbooks
nano docker-api.yaml
```

Content:

```yaml
---
- name: Install Docker and run Java application
  hosts: targets
  become: true

  tasks:
    - name: Update packages
      apt:
        update_cache: yes

    - name: Install Docker dependencies
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present

    - name: Add Docker GPG key
      shell: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

    - name: Add Docker repository
      shell: add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

    - name: Update packages again
      apt:
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    - name: Enable and start Docker
      systemd:
        name: docker
        enabled: yes
        state: started

    - name: Install pip for Python 3
      apt:
        name: python3-pip
        state: present

    - name: Install Docker SDK for Python
      pip:
        name: docker

    - name: Add user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Run container with Java application
      docker_container:
        name: java-api
        image: iesodias/java-api:latest
        state: started
        restart_policy: always
        ports:
          - "8081:8081"
```

---

#### 2. Check environment variable

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

---

#### 3. Execute the playbook

```bash
cd ~/ansible-lab
ansible-playbook -i hosts playbooks/docker-api.yaml
```

---

#### 4. Validate result

```bash
ansible -i hosts targets -m shell -a "docker ps"
```

You should see the `iesodias/java-api:latest` image running

---

### Expected result:

* Docker correctly installed on target VM
* `java-api` container running on port 8081
* Application accessible via target VM public IP


## Lab Ansible 06 – Manage user, permissions and folder structure

### Objective:

Create a new user (e.g.: devops), grant administrative permissions and configure specific directories with the correct permissions

---

### Steps:

#### 1. Create the playbook file

```bash
cd ~/ansible-lab/playbooks
nano usuario-devops.yaml
```

#### Playbook content:

```yaml
---
- name: Create user and manage permissions
  hosts: targets
  become: true

  tasks:
    - name: Create devops user with password and bash
      user:
        name: devops
        password: "{{ 'Devops123!@#' | password_hash('sha512') }}"
        shell: /bin/bash
        groups: sudo,docker
        append: yes
        state: present

    - name: Create application directory
      file:
        path: /opt/app
        state: directory
        owner: devops
        group: devops
        mode: '0755'

    - name: Create logs directory
      file:
        path: /opt/logs
        state: directory
        owner: devops
        group: devops
        mode: '0755'

    - name: Create test file in app directory
      copy:
        content: "Application started on {{ ansible_date_time.date }} {{ ansible_date_time.time }}\n"
        dest: /opt/app/status.txt
        owner: devops
        group: devops
        mode: '0644'

    - name: Create simulated log file
      shell: echo "log started on $(date)" > /opt/logs/inicial.log
```

---

### Execute the playbook:

```bash
cd ~/ansible-lab
ansible-playbook -i hosts playbooks/usuario-devops.yaml
```

---

### Validate result:

```bash
ansible -i hosts targets -m shell -a "ls -l /opt"
ansible -i hosts targets -m shell -a "cat /opt/app/status.txt"
ansible -i hosts targets -m shell -a "id devops"
```

---

### Expected result:

* User `devops` created with password and bash shell
* `sudo` permissions and access to `docker`
* `/opt/app` and `/opt/logs` structure created with correct owner
* Files `status.txt` and `inicial.log` generated and filled



## 💣 Final Lab – Destroy Ansible environment on Azure

### 🌟 Objective:

Remove all VMs, IPs, disks and resources created in the Ansible lab on Azure

---

### 🛠️ Steps:

#### 🔎 1. Check created resources (optional)

```bash
az resource list --resource-group ansible-lab-rg --output table
```

> This shows all resources within the `ansible-lab-rg` group, including VMs, IPs, disks and more

---

#### 🗑️ 2. Delete the Resource Group (and everything inside it)

```bash
az group delete --name ansible-lab-rg --yes --no-wait
```

> Explaining:
>
> * `--yes`: automatically confirms deletion
> * `--no-wait`: runs the process in the background, without blocking the terminal

---

### ✅ Expected result:

* VMs `ansible-controller` and `ansible-target` are removed
* Public IPs, disks, NICs, NSGs and VNet are also deleted
* No lab resources remain on Azure