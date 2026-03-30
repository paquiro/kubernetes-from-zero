# 01 — Your first Pod

## What is a Pod

The smallest unit in Kubernetes. It contains one or more containers that share the same IP and disk.
In practice, 95% of Pods have a single container.

**Key characteristic: it is ephemeral.** If it dies, it's gone. Nothing recreates it automatically.

## What this YAML does

Launches a Pod with nginx (a lightweight web server) listening on port 80.

## Commands

```bash
# Create
kubectl apply -f 01-primer-pod.yaml

# Check status
kubectl get pods

# Detailed info (IP, node, events)
kubectl describe pod my-first-pod

# View logs
kubectl logs my-first-pod

# Access from your browser (localhost:8080 → Pod port 80)
kubectl port-forward pod/my-first-pod 8080:80

# Delete
kubectl delete pod my-first-pod
```

## What you learned

- Pods have their own IP inside the cluster — your machine cannot reach it directly
- `port-forward` creates a temporary tunnel for development and debugging
- If you delete the Pod, it's gone forever → that's why Deployments exist
