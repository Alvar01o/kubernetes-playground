# Minikube — Deploying an Application

Guide to deploying and exposing a sample application on a local Minikube cluster.

> **Prerequisite:** Minikube is running. See [installation.md](installation.md) if not.

---

## 1. Create the Deployment

Deploy the echo-server image. Use `registry.k8s.io` — the old `k8s.gcr.io` registry is deprecated and may cause image pull failures:

```bash
kubectl create deployment hello-minikube --image=registry.k8s.io/echoserver:1.10
```

Verify the pod is running (status should reach `Running`):

```bash
kubectl get pods
```

---

## 2. Expose the Deployment

Create a NodePort service to make the deployment reachable:

```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

---

## 3. Access the Application

**Option A — Open via Minikube** (resolves the NodePort URL automatically):

```bash
minikube service hello-minikube
```

**Option B — Port-forward** (maps the service to `localhost:8080`):

```bash
kubectl port-forward service/hello-minikube 8080:8080
# Then visit http://localhost:8080
```

---

## 4. Teardown

Remove the deployment and service when done:

```bash
kubectl delete deployment hello-minikube
kubectl delete service hello-minikube
```

---

## Troubleshooting

If a pod is stuck in `ImagePullBackOff` or `ErrImagePull`, inspect it:

```bash
kubectl describe pod <pod-name>
```

The most common cause is using the deprecated `k8s.gcr.io` registry. Replace it with `registry.k8s.io` in your deployment image reference.
