# k8s-gitops part 1

Introduction to GitOps for Kubernetes using Argo CD and the Bitnami NGINX Helm chart on a local Kubernetes cluster running in Docker Desktop on a MacBook Pro M1.

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

# 3.  Create a directory and the Kubernetes manifest files. 
## Option 1: Follow Along Using My Repository

You can clone or fork my repository to get all the example manifests:

👉 https://github.com/aburayyanjeffry/k8s-gitops.git

This is recommended if you want to follow the tutorial step-by-step without missing any files.

## Option 2: Create Everything Manually

Alternatively, you can manually create the Kubernetes manifest files in this directory as you follow along in the tutorial.

# 4. Test Locally (Before GitOps)

If you want to validate everything before introducing Argo CD, you can apply the manifests directly to your Kubernetes cluster:
```bash
kubectl apply -f configmap.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl port-forward svc/nginx 8080:80
```
Open:

http://localhost:8081

Ctrl-C to stop the port-forward

Clean Up Test Resources

When you are done testing, delete all resources:
```bash
kubectl delete -f configmap.yml
kubectl delete -f deployment.yml
kubectl delete -f service.yml
```

# 5. Setup Git repository

Initialize a new local Git repository:
```bash
git init
git add .
git commit -m "Initial commit"
```

# 6. Push Repository to GitHub

You can either use your forked repository or create a new empty repository on [https://github.com]

Once created, follow the instructions provided by GitHub to push your code.

Example:
```bash
git remote add origin git@github.com:aburayyanjeffry/k8s-gitops.git
git branch -M main
git push -u origin main
```
---

# 7. Create Namespace for Argo CD

```bash
kubectl create namespace argocd
```

---

# 8. Install Argo CD

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

# 9. Access Argo CD UI

Port forward the Argo CD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open browser:

https://localhost:8080

---

# 10. Get Argo CD Admin Password

Retrieve the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

Login credentials:

```text
Username: admin
Password: <output from command above>
```

# 11. Create 
1. Click **+ NEW APP**

2. Enter the following details
- **Application Name** = nginx
- **Project Name** = default
- **SYNC POLICY** = Automatic
- **AUTO-CREATE NAMESPACE** checked
- Repository URL = https://github.com/aburayyanjeffry/k8s-gitops ( change to your repository at github )
- Path = nginx
- Cluster URL = https://kubernetes.default.svc ( Docker desktop K8s )
- Namespace = nginx-saya

3. Click create ( top left ) and wait for the application status to be **Healthy** and **Synced**

4. Click the applicaiton to see the detail

5. Verify the application at Kubernetes

```bash
kubectl get all -n nginx-saya
```

Sample running output
```bash
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-67646445cb-7dq7m   1/1     Running   0          97s

NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/nginx   NodePort   10.96.89.48   <none>        80:30707/TCP   97s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           97s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-67646445cb   1         1         1       97s
```

Port forward the nginx
```bash
kubectl port-forward svc/nginx 8081:80 -n nginx-saya
```

Open browser:

https://localhost:8081

# 12. Demonstrate GitOps in Action

Modify the deployment replica count.
```
nginx/deployment.yml
```

Change:
```yaml
spec:
  replicas: 1
```
To:
```yaml
spec:
  replicas: 2
```

Commit and push:
```bash
git add .
git commit -m "Increase replica count to 2"
git push
```
Argo CD will detect the change and automatically synchronize the cluster.

Verify:
```bash
kubectl get pods -n nginx
```

You should now see two NGINX pods running.

# 13. Update the ConfigMap

Edit the NGINX HTML content inside the ConfigMap and push the changes.

Argo CD will successfully synchronize the ConfigMap update.

However, the running NGINX Pods may continue serving the previous content.

This is expected because updating a ConfigMap does not automatically trigger a Deployment rollout.

## Option 1: Restart from Argo CD
Open the application
Select the Deployment resource
Click Actions
Click Restart

### Option 2: Restart Using kubectl
```bash
kubectl rollout restart deployment/nginx -n nginx
```

Monitor the rollout:
```bash
kubectl rollout status deployment/nginx -n nginx
```

Once the Pods restart, the updated content will be served.

# Conclusion

Congratulations! You have successfully:

Installed Kubernetes locally
Installed Argo CD
Connected Kubernetes to GitHub
Implemented a GitOps workflow
Performed automatic deployments through Git commits
Managed application updates using Argo CD
