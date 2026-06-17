# Exercise 3.2 — Health Probes

**You write:** a Deployment with **readiness** and **liveness** probes, and you'll see how Kubernetes uses them to keep traffic flowing only to healthy Pods.

---

## 🧠 Why this matters
Kubernetes can't read your app's mind — it needs you to tell it "is this Pod healthy?". That's what **probes** do, and getting them right is the difference between graceful self-healing and an outage. This is a very common interview question: *"what's the difference between a liveness and a readiness probe?"*

### The three probes
| Probe | Question it answers | What Kubernetes does if it FAILS |
|-------|--------------------|----------------------------------|
| **readiness** | "Ready to receive traffic *right now*?" | Removes the Pod from the Service (no traffic) — but does **not** restart it |
| **liveness** | "Is this process still healthy, or wedged?" | **Restarts** the container |
| **startup** | "Has a slow-starting app finished booting?" | Holds off liveness/readiness until it passes (protects slow starters) |

The classic mistake: using a liveness probe where you needed a readiness probe. If your app is just *temporarily busy*, a too-aggressive liveness probe will **restart it in a loop** instead of simply pausing traffic. Rule of thumb: **readiness gates traffic, liveness restarts.**

---

## 📋 The spec
Write `my-probes.yaml`: a Deployment `hello-probes` (image `nginx:1.27-alpine`, 2 replicas, label `app: hello-probes`) where the container has:

- a **readinessProbe**: HTTP GET `/` on port `80`, `initialDelaySeconds: 3`, `periodSeconds: 5`
- a **livenessProbe**: HTTP GET `/` on port `80`, `initialDelaySeconds: 10`, `periodSeconds: 10`, `failureThreshold: 3`

(nginx serves `/` with HTTP 200, so both probes pass — perfect for seeing healthy behaviour.)

## ✅ How to check
```bash
kubectl apply -f my-probes.yaml
kubectl get pods -l app=hello-probes
#   READY shows 1/1 only AFTER the readiness probe passes (watch it flip from 0/1)

kubectl describe pod <pod>      # the Events show probe activity / restarts
```

## 🔬 Make a probe fail on purpose (great for understanding)
Point the liveness probe at a path that returns 404 (e.g. `/healthz-nope`) and re-apply. Watch:
```bash
kubectl get pods -l app=hello-probes -w
#   RESTARTS climbs as the liveness probe fails 3x and Kubernetes restarts the container
```
Now do the same with the **readiness** probe instead: the Pod keeps running (no restarts) but `READY` stays `0/1` and the Service stops sending it traffic. Feeling that difference is the whole lesson. Revert to `/` when done.

Compare with [`solutions/deployment-probes.yaml`](solutions/deployment-probes.yaml).

## 💡 Hints
- Probes go **inside the container spec**, next to `image` and `ports`.
- `initialDelaySeconds` gives the app time to boot before the first check — too low and a slow app gets killed before it's up (that's what `startupProbe` is for).
- Besides `httpGet`, probes can be `tcpSocket` (just check a port opens) or `exec` (run a command; exit 0 = healthy).
- `failureThreshold` is how many fails in a row count as "failed"; `periodSeconds` is how often it checks.

## 🧹 Clean up
```bash
kubectl delete -f my-probes.yaml
```
