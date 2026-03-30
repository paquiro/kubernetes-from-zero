# 02 — Deployment

## What is a Deployment

Guarantees that N copies of a Pod are always running. If a Pod dies, it recreates it automatically.
This is what you use in production instead of bare Pods.

```
Deployment → manages → ReplicaSet → manages → Pods
"I want 3"             "there are 3"          Pod1, Pod2, Pod3
```

## What this YAML does

Creates 3 replicas of an nginx Pod. If any of them dies, K8s recreates it automatically.

## Key fields

| Field | Purpose |
|---|---|
| `replicas` | Number of Pods you want running at all times |
| `selector.matchLabels` | Which Pods this Deployment manages |
| `template` | Template used to create each Pod |

## Commands

```bash
# Create
kubectl apply -f 02-deployment.yaml

# List deployments
kubectl get deployments

# List the Pods it created (with their IPs)
kubectl get pods -o wide

# Scale to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5

# See the full YAML stored in K8s
kubectl get deployment nginx-deployment -o yaml

# Delete
kubectl delete deployment nginx-deployment
```

## What you learned

- Delete a Pod and K8s recreates it automatically (self-healing)
- `replicas` is the only place where the number of Pods is defined
- The Deployment knows nothing about the Service — they are independent
