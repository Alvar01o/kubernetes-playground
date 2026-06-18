# Scaling with ReplicaSet

Managing replica count for `test-app` using a Kubernetes ReplicaSet.

> **Prerequisite:** Image is loaded in Minikube and the ReplicaSet definition exists.
> See [02-DOCKERIZING.md](02-DOCKERIZING.md) and [test-app/replicaset.yaml](test-app/replicaset.yaml).

---

## 1. Create the ReplicaSet

```bash
kubectl create -f test-app/replicaset.yaml
```

> If a standalone `test-app` pod is still running, delete it first — the ReplicaSet
> uses the same label (`app: test-app`) and the selector conflict will block scheduling:
> ```bash
> kubectl delete pod test-app
> ```

---

## 2. Inspect a Pod

Check the status of any individual replica (useful to catch `ErrImageNeverPull` early):

```bash
kubectl describe pod <pod-name>
```

---

## 3. List ReplicaSets

```bash
kubectl get rs

# NAME          DESIRED   CURRENT   READY   AGE
# test-app-rs   2         2         2       30s
```

> `kubectl get rc` lists ReplicationControllers — the older primitive. Use `kubectl get rs`
> for ReplicaSets.

---

## 4. Scale Up

```bash
kubectl scale --replicas=4 -f test-app/replicaset.yaml
```

Verify all pods are running:

```bash
kubectl get pods -l app=test-app

# NAME                READY   STATUS    RESTARTS   AGE
# test-app-rs-2rmr8   1/1     Running   0          2m
# test-app-rs-5xkpq   1/1     Running   0          10s
# test-app-rs-9bnvt   1/1     Running   0          10s
# test-app-rs-lqw7c   1/1     Running   0          10s
```

---

## 5. Scale Down

When scaling by name (without a file), prefix the name with the resource type `rs/`:

```bash
kubectl scale --replicas=1 rs/test-app-rs
```

Verify only one pod remains:

```bash
kubectl get pods -l app=test-app

# NAME                READY   STATUS    RESTARTS   AGE
# test-app-rs-2rmr8   1/1     Running   0          5m
```

> The LoadBalancer Service automatically routes traffic to however many replicas
> are currently running — no changes to `service-lb.yaml` needed when scaling.
