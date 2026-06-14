# Troubleshooting

Common issues and fixes for this Kubernetes playground.

---

## Port-forward fails: pod is not running

**Symptoms:**
```
error: unable to forward port because pod is not running. Current status=Pending
```
```
NAME       READY   STATUS              RESTARTS   AGE
test-app   0/1     ErrImageNeverPull   0          9m
```

**Cause:**  
The pod fails because `imagePullPolicy: Never` tells Kubernetes never to pull from a registry — it must find the image already present inside the cluster. The image was built in your local Docker daemon but Minikube runs its own isolated Docker daemon, so the image doesn't exist inside the cluster.

**Fix:**

```bash
# 1. Load the local image into Minikube's internal Docker daemon
minikube image load test-app:latest

# 2. Confirm the image is now available inside Minikube
minikube image ls | grep test-app

# 3. Delete the stuck pod and recreate it
kubectl delete pod test-app
kubectl apply -f test-app/pod.yaml

# 4. Wait until the pod is Running
kubectl get pod test-app -w

# 5. Now port-forward will work
kubectl port-forward pod/test-app 3000:3000
```

**Expected output after applying the fix:**

```bash
# Before the fix — pod stuck with ErrImageNeverPull
kubectl get pods
# NAME                              READY   STATUS              RESTARTS   AGE
# hello-minikube-6fcb694474-qlznk   1/1     Running             0          63m
# test-app                          0/1     ErrImageNeverPull   0          9m8s

# Load the image into Minikube
minikube image load test-app:latest

# Delete the stuck pod
kubectl delete pod test-app
# pod "test-app" deleted from default namespace

# Recreate it
kubectl apply -f test-app/pod.yaml
# pod/test-app created

# Watch it come up — should reach Running within seconds
kubectl get pod test-app -w
# NAME       READY   STATUS    RESTARTS   AGE
# test-app   1/1     Running   0          19s

# Port-forward now works
kubectl port-forward test-app 3000:3000
# Forwarding from 127.0.0.1:3000 -> 3000
# Forwarding from [::1]:3000 -> 3000
# Handling connection for 3000
```

**Verify the app is responding:**
```bash
curl http://localhost:3000
# {"message":"testing kubernetes"}
```

> Any time you rebuild the image locally, you need to run `minikube image load` again
> so Minikube picks up the updated version.

---

## sudo kubectl: command not found

**Error:**
```
sudo: kubectl: command not found
```

**Cause:**  
`kubectl` is defined as a shell alias (`alias kubectl="minikube kubectl --"`). `sudo`
does not inherit shell aliases, so it cannot find the command.

**Fix — use `-E` to preserve your environment:**

```bash
sudo -E minikube kubectl -- port-forward service/test-app-lb 80:80
```

> `-E` tells `sudo` to keep the current user's environment variables, including
> `$HOME` and `$KUBECONFIG`, so Minikube can find the cluster config at `~/.kube/config`.

**Why not `sudo minikube kubectl --` without `-E`?**  
Without `-E`, `sudo` switches to root and looks for the kubeconfig at `/root/.kube/config`,
which does not exist, resulting in:
```
error: no server found for cluster "minikube"
```

---
