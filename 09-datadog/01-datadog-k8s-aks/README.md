# Lab: Monitoring AKS with Datadog (Free Tier)

This lab shows how to create an AKS cluster on Free Tier and correctly configure the Datadog agent to collect metrics, logs and visualize everything on the Datadog dashboard.

---

## Step 1 — Create Resource Group in Azure
```bash
az group create --name aks-free-rg --location eastus
```

---

## ☸️ Step 2 — Create AKS Cluster (Free Tier)
```bash
az aks create \
  --resource-group aks-free-rg \
  --name aks-free-cluster \
  --node-count 1 \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --enable-managed-identity \
  --tier free \
  --location eastus
```

---

## 🔑 Step 3 — Access Cluster via kubectl
```bash
az aks get-credentials --resource-group aks-free-rg --name aks-free-cluster --overwrite-existing
```

---

## Step 4 — Add Datadog Helm repository
```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

---

## Step 5 — Create Secret with Datadog API Key
```bash
kubectl create namespace datadog

kubectl create secret generic datadog-secret \
  --from-literal=api-key=<SUA_API_KEY_DD> \
  -n datadog
```

---

## ⚙️ Step 6 — Create `datadog-values.yaml` file

Create a file called `datadog-values.yaml` with the following content:

```yaml
datadog:
  apiKeyExistingSecret: "datadog-secret"
  clusterName: "aks-free-cluster"
  logs:
    enabled: true
    containerCollectAll: true
  hostnameForceConfigAsCanonical: false
  kubelet:
    tlsVerify: false

agents:
  useHostPID: true

rbac:
  create: true
  serviceAccount:
    create: true

clusterAgent:
  enabled: true
  replicas: 1
  service:
    enabled: true
  metricsProvider:
    enabled: true
  createPodDisruptionBudget: true
```

---

## Step 7 — Install Datadog with Helm
```bash
helm upgrade --install datadog-agent datadog/datadog \
  -f datadog-values.yaml \
  -n datadog
```

> ⚠️ If already installed before:
```bash
helm upgrade datadog-agent datadog/datadog \
  -f datadog-values.yaml \
  -n datadog
```

---

## Step 8 — Deploy `java-api` application with Datadog APM + Logs

Create a `deployment.yaml` file with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  labels:
    app: java-api
    tags.datadoghq.com/env: dev
    tags.datadoghq.com/service: java-api
    tags.datadoghq.com/version: "1.0.0"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        tags.datadoghq.com/env: dev
        tags.datadoghq.com/service: java-api
        tags.datadoghq.com/version: "1.0.0"
        admission.datadoghq.com/enabled: "true"
      annotations:
        ad.datadoghq.com/java-api.logs: '[{"source":"java","service":"java-api"}]'
        admission.datadoghq.com/java-lib.version: v1.48.1
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          env:
            - name: DD_LOGS_INJECTION
              value: "true"
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: java-api
  labels:
    app: java-api
spec:
  type: LoadBalancer
  selector:
    app: java-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8081
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-api
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

## Step 9 — Apply the deployment
```bash
kubectl apply -f deployment.yaml
```

---

## Step 10 — Get application public IP
```bash
kubectl get svc java-api
```
Access via browser or curl:
```bash
curl http://<IP_PUBLICO>
```

---

## Step 11 — Visualize in Datadog
- Go to **APM > Services** and see the traces of the `java-api` application
- Go to **Logs > Explorer** and filter by `source:java` or `service:java-api`

---

🔥 Done! Now you have a Java application on AKS monitored with APM, logs and autoscaling fully integrated with Datadog.
