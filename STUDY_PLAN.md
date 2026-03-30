# Kubernetes Study Plan: From Zero to Production

## Real project: "TaskFlow" — a task app with microservices

A web application with **3 independent services** deployed on Kubernetes:
- `frontend` — React (or static HTML served with nginx)
- `api` — Node.js/Express REST API
- `db` — PostgreSQL

This project covers 90% of what you use in real Kubernetes production environments.

---

## Prerequisites

- Docker installed and running
- `kubectl` installed (`brew install kubectl`)
- Basic Docker knowledge (knowing what an image and a container are)

---

## Phase 0 — Understand the problem Kubernetes solves (2-3h)

### What is Kubernetes?

Kubernetes (K8s) is a container orchestrator. It answers questions like:
- Where do my containers run?
- What happens if one crashes? (it restarts automatically)
- How do I scale from 1 to 10 replicas under load?
- How do I update without downtime?

### Key concepts

| Term | Mental model |
|---|---|
| **Cluster** | The "datacenter" (one or more servers) |
| **Node** | A server inside the cluster |
| **Pod** | The smallest unit K8s manages (a container or group of containers) |
| **Deployment** | "I want 3 replicas of this Pod, always" |
| **Service** | Stable access point to a group of Pods (like an internal load balancer) |
| **Namespace** | Logical folder to separate environments (dev/prod) |
| **ConfigMap** | Non-sensitive configuration variables |
| **Secret** | Sensitive variables (passwords, tokens) |
| **Ingress** | The reverse proxy that receives external traffic and routes it to Services |
| **PersistentVolume** | A disk that survives even if the Pod dies |

### Recommended reading
- [Kubernetes official concepts](https://kubernetes.io/docs/concepts/) — just the "Overview" section

---

## Phase 1 — Local environment with kind (3-4h)

**kind** (Kubernetes IN Docker) is the lightest option. It creates a real cluster inside Docker containers.

### Installation

```bash
# macOS
brew install kind

# Create a local cluster
kind create cluster --name taskflow

# Verify it works
kubectl get nodes
kubectl cluster-info
```

### Alternative: minikube

```bash
brew install minikube
minikube start
```

> **kind** is preferred for development because it's faster and integrates better with CI/CD pipelines.

### Your first Pod (exercise 1)

```yaml
# 01-primer-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: hello
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f 01-primer-pod.yaml
kubectl get pods
kubectl describe pod my-first-pod
kubectl logs my-first-pod

# Access from your machine
kubectl port-forward pod/my-first-pod 8080:80
# Open http://localhost:8080
```

---

## Phase 2 — Deployments and Services (4-5h)

Bare Pods are not used in production. **Deployments** manage Pods automatically.

### Deployment (exercise 2)

```yaml
# 02-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 3                          # 3 copies of the Pod
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: nginx:alpine          # replace with your real image
          ports:
            - containerPort: 3000
```

```bash
kubectl apply -f 02-deployment.yaml
kubectl get deployments
kubectl get pods                       # you will see 3 pods

# Simulate a failure: delete a pod and K8s recreates it
kubectl delete pod <pod-name>
kubectl get pods --watch
```

### Service (exercise 3)

```yaml
# 03-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api                           # routes to Pods with this label
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP                      # only accessible inside the cluster
```

**Service types:**
| Type | Use case |
|---|---|
| `ClusterIP` | Internal communication between services |
| `NodePort` | Exposes a port on each node (local testing) |
| `LoadBalancer` | Creates an external load balancer (cloud only) |

---

## Phase 3 — ConfigMaps, Secrets and environment variables (3h)

```yaml
# 04-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  NODE_ENV: "production"
  API_PORT: "3000"
  DB_HOST: "postgres-service"
```

```yaml
# 05-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=       # base64: echo -n "password123" | base64
  DB_USER: YWRtaW4=                    # base64: echo -n "admin" | base64
```

### Using ConfigMap and Secret in a Deployment

```yaml
spec:
  containers:
    - name: api
      image: my-api:latest
      envFrom:
        - configMapRef:
            name: api-config
        - secretRef:
            name: db-credentials
```

---

## Phase 4 — Persistent storage (3h)

Pods are ephemeral: if they die, their data is lost. Databases need **PersistentVolumes**.

```yaml
# 06-postgres.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15-alpine
          envFrom:
            - secretRef:
                name: db-credentials
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
    - port: 5432
```

---

## Phase 5 — Ingress: external traffic (3h)

**Ingress** acts as a reverse proxy that receives external requests and routes them to the right Service based on the URL.

```bash
# Install the nginx Ingress Controller in kind
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

```yaml
# 07-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: taskflow-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: taskflow.local
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

---

## Phase 6 — Full "TaskFlow" project (8-10h)

### File structure

```
taskflow/
├── k8s/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── postgres/
│   │   ├── pvc.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── api/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ingress.yaml
├── api/
│   ├── Dockerfile
│   └── src/...
└── frontend/
    ├── Dockerfile
    └── src/...
```

### Useful operational commands

```bash
# See everything in a namespace
kubectl get all -n taskflow

# Stream logs in real time
kubectl logs -f deployment/api-deployment

# Scale manually
kubectl scale deployment api-deployment --replicas=5

# Rolling update (update image with no downtime)
kubectl set image deployment/api-deployment api=my-api:v2

# View rollout history
kubectl rollout history deployment/api-deployment

# Rollback
kubectl rollout undo deployment/api-deployment

# Run a command inside a Pod
kubectl exec -it <pod-name> -- /bin/sh

# View resource usage
kubectl top pods
kubectl top nodes
```

---

## Phase 7 — Helm: the K8s package manager (4h)

Helm is like `npm` but for Kubernetes. Instead of applying 10 YAMLs, you install a "chart".

```bash
brew install helm

# Install PostgreSQL with Helm (does everything in Phase 4 automatically)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql \
  --set auth.postgresPassword=mypassword \
  --set primary.persistence.size=1Gi

# List installed releases
helm list

# Uninstall
helm uninstall my-postgres
```

### Create your own chart for TaskFlow

```bash
helm create taskflow-chart
# Edit the templates in taskflow-chart/templates/
helm install taskflow ./taskflow-chart
```

---

## Phase 8 — AWS with EKS (minimum cost) (5-6h)

### Estimated minimum cost

| Resource | Approximate cost |
|---|---|
| EKS Control Plane | ~$0.10/hour (~$73/month) |
| 1x t3.small worker node | ~$0.023/hour (~$17/month) |
| **Minimum total** | **~$90/month** |

> For learning purposes, only run the cluster when you need it and **delete it when you're done**.
> With Fargate you can get down to ~$0.04/vCPU/hour (pay only for what you use).

### Tool installation

```bash
brew install eksctl awscli

# Configure AWS credentials
aws configure
```

### Create a minimal EKS cluster

```bash
cat <<EOF > cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: taskflow-learn
  region: eu-west-1
nodeGroups:
  - name: workers
    instanceType: t3.small
    desiredCapacity: 1
    minSize: 1
    maxSize: 2
EOF

eksctl create cluster -f cluster-config.yaml

# Verify
kubectl get nodes
```

### Differences from local

| Concept | Local (kind) | AWS (EKS) |
|---|---|---|
| LoadBalancer Service | Does not work (use NodePort) | Automatically creates an ALB/NLB |
| Storage | hostPath / local-path | EBS (gp3) automatically provisioned |
| Ingress | Manual nginx | AWS Load Balancer Controller |
| Registry | Local images with `kind load` | ECR (Elastic Container Registry) |

### Push images to ECR

```bash
# Create repository
aws ecr create-repository --repository-name taskflow-api --region eu-west-1

# Login
aws ecr get-login-password --region eu-west-1 | \
  docker login --username AWS --password-stdin <account-id>.dkr.ecr.eu-west-1.amazonaws.com

# Tag and push
docker tag my-api:latest <account-id>.dkr.ecr.eu-west-1.amazonaws.com/taskflow-api:latest
docker push <account-id>.dkr.ecr.eu-west-1.amazonaws.com/taskflow-api:latest
```

### **IMPORTANT: Delete the cluster when you are done**

```bash
eksctl delete cluster --name taskflow-learn --region eu-west-1
```

---

## Recommended resources

| Resource | Type | Level |
|---|---|---|
| [kubernetes.io/docs](https://kubernetes.io/docs/tutorials/) | Official documentation | All |
| [Play with Kubernetes](https://labs.play-with-k8s.com/) | Free online lab | Beginner |
| [Killer.sh](https://killer.sh) | CKA exam simulator | Advanced |
| Book: "Kubernetes in Action" (Manning) | Book | Intermediate/Advanced |
| [kube.academy](https://kube.academy) | Free video courses | Beginner/Intermediate |

---

## Progress checklist

- [ ] Phase 0: I understand what problem K8s solves
- [ ] Phase 1: kind is running and I launched my first Pod
- [ ] Phase 2: I created a Deployment with 3 replicas and a Service
- [ ] Phase 3: I used ConfigMaps and Secrets in a Deployment
- [ ] Phase 4: PostgreSQL is running with persistent storage
- [ ] Phase 5: Ingress is routing traffic to two services
- [ ] Phase 6: Full TaskFlow running locally
- [ ] Phase 7: I installed something with Helm and created my own chart
- [ ] Phase 8: I deployed TaskFlow on EKS and deleted the cluster when done

---

## Estimated total time

| Phase | Hours |
|---|---|
| 0-2 (Fundamentals + kind + Deployments) | ~10h |
| 3-5 (Config + Storage + Ingress) | ~9h |
| 6 (Full TaskFlow project) | ~10h |
| 7 (Helm) | ~4h |
| 8 (AWS EKS) | ~6h |
| **Total** | **~39h** |

At a pace of 1-2 hours per day: approximately **3-5 weeks**.
