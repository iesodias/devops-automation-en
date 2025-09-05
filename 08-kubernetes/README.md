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

### 3. View execution plan

```bash
terraform plan
```

---

### 4. Apply infrastructure

```bash
terraform apply -auto-approve
```

---

### 5. Access AKS cluster

```bash
az aks get-credentials --resource-group rg-aks-free --name aks-free-tier
kubectl get nodes
```

# Lab 2 - Creating your first Pod in Kubernetes (port 8081)

## Objective

Manually create a Pod with the image `iesodias/java-api:latest`, running on port `8081`, and interact with it via `kubectl`.

---

## Step by Step

### 1. Create directory structure

```bash
mkdir -p ~/k8s-workshop/lab1
cd ~/k8s-workshop/lab1
```

---

### 2. Create Pod manifest

```bash
vi pod-java-api.yaml
```

Paste the content below into the file, with the correct port:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: java-api-pod
spec:
  containers:
    - name: java-api
      image: iesodias/java-api:latest
      ports:
        - containerPort: 8081
```

---

### 3. Apply the manifest

```bash
kubectl apply -f pod-java-api.yaml
```

---

### 4. Check Pod status

```bash
kubectl get pods
```

---

### 5. Interact with the Pod

```bash
# View container logs
kubectl logs java-api-pod

# Access shell (if the image allows)
kubectl exec -it java-api-pod -- sh

# View Pod details
kubectl describe pod java-api-pod
```

---

### 6. Test application locally

Use `port-forward` to access port 8081 locally:

```bash
kubectl port-forward pod/java-api-pod 8081:8081
```

Open another terminal and test with `curl`:

```bash
curl http://localhost:8081
```

---

### 7. Environment Cleanup (optional, but recommended)

After finishing tests, you can delete the Pod to keep your cluster clean:

```bash
kubectl delete -f pod-java-api.yaml
```

Or, alternatively, by Pod name:

```bash
kubectl delete pod java-api-pod
```

> Emphasize in the workshop that in the real world, you usually don't create Pods "by hand" — this is only educational.

---

## Expected Result

* Pod running the image `iesodias/java-api:latest`
* Port `8081` forwarded locally
* Application responding via `curl`

---


# Lab 3 – Using ReplicaSet to maintain multiple Pods

## Objective

* Create a ReplicaSet with the image `iesodias/java-api:latest`
* Observe automatic replica creation
* Increase and decrease replica count
* Manually delete Pods and observe ReplicaSet restoring them
* Finish with resource cleanup

---

## Step by Step

### 1. Create lab directory

```bash
mkdir -p ~/k8s-workshop/lab2
cd ~/k8s-workshop/lab2
```

---

### 2. Create ReplicaSet manifest

```bash
vi replicaset-java-api.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: java-api-replicaset
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
```

> Note the `selector.matchLabels` and `labels` in the `template`. This is crucial for the ReplicaSet to control the Pods.

---

### 3. Apply the manifest

```bash
kubectl apply -f replicaset-java-api.yaml
```

---

### 4. Check created replicas

```bash
kubectl get pods -l app=java-api
kubectl get rs
```

---

### 5. Test ReplicaSet behavior

#### Increase replica count:

```bash
kubectl scale rs java-api-replicaset --replicas=4
kubectl get pods -l app=java-api
```

#### Decrease replica count:

```bash
kubectl scale rs java-api-replicaset --replicas=1
kubectl get pods -l app=java-api
```

#### Manually delete a Pod and observe auto-recovery:

```bash
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods -l app=java-api -w
```

> The ReplicaSet will detect the absence and recreate the Pod automatically.

---

### 6. Test one of the Pods locally

Use `kubectl get pods` to get a valid Pod name and:

```bash
kubectl port-forward pod/<pod-name> 8081:8081
curl http://localhost:8081
```

---

### 7. Environment Cleanup

```bash
kubectl delete -f replicaset-java-api.yaml
```

---

## Expected Result

* Understand what a ReplicaSet is
* Visualize automatic replica creation
* Scale horizontally (up/down)
* See self-healing behavior when deleting Pods
* Finish by cleaning all resources


# Lab 4 – Deployment with replicas, rolling updates and rollback

## Objective

* Create a Deployment with multiple replicas
* Perform rolling update by changing the image
* Observe how the Deployment manages ReplicaSets
* Test horizontal scalability
* Simulate an error and perform rollback
* Access the application locally
* Delete all resources at the end

---

## Step by Step

### 1. Create lab directory

```bash
mkdir -p ~/k8s-workshop/lab3
cd ~/k8s-workshop/lab3
```

---

### 2. Create Deployment manifest

```bash
vi deployment-java-api.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
```

---

### 3. Apply the manifest

```bash
kubectl apply -f deployment-java-api.yaml
```

---

### 4. Check Deployment, ReplicaSets and Pods

```bash
kubectl get deployments
kubectl get rs
kubectl get pods -l app=java-api
```

---

### 5. Test one of the Pods via port-forward

```bash
kubectl get pods -l app=java-api
kubectl port-forward pod/<pod-name> 8081:8081
curl http://localhost:8081
```

---

### 6. Scale the Deployment

```bash
kubectl scale deployment java-api-deployment --replicas=4
kubectl get pods -l app=java-api
```

---

### 7. Update image with wrong tag (simulating failure)

```bash
kubectl set image deployment java-api-deployment java-api=iesodias/java-api:errada
```

Watch the rollout:

```bash
kubectl rollout status deployment java-api-deployment
kubectl get pods
```

You will see rollout failures (Pods in `CrashLoopBackOff` or `ImagePullBackOff`).

---

### 8. Rollback the failed update

```bash
kubectl rollout undo deployment java-api-deployment
kubectl rollout status deployment java-api-deployment
```

---

### 9. View rollout history

```bash
kubectl rollout history deployment java-api-deployment
```

---

### 10. Complete cleanup of created resources

```bash
kubectl delete -f deployment-java-api.yaml
```

---

## Expected Result

* Deployment with 2+ replicas created and functional
* Scaled to 4 Pods
* Rolling update with simulated error
* Rollback executed successfully
* Local access via port-forward
* Resources removed at the end
 
---

# Lab 5 – Exposing your application via LoadBalancer in AKS (Single Manifest)

## Objective

* Create a Deployment with multiple Java application Pods
* Expose with a LoadBalancer type Service
* Access the application via browser and curl
* See load balancing between Pods
* Finish with resource cleanup

---

## Step by Step

### 1. Create directory structure

```bash
mkdir -p ~/k8s-workshop/lab4
cd ~/k8s-workshop/lab4
```

---

### 2. Create combined manifest

```bash
vi java-api-lb.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-service
spec:
  type: LoadBalancer
  selector:
    app: java-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8081
```

---

### 3. Apply the manifest

```bash
kubectl apply -f java-api-lb.yaml
```

---

### 4. Wait for public IP to be assigned

```bash
kubectl get service java-api-service
```

Wait until you see a valid value in the `EXTERNAL-IP` field, for example:

```bash
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
java-api-service    LoadBalancer   10.0.128.34    52.170.42.120   80:80/TCP      2m
```

---

### 5. Access the application in the browser

Open the browser and access:

```
http://<EXTERNAL-IP>
```

Example:

```
http://52.170.42.120
```

You should see the Java API response.

---

### 6. Test with curl

```bash
curl http://<EXTERNAL-IP>
```

---

### 7. Test load balancing between Pods

If your application returns some identification (like hostname), execute:

```bash
watch -n 1 curl http://<EXTERNAL-IP>
```

---

### 8. Scale the Deployment

```bash
kubectl scale deployment java-api-deployment --replicas=5
kubectl get pods
```

---

### 9. Delete a Pod and observe recovery

```bash
kubectl delete pod <pod-name>
kubectl get pods -w
```

---

### 10. Lab Cleanup

```bash
kubectl delete -f java-api-lb.yaml
```

---

## Expected Result

* Application accessible via public IP and browser
* LoadBalancer type Service generating external IP in AKS
* Scalability and high availability validated
* Clean environment after completion

---

# Lab 6 – Namespaces in Kubernetes: organizing dev, hml and prod environments

## Objective

* Understand the role of Namespaces in Kubernetes
* Create isolated environments: dev, hml and prod
* Deploy the same application in all three environments
* Access Pods individually in each namespace
* Observe resource isolation
* Clean up namespaces and their objects

---

## Step by Step

### 1. Create lab directory

```bash
mkdir -p ~/k8s-workshop/lab5
cd ~/k8s-workshop/lab5
```

---

### 2. Create a single manifest with all three environments

```bash
vi namespaces-multi-env.yaml
```

Complete content:

```yaml
# Creating namespaces
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: hml
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
---
# Deployment in dev namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        env: dev
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
# Deployment in hml namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: hml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        env: hml
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
# Deployment in prod namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        env: prod
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
```

---

### 3. Apply all resources

```bash
kubectl apply -f namespaces-multi-env.yaml
```

---

### 4. View created namespaces

```bash
kubectl get namespaces
```

---

### 5. List Pods by namespace

```bash
kubectl get pods -n dev
kubectl get pods -n hml
kubectl get pods -n prod
```

---

### 6. Access a Pod from each environment

```bash
kubectl exec -n dev -it $(kubectl get pods -n dev -o jsonpath='{.items[0].metadata.name}') -- sh
```

Repeat the command for `hml` and `prod` namespaces.

---

### 7. Compare logs across environments

```bash
kubectl logs -n dev -l app=java-api
kubectl logs -n hml -l app=java-api
kubectl logs -n prod -l app=java-api
```

---

### 8. Simulate failure only in dev

```bash
kubectl delete pod -n dev -l app=java-api
kubectl get pods -n dev -w
```

> The ReplicaSet in the dev environment will recreate the Pod automatically. Other environments are not affected.

---

### 9. Create a Service per namespace (optional)

```bash
kubectl expose deployment java-api --port=80 --target-port=8081 -n dev
```

Access via port-forward:

```bash
kubectl port-forward -n dev service/java-api 8081:80
```

---

### 10. Complete lab cleanup

```bash
kubectl delete namespace dev
kubectl delete namespace hml
kubectl delete namespace prod
```

> This automatically removes all objects from each namespace.

---

## Expected Result

* Three namespaces (dev, hml, prod) active
* Same application running isolated in each environment
* Separate access and logs per namespace
* Automatic Pod recreation only in affected environment
* Complete cleanup at the end

---

# Lab 7 – CPU and Memory Request & Limit

## Objective

* Create a Deployment with CPU and memory limits
* Observe how Kubernetes reserves and limits resources
* Adjust values and see effect on the pod
* Check metrics with `kubectl top`
* Finish with resource cleanup

---

## Step by Step

### 1. Create lab directory

```bash
mkdir -p ~/k8s-workshop/lab6
cd ~/k8s-workshop/lab6
```

---

### 2. Create manifest with requests and limits

```bash
vi java-api-limited.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-limited
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api-limited
  template:
    metadata:
      labels:
        app: java-api-limited
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
```

---

### 3. Apply the manifest

```bash
kubectl apply -f java-api-limited.yaml
```

---

### 4. See the running pod

```bash
kubectl get pods
kubectl describe pod -l app=java-api-limited
```

---

### 5. Check usage with kubectl top

> If in AKS, Metrics Server is already enabled

```bash
kubectl top pod -l app=java-api-limited
```

Example output:

```
NAME                                  CPU(cores)   MEMORY(bytes)
java-api-limited-xxxxxxxxxx-xxxxx     12m          90Mi
```

---

### 6. Simulate load on pod (optional)

> If the image supports generating load or there's a configured sidecar, usage will increase until reaching limits.
> Otherwise, discuss the concept with students.

---

### 7. Increase limits via kubectl edit

```bash
kubectl edit deployment java-api-limited
```

Change values to:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

After saving:

```bash
kubectl rollout status deployment java-api-limited
```

---

### 8. View new usage with kubectl top

```bash
kubectl top pod -l app=java-api-limited
```

---

### 9. Check QoS assigned to Pod

```bash
kubectl get pod -l app=java-api-limited -o jsonpath="{.items[0].status.qosClass}"
```

You will see:

```
Burstable
```

### 10. Lab Cleanup

```bash
kubectl delete -f java-api-limited.yaml
```

---

## Expected Result

* Pod with requests and limits applied
* CPU/memory usage monitored with `kubectl top`
* Limit adjustments applied successfully
* Understanding of Kubernetes QoS (Quality of Service)
* Resources cleaned after completion

---

# Lab 8 – Health Checks with liveness and readiness probes (with failure simulation)

## Objective

* Add probes to Java application (liveness and readiness)
* See automatic restart behavior via liveness
* See Pod removal from traffic via readiness
* Simulate manual failure via kubectl edit
* Restoration and cleanup

---

## Step by Step

### 1. Create Lab structure

```bash
mkdir -p ~/k8s-workshop/lab7
cd ~/k8s-workshop/lab7
```

---

### 2. Create manifest with probes

```bash
vi java-api-health.yaml
```

Content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-health
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api-health
  template:
    metadata:
      labels:
        app: java-api-health
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8081
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8081
            initialDelaySeconds: 5
            periodSeconds: 3
            failureThreshold: 2
```

---

### 3. Apply the manifest

```bash
kubectl apply -f java-api-health.yaml
```

---

### 4. Check Pod status and probes

```bash
kubectl get pods
kubectl describe pod -l app=java-api-health
```

---

### 5. Test endpoint manually with curl

```bash
kubectl port-forward pod/$(kubectl get pod -l app=java-api-health -o jsonpath='{.items[0].metadata.name}') 8081:8081
```

In another terminal:

```bash
curl http://localhost:8081/actuator/health
```

Expected response:

```json
{"status":"UP"}
```

---

### 6. Simulate liveness failure by editing deployment

```bash
kubectl edit deployment java-api-health
```

Change:

```yaml
livenessProbe:
  httpGet:
    path: /actuator/404
```

Save and exit editor.

---

### 7. Observe Kubernetes behavior

```bash
kubectl get pods -l app=java-api-health -w
kubectl describe pod -l app=java-api-health
```

Expected messages:

```
Liveness probe failed: HTTP probe failed with statuscode: 404
Back-off restarting failed container
```

---

### 8. Restore correct endpoint

```bash
kubectl edit deployment java-api-health
```

Revert the liveness `path` to `/actuator/health`.

---

### 9. Check if pod returned to normal

```bash
kubectl get pods -l app=java-api-health
```

The Pod should be `READY 1/1` and `STATUS Running`.

---

### 10. Lab Cleanup

```bash
kubectl delete -f java-api-health.yaml
```

---

## Expected Result

* Health checks active successfully
* Liveness failure generated automatic container restart
* Readiness temporarily removed Pod from traffic
* Application restored after path correction
* Clean environment at the end

---

## Discussion Tips

| Concept      | Liveness               | Readiness                   |
| ------------ | ---------------------- | --------------------------- |
| When fails   | Container is restarted | Pod is removed from Service |
| Purpose      | Detect hang/deadlock   | Wait for app to be ready    |
| Example      | deadlock, thread stuck | cache loading               |


# Lab 9 – HPA with Sidecar: Autoscaling based on load generated via stress

## Objective

* Create a Deployment with two containers:

  * `java-api`: main application
  * `stress`: sidecar that generates CPU usage
* Apply an HPA that scales based on CPU
* See scaling behavior in real time
* Cleanup at the end

---

## Step by Step

### 1. Create Lab structure

```bash
mkdir -p ~/k8s-workshop/lab8
cd ~/k8s-workshop/lab8
```

---

### 2. Create manifest with Deployment + HPA + Sidecar

```bash
vi java-api-hpa.yaml
```

Complete content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-hpa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api-hpa
  template:
    metadata:
      labels:
        app: java-api-hpa
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "500m"
        - name: stress
          image: progrium/stress
          command: ["stress"]
          args: ["--cpu", "1"]
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "300m"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-api-hpa
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

### 3. Apply the manifest

```bash
kubectl apply -f java-api-hpa.yaml
```

---

### 4. Check if Deployment and HPA are working

```bash
kubectl get deployment
kubectl get pods
kubectl get hpa
```

Example output:

```
NAME             TARGETS   MINPODS   MAXPODS   REPLICAS
java-api-hpa     72%/50%    1         5         2
```

---

### 5. Observe scaling in real time

```bash
watch kubectl get hpa
```

And in parallel:

```bash
watch kubectl get pods -l app=java-api-hpa
```

> As stress consumes CPU, the number of replicas increases automatically.

---

### 6. Simulate load reduction

Edit the Deployment:

```bash
kubectl edit deployment java-api-hpa
```

Delete the `stress` container section and save.

> This will make new Pods come without load. HPA will see the drop in CPU usage and will reduce replicas.

---

### 7. Wait for HPA to scale down

```bash
watch kubectl get hpa
```

Example output:

```
NAME             TARGETS   MINPODS   MAXPODS   REPLICAS
java-api-hpa     15%/50%    1         5         1
```

---

### 8. Lab Cleanup

```bash
kubectl delete -f java-api-hpa.yaml
```

---

## Expected Result

* Deployment automatically scales from 1 to 5 Pods based on load
* Sidecar `stress` generates enough CPU to activate HPA
* Removing sidecar reduces load and Pods automatically decrease
* Complete cleanup at the end

# Lab 10 – Exposing application via Ingress with nip.io

## Objective

* Expose your Java application in AKS using Ingress
* Use dynamic domain with nip.io linked to Ingress fixed IP
* Test direct access via browser

---

## Step by Step

### 1. Create lab structure

```bash
mkdir -p ~/k8s-workshop/lab9
cd ~/k8s-workshop/lab9
```

---

### 2. Discover Ingress LoadBalancer fixed IP

```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

Expected output:

```
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)
ingress-nginx-controller   LoadBalancer   10.0.100.10   20.210.50.123    80:80/TCP
```

Note the EXTERNAL-IP. We'll use it in the next step.

---

### 3. Create manifest with Deployment + Service + Ingress (nip.io)

```bash
vi java-api-ingress-nip.yaml
```

Replace `20.210.50.123` with your Ingress Controller's real IP.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-service
  namespace: default
spec:
  selector:
    app: java-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8081
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: java-api-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: java-api.20.210.50.123.nip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: java-api-service
                port:
                  number: 80
```

---

### 4. Apply the manifest

```bash
kubectl apply -f java-api-ingress-nip.yaml
```

---

### 5. Access in browser

Open in browser:

```
http://java-api.<YOUR_IP>.nip.io
```

✔️ Java application should load correctly.

---

### 6. Test with curl (optional)

```bash
curl http://java-api.<YOUR_IP>.nip.io
```

---

### 7. Lab Cleanup

```bash
kubectl delete -f java-api-ingress-nip.yaml
```

---

## Expected Result

* Java application available via Ingress + nip.io domain
* Direct browser access without need to modify /etc/hosts
* 100% reusable and dynamic manifest

# Lab 11 – Autoscaling with KEDA via Cron (Scale-to-Zero)

## Objective

* Configure automatic scaling using cron scheduling
* Test real scale-to-zero (no pods outside window)
* Validate predictable scaling based on time (great for workshops and production)

---

## How to calculate UTC ⇄ your local time

Before configuring times in the manifest, check current UTC time and convert to your local time.

### Formula:

```bash
Local time = UTC - 3
UTC = Local time + 3
```

> Use `date -u` to see current UTC time and adjust `start` and `end` cron fields

---

## Lab Structure

```bash
mkdir -p ~/k8s-workshop/lab11
cd ~/k8s-workshop/lab11
```

---

## Create java-api-keda-cron.yaml manifest

```bash
vi java-api-keda-cron.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-keda
  labels:
    app: java-api-keda
spec:
  replicas: 0
  selector:
    matchLabels:
      app: java-api-keda
  template:
    metadata:
      labels:
        app: java-api-keda
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8081
            initialDelaySeconds: 20
            periodSeconds: 5
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-keda
spec:
  selector:
    app: java-api-keda
  ports:
    - port: 80
      targetPort: 8081
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: java-api-scaler
spec:
  scaleTargetRef:
    name: java-api-keda
  minReplicaCount: 0
  maxReplicaCount: 5
  cooldownPeriod: 30
  triggers:
    - type: cron
      metadata:
        timezone: UTC
        start: "56 3 * * *"
        end: "0 4 * * *"
        desiredReplicas: "2"
```

---

## Apply the manifest

```bash
kubectl apply -f java-api-keda-cron.yaml
```

---

## Testing the behavior

### 1. Before scheduled time

```bash
kubectl get pods
```

> Should show no pods (replicas = 0)

### 2. During window (e.g., 03:56 UTC)

```bash
watch kubectl get pods
```

> You will see 2 pods coming up automatically

### 3. After window (e.g., 04:00 UTC)

> Pods will be automatically removed, returning to "scale-to-zero" state

---

## Checking KEDA resources

```bash
kubectl get scaledobject
kubectl describe scaledobject java-api-scaler
kubectl get hpa
```

> You will see:
>
> * HPA automatically generated by KEDA
> * Status Ready = True
> * Scheduled behavior working

---

## Lab Cleanup

```bash
kubectl delete -f java-api-keda-cron.yaml
```

---

## Expected Result

| State                | Result                             |
| -------------------- | ---------------------------------- |
| Before scheduled time | 0 pods                            |
| During window        | 2 pods running                     |
| After window         | Automatic return to 0 pods         |

---

## Workshop Discussion Tips

| Concept              | Educational value                  |
| -------------------- | ---------------------------------- |
| Scheduled scaling    | Perfect for periodic tasks         |
| Real scale-to-zero   | Resource savings                   |
| Predictable cron     | Ideal for demonstrating control    |
| `date -u`            | Show that cluster uses UTC         |
