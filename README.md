# Kubernetes Playground

A local Kubernetes environment built with [Minikube](https://minikube.sigs.k8s.io/) to practice deploying, exposing, and managing containerized applications.

---

## What's covered

| # | File | Description |
|---|------|-------------|
| 1 | [01-INSTALLATION.md](01-INSTALLATION.md) | Install Minikube, start the cluster, set up kubectl |
| 2 | [02-DOCKERIZING.md](02-DOCKERIZING.md) | Build and run the test-app Docker image locally |
| 3 | [03-DEPLOYMENT.md](03-DEPLOYMENT.md) | Deploy and expose an application inside Minikube |
| 4 | [04-POD-COMMANDS.md](04-POD-COMMANDS.md) | Useful kubectl commands to manage pods |
| 5 | [05-LOAD-BALANCING.md](05-LOAD-BALANCING.md) | Expose the app through a LoadBalancer Service |
| 6 | [06-TROUBLESHOOTING.md](06-TROUBLESHOOTING.md) | Common issues and fixes encountered along the way |
| 7 | [07-SCALING.md](07-SCALING.md) | Scale replicas up and down using a ReplicaSet |
| 8 | [08-DEPLOYMENT-COMMANDS.md](08-DEPLOYMENT-COMMANDS.md) | kubectl commands for Deployments: scaling, rolling updates, rollbacks |

---

## Stack

- **Minikube** — runs a single-node Kubernetes cluster locally using Docker as the driver
- **kubectl** — bundled with Minikube (`alias kubectl="minikube kubectl --"`)
- **Node.js** — minimal HTTP server returning a JSON response (see [test-app/](test-app/))
- **Docker** — used to build the app image before loading it into Minikube

---

## Quick Start

```bash
# 1. Start the cluster
minikube start --driver=docker

# 2. Build and load the test app image
cd test-app && docker build -t test-app . && cd ..
minikube image load test-app:latest

# 3. Deploy the pod
kubectl apply -f test-app/pod.yaml

# 4. Expose it via LoadBalancer
kubectl apply -f test-app/service-lb.yaml
minikube tunnel

# 5. Hit the app
curl http://$(kubectl get service test-app-lb -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
# {"message":"testing kubernetes"}
```

---

## Project Structure

```
kubernetes-playground/
├── test-app/
│   ├── index.js          # Node.js HTTP server
│   ├── package.json
│   ├── Dockerfile
│   ├── pod.yaml          # Pod definition
│   ├── replicaset.yaml   # ReplicaSet definition (2 replicas)
│   └── service-lb.yaml   # LoadBalancer Service definition
├── 01-INSTALLATION.md
├── 02-DOCKERIZING.md
├── 03-DEPLOYMENT.md
├── 04-POD-COMMANDS.md
├── 05-LOAD-BALANCING.md
├── 06-TROUBLESHOOTING.md
├── 07-SCALING.md
└── 08-DEPLOYMENT-COMMANDS.md
```

---

<sub>**Special thanks to [Claude](https://claude.ai)** for helping organize, clean, and document these Kubernetes learning notes — turning raw console history into structured, readable guides.</sub>
