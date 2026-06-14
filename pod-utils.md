# Pod Useful Commands

Quick reference for managing Kubernetes pods with `kubectl`.

---

## Create & Apply

```bash
# Create a pod from a definition file
kubectl apply -f pod.yaml

# Create a pod imperatively (without a file)
kubectl run <pod-name> --image=<image>

# Dry-run: preview what would be created without applying it
kubectl apply -f pod.yaml --dry-run=client
```

---

## List & Inspect

```bash
# List all pods in the current namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods -A

# List pods and show which node they are scheduled on
kubectl get pods -o wide

# Watch pods in real time
kubectl get pods -w

# Show detailed info about a pod (events, volumes, conditions)
kubectl describe pod <pod-name>

# Output the full pod definition as YAML
kubectl get pod <pod-name> -o yaml
```

---

## Logs

```bash
# Print logs of a pod
kubectl logs <pod-name>

# Follow (stream) logs in real time
kubectl logs -f <pod-name>

# Show only the last N lines
kubectl logs <pod-name> --tail=50

# Print logs from a specific container (multi-container pods)
kubectl logs <pod-name> -c <container-name>

# Print logs from a previously crashed container
kubectl logs <pod-name> --previous
```

---

## Execute Commands

```bash
# Open an interactive shell inside a running pod
kubectl exec -it <pod-name> -- /bin/sh

# Run a one-off command without an interactive session
kubectl exec <pod-name> -- <command>
```

---

## Port Forwarding

```bash
# Forward a local port to a port inside the pod
kubectl port-forward pod/<pod-name> <local-port>:<pod-port>

# Example: access the test-app on localhost:3000
kubectl port-forward pod/test-app 3000:3000
```

---

## Delete

```bash
# Delete a pod by name
kubectl delete pod <pod-name>

# Delete using the same file used to create it
kubectl delete -f pod.yaml

# Force-delete a stuck pod immediately
kubectl delete pod <pod-name> --grace-period=0 --force
```

---

## Debugging

```bash
# Check why a pod is not starting (look at Events section)
kubectl describe pod <pod-name>

# Check resource usage (requires metrics-server)
kubectl top pod <pod-name>

# Copy a file from a pod to your local machine
kubectl cp <pod-name>:<path/in/pod> <local-path>

# Copy a file from your local machine into a pod
kubectl cp <local-path> <pod-name>:<path/in/pod>
```

---

## Namespace

```bash
# List pods in a specific namespace
kubectl get pods -n <namespace>

# Run any command against a specific namespace
kubectl <command> -n <namespace>
```
