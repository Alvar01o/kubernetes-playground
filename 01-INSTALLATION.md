# Kubernetes Playground — Installation Guide

A local Kubernetes environment using [Minikube](https://minikube.sigs.k8s.io/).

---

## Prerequisites

- Docker installed and running
- Git

---

## 1. Install Minikube

Download and install the Minikube binary:

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

---

## 2. Start the Cluster

Start Minikube using the Docker driver (recommended for Linux):

```bash
minikube start --driver=docker
```

Verify the cluster is running:

```bash
minikube status
kubectl get nodes
```

---

## 3. Set Up kubectl

Minikube ships with a bundled `kubectl`. Create an alias so you can use it as a regular command:

```bash
alias kubectl="minikube kubectl --"
```

> To make this permanent, add the alias to your `~/.bashrc` or `~/.zshrc`.

---

## 4. Deploy a Sample Application

Create a test deployment using the echo-server image:

```bash
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
```

Expose the deployment as a NodePort service on port 8080:

```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

Check the service was created:

```bash
kubectl get services hello-minikube
```

---

## 5. Access the Application

**Option A — Open via Minikube** (opens the service URL in your browser):

```bash
minikube service hello-minikube
```

**Option B — Port-forward** (maps the service to `localhost:7080`):

```bash
kubectl port-forward service/hello-minikube 7080:8080
# Then visit http://localhost:7080
```

---

## 6. Inspect All Running Pods

```bash
kubectl get po -A
```
