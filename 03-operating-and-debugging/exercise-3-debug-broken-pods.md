# Exercise 3.3 — Debug Broken Pods 🔧

**You do:** apply three deliberately broken manifests, diagnose each one using only `kubectl`, and fix it. This is the most interview-relevant exercise in the whole lab — being able to *systematically* debug is what separates "I did a tutorial" from "I can run this in production".

> **Rule: diagnose before you peek.** For each one, find the root cause yourself using the debugging loop, write down the cause in one sentence, *then* check against the answer.

---

## 🧠 The debugging loop (use it every time)
1. `kubectl get pods` → read the **STATUS**.
2. `kubectl describe pod <name>` → read the **Events** at the bottom.
3. `kubectl logs <pod>` (`--previous` if it crashed) → what did the app say?
4. Service problem? `kubectl get endpoints <svc>` → empty = label mismatch.

---

## 🐛 Broken #1 — `broken/broken-1-imagepullbackoff.yaml`
```bash
kubectl apply -f broken/broken-1-imagepullbackoff.yaml
kubectl get pods                 # what STATUS?
kubectl describe pod <name>      # read the Events
```
**Your job:** figure out why the Pod never runs, and fix the manifest so it does.

<details>
<summary>💡 Hint</summary>

The STATUS is `ImagePullBackOff` / `ErrImagePull`. That always means Kubernetes tried and failed to download the image. The Events line "Failed to pull image ... not found" points right at the cause. Look very closely at the image **tag**.
</details>

<details>
<summary>✅ Answer</summary>

The image tag is misspelled (`nginx:1.27-alpinnne`). There is no such tag, so the pull fails forever. Fix: correct it to `nginx:1.27-alpine`. See [`solutions/fixed-1-imagepullbackoff.yaml`](solutions/fixed-1-imagepullbackoff.yaml).

*Real-world version of this bug:* a private image with no `imagePullSecrets`, or a typo'd registry hostname. Same symptom, same debugging path.
</details>

---

## 🐛 Broken #2 — `broken/broken-2-crashloop.yaml`
```bash
kubectl apply -f broken/broken-2-crashloop.yaml
kubectl get pods                 # STATUS climbs in RESTARTS
kubectl logs <name>              # ...and --previous to see the last crash
```
**Your job:** explain why the container keeps restarting, and fix it so the Pod stays `Running`.

<details>
<summary>💡 Hint</summary>

The STATUS becomes `CrashLoopBackOff` and RESTARTS keeps climbing. That means the container *starts but then exits*. `describe` shows the container's last state was "Terminated" with an exit code; `kubectl logs <pod> --previous` shows what it printed right before dying.
</details>

<details>
<summary>✅ Answer</summary>

The container's command runs, prints a message, and then `exit 1` — so it dies immediately and Kubernetes keeps restarting it (with an increasing back-off delay, hence *CrashLoopBackOff*). The fix is to give it a command that **keeps running in the foreground** (a long-lived process). See [`solutions/fixed-2-crashloop.yaml`](solutions/fixed-2-crashloop.yaml).

*Real-world version:* the app crashes on boot because a required env var / config / dependency is missing. The fix in real life is "read the logs, supply what's missing" — but the *diagnosis path is identical*. A container whose main process exits = CrashLoopBackOff.
</details>

---

## 🐛 Broken #3 — `broken/broken-3-service-no-endpoints.yaml`
This file has a Deployment **and** a Service. The Pods run fine, but the Service returns nothing.
```bash
kubectl apply -f broken/broken-3-service-no-endpoints.yaml
kubectl get pods                 # Pods are Running and Ready (1/1)
kubectl get endpoints web        # ...but THIS is empty
kubectl port-forward svc/web 8080:80   # connecting fails / nothing to forward to
```
**Your job:** the Pods are healthy yet the Service is dead. Find out why and fix it.

<details>
<summary>💡 Hint</summary>

When Pods are healthy but a Service doesn't work, it's almost always a **label mismatch**. A Service routes by `selector`. Compare the Service's `selector` to the Deployment's Pod `template.metadata.labels`. `kubectl get endpoints web` being empty is the smoking gun — the Service matched zero Pods.
</details>

<details>
<summary>✅ Answer</summary>

The Deployment labels its Pods `app: web-frontend`, but the Service selector is `app: webfrontend` (missing the hyphen). They don't match, so the Service has **no endpoints** and forwards traffic to nothing. Fix: make the selector match the Pod label exactly (`app: web-frontend`). See [`solutions/fixed-3-service-no-endpoints.yaml`](solutions/fixed-3-service-no-endpoints.yaml).

*This is the single most common Kubernetes networking bug.* Whenever a Service "doesn't work" but the Pods are healthy, check `kubectl get endpoints <svc>` first.
</details>

---

## 🎓 What to take away
You diagnosed all three with the *same loop*: **get → describe → logs → endpoints**. In an interview, narrating that loop calmly ("first I'd check the Pod status, then the events, then the logs...") matters more than memorising any single fix.

## 🧹 Clean up
```bash
kubectl delete -f broken/ --ignore-not-found
kubectl delete -f solutions/ --ignore-not-found
```
