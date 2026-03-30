# 07 — PostgreSQL with persistent storage

## The problem it solves

Pods are ephemeral: if they die, their data is lost. A database needs
data to survive even if the Pod dies and gets recreated.

## The three resources in this YAML

### PersistentVolumeClaim (PVC)
The disk "request". You declare how much storage you need and K8s provisions it.

```
PVC (request)   →  K8s provisions  →  PV (actual disk)
"I need 1GB"                          disk on the node
```

### Deployment
The PostgreSQL Pod that mounts the PVC at `/var/lib/postgresql/data`.
Always `replicas: 1` — a `ReadWriteOnce` disk cannot be written by multiple Pods simultaneously.

### Service
Exposes PostgreSQL inside the cluster as `postgres-service:5432`.
Other Pods (like the API) use that name to connect.

## Dependencies

Requires the `db-credentials` Secret from exercise 05.

## PVC AccessModes

| Mode | Meaning |
|---|---|
| `ReadWriteOnce` | A single Pod can read and write (standard for databases) |
| `ReadOnlyMany` | Many Pods can read, none can write |
| `ReadWriteMany` | Many Pods can read and write (requires special storage) |

## Commands

```bash
# Create everything at once (PVC + Deployment + Service)
kubectl apply -f 07-postgres.yaml

# Check the disk status
kubectl get pvc

# Connect to postgres
kubectl exec -it $(kubectl get pod -l app=postgres -o jsonpath='{.items[0].metadata.name}') \
  -- psql -U admin -d taskflow

# Verify data persists after deleting the Pod
kubectl delete pod $(kubectl get pod -l app=postgres -o jsonpath='{.items[0].metadata.name}')
# The Deployment recreates the Pod → data is still in the PVC

# Delete everything (PVC must be deleted separately — K8s protects it)
kubectl delete deployment postgres
kubectl delete service postgres-service
kubectl delete pvc postgres-pvc
```

## What you learned

- The Pod can die, the disk (PVC) survives
- Databases always use `replicas: 1` with `ReadWriteOnce`
- Other services connect by DNS name: `postgres-service:5432`
- The PVC must be deleted explicitly to avoid accidental data loss
