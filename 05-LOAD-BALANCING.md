# Load Balancing the Test App

Exposing the `test-app` pod behind a Kubernetes LoadBalancer Service in Minikube.

> **Prerequisite:** `test-app` pod is running. See [01-INSTALLATION.md](01-INSTALLATION.md) and [03-DEPLOYMENT.md](03-DEPLOYMENT.md).

---

## 1. Apply the LoadBalancer Service

```bash
kubectl apply -f test-app/service-lb.yaml
```

---

## 2. Start the Minikube Tunnel

Minikube does not assign an external IP to LoadBalancer services automatically.
The tunnel simulates a cloud provider's load balancer by creating a network route
from your machine into the cluster.

Run this in a **dedicated terminal and keep it alive**:

```bash
minikube tunnel
```

---

## 3. Get the External IP

```bash
kubectl get service test-app-lb

# NAME          TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
# test-app-lb   LoadBalancer   10.103.95.91   10.103.95.91   80:30120/TCP   116s
```

> In Minikube, the `EXTERNAL-IP` matches the `CLUSTER-IP`. In a real cloud provider
> (AWS/GCP/Azure) it would be a public IP or hostname assigned by the provider.

---

## 4. Access the App Through the LoadBalancer

```bash
curl http://10.103.95.91
# {"message":"testing kubernetes"}
```

The app is now reachable on port `80` through the LoadBalancer — no need to know
that the container listens on port `3000` internally.

---

## 5. Access on localhost:80 (Optional)

Port 80 is a privileged port on Linux, so `sudo` is required. Because `kubectl` is
an alias that `sudo` does not inherit, and because `sudo` runs as root (losing your
kubeconfig), use `-E` to preserve your environment:

```bash
sudo -E minikube kubectl -- port-forward service/test-app-lb 80:80
```

```bash
curl http://localhost
# {"message":"testing kubernetes"}
```

---

## How It Works

```
Your machine :80  →  LoadBalancer (minikube tunnel)  →  Service :80  →  Pod :3000
```

The `service-lb.yaml` defines the mapping:

| Field        | Value | Meaning                              |
|--------------|-------|--------------------------------------|
| `port`       | 80    | Port exposed by the LoadBalancer     |
| `targetPort` | 3000  | Port the container actually listens on |
| `selector`   | `app: test-app` | Matches the pod label        |
