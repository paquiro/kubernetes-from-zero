# 05 — Secret

## What is a Secret

Like a ConfigMap but for sensitive data: passwords, tokens, API keys.
Values are stored in base64.

## Important warning

**Base64 is not encryption.** Anyone with cluster access can decode it:

```bash
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

Native K8s Secrets are meant to separate secrets from code, not to encrypt them.
In real production environments use AWS Secrets Manager, HashiCorp Vault, or Sealed Secrets.

## How to generate base64 values

```bash
echo -n "my-password" | base64     # encode
echo -n "cGFzc3dvcmQxMjM=" | base64 -d  # decode
```

## How to use it in a Deployment

```yaml
envFrom:
  - secretRef:
      name: db-credentials    # loads ALL variables from the Secret
```

Or variable by variable:

```yaml
env:
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: DB_PASSWORD
```

K8s automatically decodes the base64 — the Pod receives the plain value.

## Commands

```bash
# Create
kubectl apply -f 05-secret.yaml

# List secrets (values appear obfuscated)
kubectl get secrets
kubectl describe secret db-credentials

# View a value in plain text (careful!)
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 -d

# Delete
kubectl delete secret db-credentials
```

## What you learned

- Secrets are base64, not real encryption
- Production environments need an external solution (Vault, AWS Secrets Manager)
- The Pod receives decoded values automatically
