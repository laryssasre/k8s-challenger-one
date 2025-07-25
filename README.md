# Kubernetes Challenge 01 – K3D Fundamentals

This project is part of a local hands-on Kubernetes learning journey using [K3D](https://k3d.io/) on local machine.
The goal is to understand how to deploy, expose, and organize applications in Kubernetes clusters from scratch.

---

## What This Challenge Covers

- Setting up a local Kubernetes cluster with K3D
- Creating isolated environments (namespaces)
- Enforcing resource usage limits
- Deploying a containerized app (NGINX)
- Exposing the app using a Service and Ingress
- Understanding key K8s building blocks in practice

---
## ⚙️ Requirements

- Docker
- k3d
- kubectl
- Helm

---

## 🚀 How to Run

### 1. Create the cluster

```bash
k3d cluster create challenger-k8s-one --agents 2 --port "88:80@loadbalancer"
```

### 2. Apply the manifests

```bash
kubectl apply -f manifests/
```

### 3. Install Ingress Controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

### 4. Access the App

Open in your browser:

```
http://localhost:88
```

You should see the NGINX welcome page.

---

## You're Done!

Use `kubectl get all -n dev` or `k9s` to explore your resources.

---

## Component Explanations (Line-by-Line)

### 📁 `namespace.yaml`

```yaml
apiVersion: v1           # API version (basic and stable)
kind: Namespace          # Declares this is a Namespace
metadata:
  name: dev              # Name of the namespace
```

Creates a new isolated environment inside the cluster called `dev`.

---

### 📁 `resourcequota.yaml`

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

Limits how much CPU and memory resources can be used inside the `dev` namespace and forces Pods to declare those limits.

---

### 📁 `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

Creates a NGINX container managed by a Deployment that ensures 1 replica is always running with resource limits defined.

---

### 📁 `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

Exposes the NGINX Pod on an internal IP that other K8s resources (like Ingress) can use.

---

### 📁 `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

Defines an Ingress route to expose the Service through your browser at `http://localhost:88`.

---
