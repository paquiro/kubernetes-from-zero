# 04 — ConfigMap

## What is a ConfigMap

Stores non-sensitive configuration variables outside of the Docker image.

**Without ConfigMap:** every configuration change requires rebuilding the image.
**With ConfigMap:** the image is generic, configuration is injected when the Pod starts.

## What this YAML does

Defines three environment variables used by the API:
- `NODE_ENV` — runtime environment
- `API_PORT` — port the server listens on
- `DB_HOST` — DNS name of the PostgreSQL Service

## How to use it in a Deployment

```yaml
envFrom:
  - configMapRef:
      name: api-config    # loads ALL variables from the ConfigMap
```

Or variable by variable:

```yaml
env:
  - name: NODE_ENV
    valueFrom:
      configMapKeyRef:
        name: api-config
        key: NODE_ENV
```

## Commands

```bash
# Create
kubectl apply -f 04-configmap.yaml

# List ConfigMaps
kubectl get configmaps

# View the contents
kubectl describe configmap api-config

# Delete
kubectl delete configmap api-config
```

## What you learned

- The Docker image doesn't change when you update configuration
- You can have different ConfigMaps for dev/staging/prod using the same image
- For sensitive values (passwords) use Secret, not ConfigMap
