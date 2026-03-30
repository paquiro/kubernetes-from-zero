# 03 — Service

## What is a Service

A stable IP and DNS name that always points to a group of Pods.

Pods have IPs that change every time they are recreated. The Service solves that:
you always call the same name (`nginx-service`) and it load-balances across available Pods.

```
Your API → "call postgres-service:5432"
                    ↓
          Service (fixed IP, stable DNS name)
                    ↓
        load-balances across Pod1 / Pod2 / Pod3
```

## What this YAML does

Exposes Pods with label `app: nginx` on port 80 inside the cluster.

## How it connects to the Deployment

There is no explicit reference to the Deployment. The connection is made through **labels**:

- The Deployment creates Pods with `labels: app: nginx`
- The Service selects Pods with `selector: app: nginx`

Any Pod with that label receives traffic, regardless of which Deployment created it.

## Service types

| Type | Purpose |
|---|---|
| `ClusterIP` | Only accessible inside the cluster (inter-service communication) |
| `NodePort` | Exposes a port on the node (testing, not production) |
| `LoadBalancer` | Creates an external load balancer in cloud environments (AWS, GCP...) |

## Commands

```bash
# Create
kubectl apply -f 03-service.yaml

# List services
kubectl get services

# Test from inside the cluster (using DNS name)
kubectl run test --image=curlimages/curl --restart=Never --rm -it -- curl http://nginx-service

# Delete
kubectl delete service nginx-service
```

## What you learned

- The Service doesn't know how many Pods there are — that's the Deployment's job
- Inside the cluster you can call any Service by its name
- ClusterIP is the default type and the most commonly used
