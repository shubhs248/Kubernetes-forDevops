# Exercise 2.1 — A Pod, then a Deployment

**You write:** two manifests — first a single Pod, then the same app as a Deployment — and you'll *feel* why nobody runs bare Pods in production.

---

## 🧠 Why this matters
A **Pod** runs your container, but if it dies, it stays dead — nothing brings it back. A **Deployment** wraps the same Pod template in a controller that keeps the desired number running, replaces dead ones, and lets you roll out new versions. This exercise makes the difference concrete by having you break a Pod and watch nothing happen, then break a Deployment's Pod and watch it heal.

---

## 📋 Part A — a bare Pod

Write `my-pod.yaml` for a single Pod:

- `kind: Pod`, `apiVersion: v1`
- name: `hello-pod`
- label: `app: hello`
- one container named `web`, image `nginx:1.27-alpine`, container port `80`

Apply and test the "no self-healing" behaviour:

```bash
kubectl apply -f my-pod.yaml
kubectl get pods
kubectl delete pod hello-pod     # it's gone — and nothing recreates it
kubectl get pods                 # empty. That's the problem with bare Pods.
```

## 📋 Part B — the same app as a Deployment

Write `my-deployment.yaml`:

- `kind: Deployment`, `apiVersion: apps/v1`
- name: `hello`
- `replicas: 2`
- a **selector** that matches Pods with label `app: hello`
- a Pod **template** whose Pods also carry the label `app: hello` (the selector and template labels MUST match)
- container `web`, image `nginx:1.27-alpine`, container port `80`

Apply and test the "self-healing" behaviour:

```bash
kubectl apply -f my-deployment.yaml
kubectl get pods                       # two hello-xxxx Pods
kubectl delete pod <one-hello-pod>     # delete one...
kubectl get pods                       # ...a replacement appears within seconds
```

## ✅ How to check
```bash
kubectl get deploy hello               # READY should read 2/2
kubectl describe deploy hello          # see the ReplicaSet it created
```
Compare with [`solutions/pod.yaml`](solutions/pod.yaml) and [`solutions/deployment.yaml`](solutions/deployment.yaml).

## 💡 Hints
- The three places labels appear in a Deployment: `spec.selector.matchLabels`, and `spec.template.metadata.labels`. The selector matches the template — if they differ, `apply` will reject it.
- `replicas` lives directly under `spec`, not under `template`.
- The container `spec` inside `template.spec.containers` is *identical* to what you'd put in a bare Pod — a Deployment just wraps a Pod template.

## 🧹 Clean up
```bash
kubectl delete -f my-deployment.yaml
```
