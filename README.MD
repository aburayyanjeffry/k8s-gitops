Introduction to GitOps for Kubernetes using Argo CD and NGINX web application on a local Kubernetes cluster running in Docker Desktop on a MacBook Pro M1.

This tutorial will cover:

- Installing Kubernetes locally with Docker Desktop
- Installing Argo CD
- Deploying applications using GitOps
- Syncing deployments automatically with Argo CD
- Updating configuration through Git and letting Argo CD redeploy automatically

---

# Architecture

```text
 Git Repository
       │
       ▼
    Argo CD
       │
       ▼
 Kubernetes Cluster
       │
       ▼
     NGINX
```

Whenever changes are pushed to Git, Argo CD detects the changes and synchronizes the Kubernetes cluster automatically.

---

# Prerequisites

- MacBook Pro M1/M2
- Homebrew installed
- Docker Desktop installed
- Internet connection

---

# 1. Install Docker Desktop

Follow the official Docker Desktop installation guide for macOS:

https://docs.docker.com/desktop/setup/install/mac-install/

After installation, start Docker Desktop.

---

# 2. Enable Kubernetes

1. Open Docker Desktop
2. Go to Settings → Kubernetes
3. Enable Kubernetes
4. Click Apply & Restart

Docker Desktop will request permission to install Kubernetes components.

Select Allow or Yes to continue.

Verify Kubernetes is running:

```bash
kubectl cluster-info
```

Expected output:

```text
Kubernetes control plane is running
CoreDNS is running
```

---

# 3. Create Namespace for Argo CD

```bash
kubectl create namespace argocd
```

---

# 4. Install Argo CD

Install Argo CD using the official manifests:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify pods are running:

```bash
kubectl get pods -n argocd
```

Wait until all pods show:

```text
STATUS = Running
```

---

# 5. Access Argo CD UI

Port forward the Argo CD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open browser:

https://localhost:8080

---

# 4.  Create a directory and the Kubernetes manifest files. 
## Option 1: Follow Along Using My Repository

You can clone or fork my repository to get all the example manifests:

👉 https://github.com/aburayyanjeffry/k8s-gitops.git

This is recommended if you want to follow the tutorial step-by-step without missing any files.

## Option 2: Create Everything Manually

Alternatively, you can manually create the Kubernetes manifest files in this directory as you follow along in the tutorial.

## 📚 Tutorial Chapters

Navigate sequentially through the modules to see your deployment strategy evolve:

### 🔹 [Part 1: Simple Manifests](./part1-simple/)
### 🔹 [Part 2: TBC ](./part2-tbc/)
### 🔹 [Part 3: TBC ](./part3-tbc/)

