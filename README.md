## Kubernetes Task 5 - Minikube Cluster Setup (Windows)

This repository contains a complete, Windows-optimized, phased guide to set up a local Kubernetes cluster using Minikube, deploy an Nginx application, expose it, scale it, perform rolling updates, and organize deliverables (YAML files and screenshots).

### Prerequisites
- Windows 10/11 (Docker Desktop supported)
- Docker Desktop for Windows (required for Docker driver)
- PowerShell (run as Administrator when needed)
- Internet access

### Folder Structure
- `deployment.yaml` — Nginx Deployment (replicas)
- `service.yaml` — NodePort Service (external access)
- `service-clusterip.yaml` — ClusterIP Service (internal access)
- `configmap.yaml` — ConfigMap example
- `screenshots/` — Place all required screenshots here

---

## Phase 1: Environment Setup (20–30 mins)

#### Step 1: Install Docker Desktop
- Download and install from `https://www.docker.com/products/docker-desktop/`
- Restart your computer after installation
- Open Docker Desktop and ensure it is running

#### Step 2: Verify Docker
```powershell
docker --version
docker ps
```

#### Step 3: Install Chocolatey (optional but recommended)
Run PowerShell as Administrator:
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

#### Step 4: Install kubectl
Option 1 (recommended):
```powershell
choco install kubernetes-cli -y
```
Option 2 (manual): download `kubectl.exe` from `https://dl.k8s.io/release/stable.txt` (Windows amd64) and add its folder to PATH.

Verify:
```powershell
kubectl version --client
```

#### Step 5: Install Minikube
Option 1 (recommended):
```powershell
choco install minikube -y
```
Option 2: Download the Windows installer from `https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe` and run it.

Verify:
```powershell
minikube version
```

#### Step 6: Start Minikube Cluster
```powershell
minikube start --driver=docker

minikube status
kubectl cluster-info
kubectl get nodes
```

Screenshot 1: PowerShell showing `minikube status`, `kubectl cluster-info`, and `kubectl get nodes`.

---

## Phase 2: Create and Apply Deployment (20–25 mins)

Create a working folder (optional):
```powershell
mkdir C:\\k8s-task5
cd C:\\k8s-task5
```

Create `deployment.yaml` (already included in repo):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

Apply and verify:
```powershell
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
kubectl get pods -o wide
```

Screenshot 2: `kubectl get pods` showing 3 pods in Running status.

---

## Phase 3: Expose Application (10–15 mins)

Create `service.yaml` (NodePort):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

Apply and verify:
```powershell
kubectl apply -f service.yaml
kubectl get services
kubectl get svc

# Get access URL
minikube service nginx-service --url
```

Open the printed URL (e.g., `http://192.168.49.2:30080`) in your browser.

Screenshot 3: `kubectl get services` output and browser with Nginx welcome page.

---

## Phase 4: Scaling and Management (15–20 mins)

Scale up and verify:
```powershell
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods
kubectl get deployments
```

Screenshot 4: `kubectl get pods` showing 5 pods.

Scale down:
```powershell
kubectl scale deployment nginx-deployment --replicas=2
kubectl get pods
```

Describe and logs:
```powershell
kubectl describe deployment nginx-deployment
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Screenshot 5: Output of `kubectl describe deployment nginx-deployment`.

---

## Phase 5: Advanced Operations (15–20 mins)

Create a namespace:
```powershell
kubectl create namespace dev-environment
kubectl get namespaces
```

Create `configmap.yaml` (example):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  database_url: "mongodb://localhost:27017"
  app_name: "My Kubernetes App"
  environment: "development"
```

Apply and view:
```powershell
kubectl apply -f configmap.yaml
kubectl get configmaps
kubectl describe configmap app-config
```

Test a ClusterIP service (`service-clusterip.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

```powershell
kubectl apply -f service-clusterip.yaml
kubectl get services
```

Perform a rolling update (edit image in `deployment.yaml`):
```yaml
image: nginx:1.25
```

```powershell
kubectl apply -f deployment.yaml
kubectl rollout status deployment nginx-deployment
kubectl rollout history deployment nginx-deployment
```

Screenshot 6: Output of `kubectl rollout status`.

---

## Phase 6: Cleanup and Documentation (10–15 mins)

List all resources:
```powershell
kubectl get all
kubectl get all -o wide
```

Screenshot 7: Final `kubectl get all` output.

Cleanup (optional):
```powershell
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete -f configmap.yaml
minikube stop
# minikube delete  # if you want to remove the cluster
```

---

## Phase 7: GitHub Repository Setup (15–20 mins)

Create `screenshots/` and move your images there. Then initialize Git:
```powershell
git init
git add .
git commit -m "Initial commit: Kubernetes Task 5 - Minikube deployment"
git remote add origin https://github.com/YOUR_USERNAME/k8s-task5.git
git branch -M main
git push -u origin main
```

Suggested `.gitignore` entries are provided in the repo.

---

## Interview Questions — Quick Answers
- **What is Kubernetes?** Open-source container orchestration for deployment, scaling, management.
- **Role of kubelet?** Node agent that ensures containers in Pods are running and reports to control plane.
- **Pods vs Deployments vs Services?** Pod = smallest unit; Deployment = manages replicas and updates; Service = stable networking to Pods.
- **How to scale?** `kubectl scale deployment <name> --replicas=<n>`
- **What is a namespace?** Virtual cluster for isolation and organization.
- **Service types?** ClusterIP (internal), NodePort (node IP + static port), LoadBalancer (cloud LB).
- **What are ConfigMaps?** Non-secret key/value configuration for Pods.
- **Rolling updates?** Update image, then `kubectl rollout status` and `kubectl rollout history`.

---

## Troubleshooting (Windows)
- **Docker not running:** Start Docker Desktop; restart if needed.
- **Hyper-V conflicts:** Prefer `--driver=docker`. If using Hyper-V, ensure it is enabled (Windows Pro/Enterprise).
- **PowerShell execution policy:** Run as Admin; `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`.
- **kubectl not recognized:** Add its install directory to PATH (e.g., `C:\\Program Files\\kubectl\\`).

---

## Time Estimate
- Total: 2–3 hours
- Setup: 30 mins
- Deployment: 45 mins
- Testing: 30 mins
- Documentation: 45 mins

Good luck with your task! 🚀