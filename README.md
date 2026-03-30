# kubernetes-from-zero

A hands-on Kubernetes guide from scratch, with real exercises and a full microservices project (TaskFlow) running locally with kind and on AWS EKS.

## What you will build

**TaskFlow** — a task management app with 3 independent services:

```
frontend (nginx)  ──►  api (Node.js)  ──►  db (PostgreSQL)
```

All running on Kubernetes, locally and on AWS.

## Prerequisites

- [Docker](https://www.docker.com/products/docker-desktop/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) — `brew install kubectl`
- [kind](https://kind.sigs.k8s.io/) — `brew install kind`

## Quick start

```bash
# 1. Create a local cluster
kind create cluster --name taskflow

# 2. Work through the exercises in order
cd exercises
kubectl apply -f 01-primer-pod.yaml
```

## Exercises

| # | Topic | Concepts |
|---|---|---|
| [01](exercises/01-primer-pod.yaml) | [Your first Pod](exercises/01-primer-pod.md) | Pod, port-forward, ephemeral containers |
| [02](exercises/02-deployment.yaml) | [Deployment](exercises/02-deployment.md) | Replicas, self-healing, scaling |
| [03](exercises/03-service.yaml) | [Service](exercises/03-service.md) | ClusterIP, DNS, load balancing |
| [04](exercises/04-configmap.yaml) | [ConfigMap](exercises/04-configmap.md) | Environment variables, configuration |
| [05](exercises/05-secret.yaml) | [Secret](exercises/05-secret.md) | Sensitive data, base64 |
| [06](exercises/06-deployment-con-config.yaml) | [Deployment + Config](exercises/06-deployment-con-config.md) | Injecting ConfigMap and Secret into a Pod |
| [07](exercises/07-postgres.yaml) | [Persistent storage](exercises/07-postgres.md) | PVC, PersistentVolume, stateful workloads |

## Study plan

See [STUDY_PLAN.md](STUDY_PLAN.md) for the full 8-phase roadmap, including Ingress, Helm, and AWS EKS.

## Estimated time

| Phases | Hours |
|---|---|
| 0–2: Fundamentals + kind + Deployments | ~10h |
| 3–5: Config + Storage + Ingress | ~9h |
| 6: Full TaskFlow project | ~10h |
| 7: Helm | ~4h |
| 8: AWS EKS | ~6h |
| **Total** | **~39h** |

## Resources

- [kubernetes.io/docs](https://kubernetes.io/docs/tutorials/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/) — free online lab
- [kube.academy](https://kube.academy) — free video courses
- Book: *Kubernetes in Action* (Manning)
