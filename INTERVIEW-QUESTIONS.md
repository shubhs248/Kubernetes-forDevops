# 🎤 Kubernetes — Interview Questions

> Kubernetes is *the* most-tested DevOps/SRE topic, and interviewers go deep. Plain-English answers you can say out loud, plus the "a Pod is broken, what do you do?" scenarios that really decide the interview. Do the lab (especially the debugging part) first.

**How to use this file:** cover the answers, read a question, answer out loud, then check. For debugging questions, narrate your *process* — interviewers care how you think, not just the fix.

---

## 🟢 Warm-up & core concepts

**1. What problem does Kubernetes solve?**
It runs and manages containers across many machines, automatically: scheduling them onto nodes, restarting failed ones, scaling up/down, rolling out new versions, load-balancing traffic, and keeping the actual state matching the desired state. Docker runs a container on one host; Kubernetes runs many reliably across a cluster.

**2. Explain the desired-state / reconciliation model.**
You declare what you want (e.g. "3 replicas of this app") and a control loop continuously compares desired vs actual state and acts to close the gap. That's why Kubernetes self-heals — kill a Pod and the loop recreates it because actual (2) no longer matches desired (3).

**3. What is a Pod?**
The smallest deployable unit — one or more containers that share a network namespace (same IP, talk over `localhost`) and storage. You usually don't create Pods directly; controllers like Deployments create them for you. Pods are ephemeral and get a new IP each time.

**4. Pod vs Deployment vs ReplicaSet — how do they relate?**
A **Deployment** manages a **ReplicaSet**, which keeps a set number of **Pods** running. You write the Deployment; it creates a ReplicaSet per version and the ReplicaSet ensures the Pod count. Chain: **Deployment → ReplicaSet → Pods**.

**5. Why use a Deployment instead of creating Pods directly?**
A bare Pod isn't recreated if it dies. A Deployment keeps the desired number of Pods alive, replaces failed ones, and gives you rolling updates and rollbacks. Self-healing and versioned deploys come from the controller, not the Pod.

**6. What is a Service and why is it needed?**
Pods are disposable and change IPs, so you can't talk to them directly. A Service is a stable name + virtual IP that load-balances to all Pods matching its **label selector**. It's the permanent front door to an ever-changing set of Pods.

**7. How does a Service know which Pods to send traffic to?**
Through **labels and a selector**. The Service's `selector` is matched against Pod labels; matching Pods become its **endpoints**. If the selector doesn't match any Pod labels, the Service has no endpoints and traffic goes nowhere — the most common networking bug.

---

## 🔵 Networking & exposure

**8. What are the Service types?**
- **ClusterIP** (default) — reachable only inside the cluster.
- **NodePort** — opens a static port on every node (30000–32767).
- **LoadBalancer** — provisions an external load balancer (cloud).
- **ExternalName** — maps to an external DNS name.

**9. What is an Ingress and how is it different from a Service?**
A Service exposes one app. An **Ingress** is an HTTP(S) router/reverse-proxy in front of many Services — it does host/path-based routing, TLS termination, and a single entry point. It needs an Ingress controller (nginx, Traefik) to actually do the work.

**10. How do Pods find each other / do service discovery?**
Via the Service name as DNS. A Pod can reach another app at `http://<service-name>` (or `<service>.<namespace>.svc.cluster.local`). Kubernetes runs an internal DNS (CoreDNS) that resolves Service names to their ClusterIP.

**11. `port` vs `targetPort` vs `nodePort`?**
- `port` — the port the Service listens on.
- `targetPort` — the container port traffic is forwarded to.
- `nodePort` — (NodePort type only) the port opened on each node.

---

## 🟣 Config, state & scheduling

**12. ConfigMap vs Secret?**
Both externalise config from the image and can be injected as env vars or mounted files. A **ConfigMap** is for plain config; a **Secret** is for sensitive data — base64-encoded, can be encrypted at rest and access-controlled. Base64 is *not* encryption, so don't treat Secrets as inherently safe in Git.

**13. How do you give an app its configuration?**
ConfigMaps/Secrets injected as environment variables (`env`/`envFrom`) or mounted as files (`volumeMounts`). This keeps the image generic and lets the same image run in dev/staging/prod with different config.

**14. requests vs limits for resources?**
- **requests** — what the Pod is *guaranteed*; the scheduler uses it to place the Pod.
- **limits** — the *ceiling*; exceeding the CPU limit throttles, exceeding the memory limit gets the container **OOMKilled**.
Setting both is good practice for stability and fair sharing.

**15. Deployment vs StatefulSet vs DaemonSet?**
- **Deployment** — stateless apps; interchangeable Pods.
- **StatefulSet** — stateful apps needing stable network IDs and persistent storage per Pod (databases).
- **DaemonSet** — one Pod on every node (log/metrics agents).

**16. How does storage persist beyond a Pod's life?**
With **PersistentVolumes (PV)** and **PersistentVolumeClaims (PVC)**. A PVC requests storage; it's bound to a PV (often dynamically provisioned via a StorageClass). The Pod mounts the PVC, so data survives Pod restarts/rescheduling.

**17. How does the scheduler decide where a Pod goes?**
It filters nodes that can fit the Pod (resource requests, node selectors/affinity, taints/tolerations) and scores the rest, picking the best. `Pending` means no node passed the filters — usually not enough CPU/memory or a taint with no matching toleration.

---

## 🟠 Operating: rollouts, scaling, health

**18. How do you do a zero-downtime deployment?**
A **rolling update**: the Deployment creates a new ReplicaSet and shifts Pods gradually (governed by `maxUnavailable`/`maxSurge`), keeping the app available throughout. `kubectl set image` or editing the manifest triggers it.

**19. A deploy went bad. How do you roll back?**
`kubectl rollout undo deployment/<name>` (optionally `--to-revision=N`). It works because the old ReplicaSet is kept (scaled to 0), so rollback just scales it back up. `kubectl rollout history` shows revisions.

**20. Explain liveness vs readiness vs startup probes.**
- **readiness** — "ready for traffic?" Failing removes the Pod from the Service (no restart).
- **liveness** — "still healthy?" Failing **restarts** the container.
- **startup** — "finished booting?" Holds off the other probes for slow starters.
Key mistake: using liveness where you needed readiness — a temporarily-busy app gets restart-looped instead of just pausing traffic.

**21. How do you scale an app? What about automatically?**
Manually: `kubectl scale deployment/<name> --replicas=N` or edit the manifest. Automatically: a **HorizontalPodAutoscaler (HPA)** adds/removes Pods based on CPU/memory or custom metrics. (Cluster Autoscaler adds/removes *nodes*.)

---

## 🔴 Debugging (the questions that decide it)

**22. A Pod is in `CrashLoopBackOff`. Walk me through it.**
It means the container starts then exits/errors repeatedly. Steps: `kubectl describe pod` (check last state + exit code + events), `kubectl logs <pod> --previous` (what it said before dying), then fix the root cause — usually a missing config/env/dependency, a bad command, or the main process not staying in the foreground. The "BackOff" is Kubernetes waiting longer between restart attempts.

**23. A Pod is stuck in `ImagePullBackOff`. What's wrong?**
Kubernetes can't pull the image. Causes: wrong image name/tag, image doesn't exist, wrong registry, or missing `imagePullSecrets` for a private registry. `kubectl describe pod` Events spell out the pull error. Fix the reference or add the pull secret.

**24. A Pod is stuck in `Pending`. Why?**
It can't be scheduled. Usually insufficient CPU/memory on any node, a PVC that can't be bound, or taints/node-affinity that no node satisfies. `kubectl describe pod` Events ("0/3 nodes available...") tell you exactly which constraint failed.

**25. The Pods are healthy but the Service returns nothing. How do you debug?**
Check `kubectl get endpoints <svc>` — if it's empty, the Service selector doesn't match the Pod labels (the #1 cause). Also verify `targetPort` matches the container port and that the Pods are actually `Ready` (a failing readiness probe removes them from the Service).

**26. What's your general debugging loop?**
`kubectl get pods` (status) → `kubectl describe pod` (events) → `kubectl logs` (`--previous` if crashed) → for networking, `kubectl get endpoints`. Calmly narrating this loop is often what interviewers are actually testing.

**27. What does `kubectl describe` give you that `kubectl get` doesn't?**
`get` is a quick list/status. `describe` shows full detail plus the **Events** timeline at the bottom — pulled image, scheduling decisions, probe failures, restarts. Events are usually where the "why" lives.

---

## 🟤 Architecture & "senior" questions

**28. What are the main components of a Kubernetes cluster?**
- **Control plane:** API server (front door), etcd (the cluster's database/state store), scheduler (places Pods), controller manager (runs the control loops).
- **Each node:** kubelet (runs Pods, reports health), kube-proxy (Service networking), and a container runtime (containerd).

**29. What is etcd and why does it matter?**
The key-value store holding the entire cluster state (the "source of truth"). If etcd is lost without a backup, you lose the cluster's desired state. Backing up etcd is a critical ops responsibility.

**30. What happens when you run `kubectl apply -f deploy.yaml`?**
`kubectl` sends the manifest to the **API server**, which validates and stores it in **etcd**. The **Deployment controller** notices and creates a ReplicaSet; the **ReplicaSet controller** creates Pods; the **scheduler** assigns each Pod to a node; that node's **kubelet** tells the runtime to pull the image and start the container.

**31. How do you handle secrets properly (beyond plain Secrets)?**
Enable encryption-at-rest for etcd, restrict access with RBAC, avoid committing Secrets to Git (use Sealed Secrets / SOPS / external managers like Vault or cloud secret stores), and rotate regularly. Treat a base64 Secret as "not in the image", not "secure".

**32. What is RBAC?**
Role-Based Access Control — who can do what. **Roles/ClusterRoles** define permissions; **RoleBindings/ClusterRoleBindings** grant them to users or ServiceAccounts. Follow least-privilege.

**33. Namespaces — what are they for?**
Logical isolation/grouping within one cluster — separating teams, environments, or apps. They scope names and let you apply resource quotas and RBAC per group. They are *not* a hard security boundary by themselves.

**34. How do you keep an app available during node failure?**
Run multiple replicas spread across nodes (anti-affinity / topology spread), set resource requests so the scheduler can reschedule, use a PodDisruptionBudget to limit voluntary disruptions, and use readiness probes so traffic only hits healthy Pods.

---

## 🧠 Hands-on prompts to rehearse

All of these are in this lab.

1. **Write a Deployment + Service** from scratch (correct label selectors).
2. **Inject a ConfigMap as a file and a Secret as env vars.**
3. **Do a rolling update, then roll it back.**
4. **Add readiness + liveness probes** and explain the difference.
5. **Diagnose and fix** an ImagePullBackOff, a CrashLoopBackOff, and a Service with no endpoints.

---

## ⭐ Found this useful?
If this helped your prep, please **star** ⭐, **fork** 🍴, and **share** 🔗 the repo on LinkedIn. Got a Kubernetes question from a real interview? Add it via [CONTRIBUTING.md](CONTRIBUTING.md).

Made by **Shubham Sharma** · [GitHub](https://github.com/shubhs248) · [LinkedIn](https://www.linkedin.com/in/shubhs248)
