# Exercise 3: Scaling a Flask App on a Single Node using ReplicaSets

## Use Case: E-commerce Flash Sale
During a flash sale (like Big Billion Days), traffic suddenly spikes. A single Pod would crash under the load. A **ReplicaSet** runs multiple identical Pods of the same app and spreads requests across them, then scales back down after the sale.

## Objective
- Understand ReplicaSets and Pods
- Scale the Flask app up from 3 to 5 replicas
- Observe self-healing when a Pod is deleted
- Observe load distribution across Pods

## Files
| File | Purpose |
|------|---------|
| `app.py` | Flash sale Flask app with `/`, `/buy` and `/health` endpoints; responses show which Pod served the request |
| `Dockerfile` | Builds the app image using gunicorn on port 5000 |
| `flashsale-replicaset.yaml` | ReplicaSet (3 replicas, readiness/liveness probes, resource limits) + ClusterIP Service |

## Steps and Commands

### 1. Check the single node
```bash
kubectl get nodes
```

### 2. Build the image inside Minikube
```bash
minikube image build -t flashsale:1.0 .
```

### 3. Create the ReplicaSet and Service
```bash
kubectl apply -f flashsale-replicaset.yaml
kubectl get pods
kubectl get rs
```

### 4. Scale up to 5 replicas
```bash
kubectl scale rs flashsale-rs --replicas=5
kubectl get rs
kubectl get pods
```

### 5. Delete a Pod and observe self-healing
```bash
kubectl delete pod <pod-name>
kubectl get pods -o wide
```

### 6. Send buy requests and observe load distribution
```bash
kubectl run buyer --rm -i --restart=Never --image=curlimages/curl -- sh -c 'sleep 5; for i in 1 2 3 4 5 6; do curl -s http://flashsale-svc/buy; echo; done'
```

## Output Screenshots

### ReplicaSet created with 3 Pods
![3 Replicas](screenshots/1.png)

### Scaled up to 5 Pods
![5 Replicas](screenshots/2.png)

### Deleted Pod automatically replaced (self-healing)
![Self Healing](screenshots/3.png)

### Buy requests served by different Pods (load distribution)
![Load Distribution](screenshots/4.png)

## Q&A
1. **Initial number of replicas?** 3
2. **Pods running after applying the ReplicaSet?** 3
3. **What happens when scaled to 5?** Kubernetes creates 2 more Pods to reach the desired count of 5.
4. **What happens when one Pod is deleted?** The ReplicaSet automatically creates a new Pod to keep 5 running.
5. **How does Kubernetes maintain replicas?** It continuously compares the actual Pod count with the desired count and creates or deletes Pods to match.
6. **How many nodes are running?** 1 (minikube)
7. **Where are the Pods running?** All 5 Pods run on the single `minikube` node.

## Learnings
- **Pod distribution:** each Pod is an identical worker; scaling means creating clones.
- **Resiliency:** a failed Pod is replaced automatically, so users don't notice downtime.
- **Efficiency:** add Pods when demand spikes, remove them when demand drops.
- **Load balancing:** the Service spreads requests across all healthy Pods.
