# Exercise 3.1 — Scaling, Rolling Updates & Rollback

**You do:** scale a Deployment up and down, push a new version with a rolling update, then roll back a "bad" version — all with zero downtime.

---

## 🧠 Why this matters
This is the day-to-day of running apps on Kubernetes. A **rolling update** replaces Pods a few at a time so the app stays up during a deploy; if the new version is broken, **rollback** instantly returns to the last good one. Interviewers love "how do you deploy a new version with no downtime, and what if it's broken?" — this exercise *is* that answer.

### What a rolling update actually does
When you change the image, the Deployment creates a **new ReplicaSet** for the new version and slowly shifts Pods from old to new, obeying two settings:
- `maxUnavailable` — how many Pods may be down during the rollout.
- `maxSurge` — how many *extra* Pods it may create above the desired count.

Because the old ReplicaSet is kept (scaled to 0), rollback is just "scale the old one back up" — which `kubectl rollout undo` does for you.

---

## 📋 Steps

Start from a running Deployment (use [`solutions/deployment-rollout.yaml`](solutions/deployment-rollout.yaml), which is `hello` with 3 replicas and an explicit rollout strategy):

```bash
kubectl apply -f solutions/deployment-rollout.yaml
kubectl get pods -l app=hello
```

**1) Scale.**
```bash
kubectl scale deployment hello --replicas=5
kubectl get pods -l app=hello -w     # watch 2 more appear, then Ctrl-C
kubectl scale deployment hello --replicas=3
```

**2) Rolling update to a new version.** Change the image and watch it roll:
```bash
kubectl set image deployment/hello web=nginx:1.27
kubectl rollout status deployment/hello       # waits until the rollout completes
kubectl get rs -l app=hello                   # you'll see old + new ReplicaSets
```

**3) Ship a "bad" version and roll back.** Point at an image tag that doesn't exist:
```bash
kubectl set image deployment/hello web=nginx:does-not-exist
kubectl rollout status deployment/hello       # it stalls — new Pods can't pull
kubectl get pods -l app=hello                 # some Pods in ImagePullBackOff
kubectl rollout undo deployment/hello         # back to the last good version
kubectl rollout status deployment/hello       # healthy again
```

**4) See the history.**
```bash
kubectl rollout history deployment/hello
```

## ✅ What success looks like
- Scaling changes the Pod count within seconds.
- The good rollout never drops below a working app.
- The bad rollout leaves the *old* Pods running (that's the safety net), and `undo` restores full health.

## 💡 Hints
- `kubectl rollout status` is your "is the deploy done?" command — great in CI/CD pipelines.
- A rollout only triggers when the Pod **template** changes (e.g. image, env). Editing unrelated metadata won't roll.
- `kubectl rollout undo deployment/hello --to-revision=2` rolls back to a specific revision.
- Add `kubectl annotate deployment/hello kubernetes.io/change-cause="..."` (or use `--record` on older clusters) to label revisions in history.

## 🧹 Clean up
```bash
kubectl delete -f solutions/deployment-rollout.yaml
```
