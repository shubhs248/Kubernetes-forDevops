# Exercise 2.2 — A Service (and how it finds Pods)

**You write:** a Service that load-balances to your `hello` Deployment, and you'll learn the label-selector link that trips up almost everyone.

---

## 🧠 Why this matters
Pods are disposable and keep changing IPs, so nothing should ever talk to a Pod directly. A **Service** is a stable name and IP that automatically forwards traffic to whatever Pods currently match its **selector**. The magic — and the #1 source of "my Service returns nothing" bugs — is that the Service finds Pods purely by **matching labels**. Get the labels right and traffic flows; get them wrong and the Service points to empty space.

---

## 📋 The spec
First make sure the `hello` Deployment from Exercise 1 is running (its Pods have label `app: hello`).

Write `my-service.yaml`:

- `kind: Service`, `apiVersion: v1`
- name: `hello`
- `selector` that matches `app: hello` (this is how it finds the Pods)
- one port: `port: 80` (the Service's own port) and `targetPort: 80` (the container's port)
- type: leave it default (`ClusterIP`) — we'll reach it with `port-forward`

## ✅ How to check
```bash
kubectl apply -f my-service.yaml
kubectl get svc hello

# THE KEY CHECK: does the Service actually point to Pods?
kubectl get endpoints hello
#   You should see 2 IP:port pairs (one per Pod). EMPTY = your selector is wrong.

# Reach the app from your laptop
kubectl port-forward svc/hello 8080:80
#   open http://localhost:8080  (you should see the nginx welcome page)
```
Compare with [`solutions/service.yaml`](solutions/service.yaml).

## 🔬 Prove the selector matters (do this!)
Edit your Service's selector to `app: nope`, re-apply, and run `kubectl get endpoints hello` again — it's now **empty**, and `port-forward` returns errors. This is *exactly* what a broken Service looks like in real life. Change it back to `app: hello` and the endpoints return. Remembering this one experiment will save you in an interview.

## 💡 Hints
- `port` vs `targetPort`: `port` is what clients call on the Service; `targetPort` is the port the container listens on. They're often the same number but don't have to be.
- A Service selector is a *flat map* (`app: hello`), unlike a Deployment selector which nests under `matchLabels`.
- `ClusterIP` is internal-only. On a local cluster, `port-forward` is the simplest way to reach it. (NodePort is covered in the cheatsheet.)

## 🧹 Clean up
```bash
kubectl delete -f my-service.yaml
```
