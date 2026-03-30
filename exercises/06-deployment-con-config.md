# 06 — Deployment with ConfigMap and Secret

## What this YAML does

A Deployment that loads environment variables from the `api-config` ConfigMap
and the `db-credentials` Secret created in the previous exercises.

## Dependencies

Before applying this YAML you need:
- `04-configmap.yaml` → ConfigMap `api-config`
- `05-secret.yaml` → Secret `db-credentials`

## How to verify the variables reach the Pod

```bash
# Shell into the Pod
kubectl exec -it <pod-name> -- sh

# List all environment variables
env

# Exit
exit
```

Or directly:

```bash
kubectl exec -it $(kubectl get pod -l app=api -o jsonpath='{.items[0].metadata.name}') -- env
```

## What happens when you update the ConfigMap

If you modify a ConfigMap and run `kubectl apply`, existing Pods **do not update automatically**.
You need to restart the Deployment:

```bash
kubectl rollout restart deployment api-deployment
```

## Commands

```bash
# Create (requires 04 and 05 applied first)
kubectl apply -f 06-deployment-con-config.yaml

# Check variables inside the Pod
kubectl exec -it $(kubectl get pod -l app=api -o jsonpath='{.items[0].metadata.name}') -- env

# Restart Pods to pick up new configuration
kubectl rollout restart deployment api-deployment

# Delete
kubectl delete deployment api-deployment
```

## What you learned

- `envFrom` loads all variables from a ConfigMap or Secret at once
- Pods do not restart automatically when a ConfigMap changes
- The Docker image is generic — configuration comes from outside
