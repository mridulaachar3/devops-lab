# Exercise 10: Multi-Node Kubernetes Cluster with Multiple Applications and ReplicaSets

## Storyboard
An e-commerce platform with two services:
1. **Product Catalog** (AppA): lists available products, **2 replicas**
2. **Shopping Cart** (AppB): manages user carts, **3 replicas**

Replicas of each service are spread across **different nodes** using pod anti-affinity, so the store stays available even if one node fails.

## Files
| File | Purpose |
|------|---------|
| `product_catalog.py` | Flask app: `GET /products` |
| `shopping_cart.py` | Flask app: `GET /cart` and `POST /cart` |
| `Dockerfile.product` | Image for Product Catalog |
| `Dockerfile.shopping` | Image for Shopping Cart |
| `product_catalog_deployment.yaml` | Deployment, 2 replicas, pod anti-affinity |
| `shopping_cart_deployment.yaml` | Deployment, 3 replicas, pod anti-affinity |
| `product_catalog_service.yaml` | NodePort Service for Product Catalog |
| `shopping_cart_service.yaml` | NodePort Service for Shopping Cart |

## Steps and Commands

### 1. Start a 3-node Minikube cluster
~~~bash
minikube delete
minikube start --nodes 3 -p devops-multinode --driver=docker --memory=2048 --cpus=2
kubectl get nodes
~~~

### 2. Build the images
~~~bash
docker build -t product-catalog:latest -f Dockerfile.product .
docker build -t shopping-cart:latest -f Dockerfile.shopping .
~~~

### 3. Load the images into all nodes
~~~bash
minikube -p devops-multinode image load product-catalog:latest
minikube -p devops-multinode image load shopping-cart:latest
minikube -p devops-multinode image ls | grep -E "product-catalog|shopping-cart"
~~~

### 4. Deploy the apps and services
~~~bash
kubectl apply -f product_catalog_deployment.yaml
kubectl apply -f shopping_cart_deployment.yaml
kubectl apply -f product_catalog_service.yaml
kubectl apply -f shopping_cart_service.yaml
kubectl get pods -o wide
~~~

### 5. Test Product Catalog
~~~bash
minikube -p devops-multinode service product-catalog-service --url
~~~
Open `<url>/products` in the browser.

### 6. Test Shopping Cart (GET and POST)
~~~bash
kubectl run cart-test --rm -i --restart=Never --image=curlimages/curl -- sh -c 'sleep 5; curl -s http://shopping-cart-service/cart; echo; curl -s -X POST http://shopping-cart-service/cart -H "Content-Type: application/json" -d "{\"id\": 1, \"name\": \"Laptop\", \"quantity\": 1}"; echo'
~~~

## Result: Pod Distribution
| App | Replicas | Nodes |
|-----|----------|-------|
| product-catalog | 2 | devops-multinode-m02, devops-multinode-m03 |
| shopping-cart | 3 | devops-multinode, devops-multinode-m02, devops-multinode-m03 |

Each replica of the same app runs on a different node because of `podAntiAffinity` with `topologyKey: kubernetes.io/hostname`.

## Changes from the Lab Manual
- The Shopping Cart deployment uses namespace `default` (the manual's `devops-exercise` namespace doesn't exist and doesn't match the Service's namespace).
- Minikube here uses the containerd runtime, so images are loaded with `minikube image load` and checked with `minikube image ls` instead of the registry add-on and `minikube ssh -- docker images`.
- `--memory=2048` keeps the 3 nodes within laptop memory; `--force` isn't needed because Minikube isn't run as root.
- Shopping Cart is tested with `curl` from a temporary pod inside the cluster.

## Output Screenshots

### 3 nodes ready
![Nodes](screenshots/1.png)

### Images built
![Docker Images](screenshots/2.png)

### Images loaded into Minikube
![Images Loaded](screenshots/3.png)

### Pods distributed across nodes
![Pod Distribution](screenshots/4.png)

### Product Catalog in browser
![Product Catalog](screenshots/5.png)

### Shopping Cart GET and POST
![Shopping Cart](screenshots/6.png)

## Summary
Two Python applications were deployed on a multi-node Minikube cluster with replicas, with each Pod of the same app running on a different node. This maximises availability and fault tolerance across the cluster.
