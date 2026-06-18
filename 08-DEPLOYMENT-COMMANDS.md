# Deployment Useful Commands

Quick reference for managing Kubernetes Deployments with `kubectl`.

A Deployment manages a ReplicaSet underneath and adds **rolling updates**, **rollbacks**,
and **revision history** — making it the standard way to run stateless applications in production.

---

## Create & Apply

```bash
# Create a deployment from a file
kubectl apply -f test-app/deployment.yaml

# Create a deployment imperatively
kubectl create deployment <name> --image=<image>

# Dry-run: preview without applying
kubectl apply -f test-app/deployment.yaml --dry-run=client
```

---

## List & Inspect

```bash
# List all deployments
kubectl get deployments

# Show detailed info: replicas, strategy, events, conditions
kubectl describe deployment <name>

# Output the full deployment definition as YAML
kubectl get deployment <name> -o yaml

# List the ReplicaSets managed by the deployment
kubectl get rs -l app=test-app
```

---

## Scaling

```bash
# Scale to a specific number of replicas
kubectl scale deployment <name> --replicas=4

# Example
kubectl scale deployment test-app-deployment --replicas=4

# Verify
kubectl get pods -l app=test-app
```

---

## Rolling Updates

A rolling update gradually replaces old pods with new ones — zero downtime.
Triggered automatically when the pod template changes (e.g. a new image version).

```bash
# Update the container image — triggers a rolling update
kubectl set image deployment/<name> <container-name>=<new-image>

# Example: update test-app to a new version
kubectl set image deployment/test-app-deployment test-app=test-app:v2

# Watch the rollout progress in real time
kubectl rollout status deployment/<name>

# Describe rollout events
kubectl describe deployment <name>
```

**How it works by default:**

- Brings up new pods before terminating old ones
- Never goes below the desired replica count during the update
- Controlled by `RollingUpdate` strategy (default):

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # How many extra pods can exist during the update
    maxUnavailable: 0  # How many pods can be unavailable during the update
```

---

## Rollback

Deployments keep a revision history, so you can revert to any previous version.

```bash
# View rollout history
kubectl rollout history deployment/<name>

# View details of a specific revision
kubectl rollout history deployment/<name> --revision=2

# Roll back to the previous revision
kubectl rollout undo deployment/<name>

# Roll back to a specific revision
kubectl rollout undo deployment/<name> --to-revision=2
```

---

## Pause & Resume

Useful when you want to apply multiple changes without triggering a rollout for each one.

```bash
# Pause the deployment (stops new rollouts from being triggered)
kubectl rollout pause deployment/<name>

# Make your changes (e.g. update image, change env vars)
kubectl set image deployment/<name> <container-name>=<new-image>

# Resume — triggers a single rollout with all the changes
kubectl rollout resume deployment/<name>
```

---

## Restart

Force a rolling restart of all pods without changing the image or config:

```bash
kubectl rollout restart deployment/<name>
```

> Useful to pick up changes like a refreshed secret or configmap.

---

## Delete

```bash
# Delete the deployment (also removes the managed ReplicaSet and pods)
kubectl delete deployment <name>

# Delete using the definition file
kubectl delete -f test-app/deployment.yaml
```

---

## Deployment vs ReplicaSet — Summary

| Feature               | ReplicaSet | Deployment |
|-----------------------|------------|------------|
| Maintains N replicas  | ✅         | ✅         |
| Manual scaling        | ✅         | ✅         |
| Rolling updates       | ❌         | ✅         |
| Rollback              | ❌         | ✅         |
| Revision history      | ❌         | ✅         |
| Pause / Resume        | ❌         | ✅         |
| Zero-downtime updates | ❌         | ✅         |

> Use a Deployment in almost every case. Use a ReplicaSet directly only if you need
> fine-grained control over the update process that a Deployment does not expose.
