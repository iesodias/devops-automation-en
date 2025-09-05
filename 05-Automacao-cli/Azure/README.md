# 🧪 Lab 0.1 – Introduction to Azure CLI

## 🎯 Objective

Learn the first Azure CLI commands for authentication and practical resource querying (even when no resources are provisioned yet).

---

## 🛠️ Prerequisites

* Have an Azure account
* Have Azure CLI installed ([official link](https://learn.microsoft.com/cli/azure/install-azure-cli))

---

## 🚀 Step by Step

### 🔐 Step 1: Login to Azure

```bash
az login
```

### 📋 Step 2: List available subscriptions

```bash
az account list --output table
```

* Shows the subscriptions your account has access to.
* `--output table` makes the result more readable.

---

### 📦 Step 3: List existing Resource Groups

```bash
az group list --output table
```

* Shows all Resource Groups in the current subscription.
* In your case, groups like these should appear:

  * `terraform-demo-rg`
  * `cloud-shell-storage-eastus`
  * `NetworkWatcherRG`
  * `DefaultResourceGroup-EUS`
  * `rg-checkov-lab`
  * `rg-tfstate`

> **Warning**: the `az group list` command **does not accept the `--resource-group` parameter**. To query a specific group, use `az group show`.

Example:

```bash
az group show --name terraform-demo-rg --output table
```

---

### 🔍 Step 4: List resources from an existing group (even if empty)

```bash
az resource list --resource-group terraform-demo-rg --output table
```

* Shows the resources within the `terraform-demo-rg` group.
* If there are no resources yet, the return will be an empty list.

You can also test with other existing groups:

```bash
az resource list --resource-group rg-checkov-lab --output table
az resource list --resource-group rg-tfstate --output table
```

---

### 📁 Step 5: Test VM listing (even if none exist)

```bash
az vm list --output table
```

* Even without created VMs, this command confirms that the CLI is working correctly.

> Example with filter:

```bash
az vm list --query '[].{name:name, location:location}' --output table
```

---

## ✅ Conclusion

Now you are authenticated, viewed your real subscriptions and resource groups, and tested commands to query resources and VMs. Even without provisioned infrastructure, you already have a solid foundation to continue with the next labs. Let's get to practice! 💪

# 🧪 Lab 2 – Create VMs in batch via Bash + Azure CLI (Baby Steps)

## 🎯 Objective

Create multiple virtual machines in Azure first using manual commands and then automate with Bash + Azure CLI.

---

## 🛠️ Prerequisites

* Azure CLI installed
* Already authenticated via `az login`
* An existing Resource Group (e.g.: `terraform-demo-rg`)

---

## 🚶 Stage 1 – Manual commands to create VMs

### 📌 Define variables in terminal:

```bash
az group create --name devops-automation --location eastus
```

```bash
RG="devops-automation"
LOCATION="eastus"
PASSWORD="StrongPassword123"
```

### 💻 Create the first VM manually:

```bash
az vm create \
  --resource-group $RG \
  --name vm01 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --admin-password $PASSWORD \
  --authentication-type password \
  --location $LOCATION
```

### 💻 Create the second VM:

```bash
az vm create \
  --resource-group $RG \
  --name vm02 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --admin-password $PASSWORD \
  --authentication-type password \
  --location $LOCATION
```

### 📄 Check the created VMs:

```bash
az vm list --resource-group $RG --output table
```

---

## 🔍 Stage 1.1 – Explore created VMs properties

### 1️⃣ List public IPs

```bash
az vm list-ip-addresses --resource-group $RG --output table
```

### 2️⃣ View VM `vm01` details

```bash
az vm show --resource-group $RG --name vm01 --output json
```

### 3️⃣ Get only the status of `vm01`

```bash
az vm get-instance-view --resource-group $RG --name vm01 --query "instanceView.statuses[*].displayStatus" --output table
```

### 4️⃣ View disk information for `vm01`

```bash
az disk list --resource-group $RG --output table
```

### 5️⃣ View NICs associated with VMs

```bash
az network nic list --resource-group $RG --output table
```

### 6️⃣ View network security group (NSG)

```bash
az network nsg list --resource-group $RG --output table
```

### 7️⃣ Get public IP directly (example with `jq` if installed)

```bash
az vm list-ip-addresses --resource-group $RG --name vm01 --query "[].virtualMachine.network.publicIpAddresses[].ipAddress" --output tsv
```

---

## 🗑️ Stage 1.2 – Delete VMs before recreating

Before proceeding to automation, let's remove the VMs we created manually:

### 🧼 Command to delete `vm01`:

```bash
az vm delete --resource-group $RG --name vm01 --yes
```

### 🧼 Command to delete `vm02`:

```bash
az vm delete --resource-group $RG --name vm02 --yes
```

You can verify if the VMs were removed with:

```bash
az vm list --resource-group $RG --output table
```

---

## 🤖 Stage 2 – Transform into automated script

### 📂 Create structure and script:

```bash
mkdir -p ~/labs/azure/vm-lote
cd ~/labs/azure/vm-lote
touch criar-vms.sh
chmod +x criar-vms.sh
```

### ✍️ Content of `criar-vms.sh`:

```bash
#!/bin/bash

RESOURCE_GROUP="devops-automation"
LOCATION="brazilsouth"
PASSWORD="StrongPassword123!"

criar_vm() {
  local NOME_VM=$1
  echo "Creating VM: $NOME_VM"

  az vm create \
    --resource-group $RESOURCE_GROUP \
    --name $NOME_VM \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --admin-password $PASSWORD \
    --authentication-type password \
    --location $LOCATION \
    --no-wait
}

# List of VMs to create
VMS=(vm01 vm02)

for nome in "${VMS[@]}"; do
  criar_vm $nome
done

echo "Batch creation started. Wait for VMs to be provisioned."
```

### ▶️ Run the script:

```bash
./criar-vms.sh
```

# 🧪 Lab 3 – Filter and save information with Azure CLI + jq/awk

## 🎯 Objective

Learn to use command-line commands to filter and save useful VM data — such as name, IP and status — simulating what we would do in pipelines like GitHub Actions (but here entirely via terminal).

---

## 🛠️ Prerequisites

* VMs already created (e.g.: `vm01` and `vm02`)
* Azure CLI installed
* Tools like `jq` and `awk` available in terminal

---

## 🚶 Stage 1 – List and save public IPs

### 🧾 How raw output looks before filtering

If you run the command below:

```bash
az vm list-ip-addresses --resource-group devops-automation --output json
```

The output will be similar to:

```json
[
  {
    "virtualMachine": {
      "name": "vm01",
      "network": {
        "publicIpAddresses": [
          {
            "ipAddress": "20.120.45.101"
          }
        ]
      }
    }
  },
  {
    "virtualMachine": {
      "name": "vm02",
      "network": {
        "publicIpAddresses": [
          {
            "ipAddress": "20.120.45.102"
          }
        ]
      }
    }
  }
]
```

This structure will be filtered with `jq` in the next step.

### 🔍 Get IPs with Azure CLI + jq:

This command filters only the VM name and public IP from the previous JSON structure, making the output lean and easy to manipulate with other tools.

```bash
az vm list-ip-addresses --resource-group devops-automation \
  --query "[].virtualMachine.{name:name, ip:network.publicIpAddresses[0].ipAddress}" \
  -o json | jq -r '.[] | "\(.name) \(.ip)"'
```

### 💾 Save IPs to a file:

Here we are redirecting the filtered output to the `vms-ips.txt` file, which is useful for reusing this information later in scripts or future automation steps.

```bash
az vm list-ip-addresses --resource-group devops-automation \
  --query "[].virtualMachine.{name:name, ip:network.publicIpAddresses[0].ipAddress}" \
  -o json | jq -r '.[] | "\(.name) \(.ip)"' > vms-ips.txt
```

### 📂 Check saved content:

This command shows the content of the `vms-ips.txt` file generated in the previous step. This way we can confirm that filtering and redirection worked correctly.

```bash
cat vms-ips.txt
```

```bash
IP_VM1=$(awk '$1 == "vm01" { print $2 }' vms-ips.txt)
echo $IP_VM1

IP_VM2=$(awk '$1 == "vm02" { print $2 }' vms-ips.txt)
echo $IP_VM1
```

---

## 📌 Stage 2 – Extract VM status

### 🧾 How raw output looks before filtering

If you run the command below for a VM:

```bash
az vm get-instance-view --resource-group devops-automation --name vm01 --output json
```

You will see something like this inside the `instanceView` key:

```json
{
  "statuses": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      ...
    },
    {
      "code": "PowerState/running",
      "displayStatus": "VM running",
      ...
    }
  ]
}
```

Based on this, we'll use `--query` to extract only the power status (on/off) cleanly.

### 🔍 Use loop to get current status:

In this example, we go through VMs `vm01` and `vm02` to collect their current status (on/off) using Azure CLI and save everything to the `status.txt` file.

```bash
for vm in vm01 vm02; do
  az vm get-instance-view \
    --resource-group devops-automation \
    --name $vm \
    --query "instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus" \
    --output tsv >> status.txt
done
```

### 📂 Display collected status:

This command prints the content of `status.txt`, allowing you to check the execution status of each VM queried in the previous loop.

```bash
cat status.txt
```

---

## 📋 Stage 3 – Use `awk` to reformat output

### 🛠️ Simple usage example:

This command reads the `vms-ips.txt` file and uses `awk` to format it more readably, useful when we want to print organized logs or prepare content for reporting or integration.

```bash
awk '{ print "VM:" $1 " | IP:" $2 }' vms-ips.txt
```
