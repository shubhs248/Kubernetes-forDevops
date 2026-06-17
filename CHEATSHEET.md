# 📋 Kubernetes Quick-Revision Cheatsheet

A one-page reminder of the `kubectl` and YAML you use most. Use it alongside the exercises.

## The objects (what each one is for)
| Object | In one line |
|--------|-------------|
| **Pod** | The smallest unit — one (or a few) containers that run together. Usually you don't create these directly. |
| **ReplicaSet** | Keeps *N* identical Pods running. Managed for you by a Deployment. |
| **Deployment** | What you actually write. Manages ReplicaSets → Pods, and does rolling updates/rollbacks. |
| **Service** | A stable name + IP that load-balances to a set of Pods (Pods come and go; the Service stays). |
| **ConfigMap** | Non-secret config (env vars, files) kept out of your image. |
| **Secret** | Like a ConfigMap, but for sensitive values (base64-encoded, can be encrypted at rest). |
| **Namespace** | A folder to group and isolate objects. |

## Looking around (read-only, run these constantly)
```bash
kubectl get pods                      # list pods (-A = all namespaces, -o wide = more detail)
kubectl get pods,svc,deploy           # several kinds at once
kubectl get pods -w                   # watch live changes
kubectl describe pod <name>           # full detail + EVENTS (your #1 debugging tool)
kubectl logs <pod>                    # container output (-f follow, --previous = last crash)
kubectl get events --sort-by=.lastTimestamp   # what just happened in the cluster
kubectl explain deployment.spec       # built-in docs for any field
```

## Creating & changing things
```bash
kubectl apply -f file.yaml            # create or update from a manifest (the main way)
kubectl apply -f .                    # apply every manifest in this folder
kubectl delete -f file.yaml           # remove what the manifest created
kubectl scale deploy/web --replicas=5 # change the number of Pods
kubectl set image deploy/web app=nginx:1.27   # trigger a rolling update
kubectl rollout status deploy/web     # watch a rollout finish
kubectl rollout undo deploy/web       # roll back to the previous version
kubectl rollout history deploy/web    # see past revisions
```

## Getting inside / reaching the app
```bash
kubectl exec -it <pod> -- sh          # shell into a running container
kubectl port-forward svc/web 8080:80  # reach a Service from your laptop
kubectl run tmp --rm -it --image=busybox -- sh   # throwaway debug pod
```

## A minimal Deployment + Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello          # MUST match the Pod template labels below
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: web
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello            # routes to Pods with this label
  ports:
    - port: 80            # the Service port
      targetPort: 80      # the container port
```

## Health probes (add under the container)
```yaml
          readinessProbe:        # "ready to receive traffic?" — gates the Service
            httpGet: { path: /, port: 80 }
            initialDelaySeconds: 3
          livenessProbe:         # "still alive?" — if it fails, restart the container
            httpGet: { path: /, port: 80 }
            periodSeconds: 10
```

## Resources (requests = guaranteed, limits = ceiling)
```yaml
          resources:
            requests: { cpu: "100m", memory: "64Mi" }
            limits:   { cpu: "250m", memory: "128Mi" }
```

## Service types (how the app is reached)
- **ClusterIP** (default) — reachable only *inside* the cluster.
- **NodePort** — opens a port on every node (`30000–32767`); good for local clusters.
- **LoadBalancer** — asks the cloud for an external IP (AWS/GCP/Azure).

## The debugging loop (memorise this!)
1. `kubectl get pods` — what's the **STATUS**? (`Pending`, `ImagePullBackOff`, `CrashLoopBackOff`, `Running`)
2. `kubectl describe pod <name>` — read the **Events** at the bottom.
3. `kubectl logs <pod>` (and `--previous` if it crashed) — what did the app say?
4. Service returns nothing? `kubectl get endpoints <svc>` — empty means the **selector doesn't match** any Pod labels.

| Status you see | Usually means |
|----------------|---------------|
| `ImagePullBackOff` / `ErrImagePull` | wrong image name/tag, or registry auth |
| `CrashLoopBackOff` | container starts then exits/errors — check `logs --previous` |
| `Pending` | nowhere to schedule — not enough CPU/memory, or no node matches |
| `Running` but app unreachable | Service selector vs Pod labels, or wrong `targetPort` |

---

## ⭐ Found this useful?
Please **star** ⭐, **fork** 🍴, and **share** 🔗 this repo on LinkedIn so others can use it too. Want to improve it? See [CONTRIBUTING.md](CONTRIBUTING.md).

Made by **Shubham Sharma** · [GitHub](https://github.com/shubhs248) · [LinkedIn](https://www.linkedin.com/in/shubhs248)
