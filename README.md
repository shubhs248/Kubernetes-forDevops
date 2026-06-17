# ☸️ Kubernetes for DevOps — Practice Lab

> A **clone-and-go** lab to actually *understand* Kubernetes — not just copy YAML. You'll deploy a real app, expose it, configure it, scale it, roll it out, and (most importantly) **fix it when it breaks**. Every step explains the *why*, because that's what interviews and on-call actually test.

[![Made with Kubernetes](https://img.shields.io/badge/Made%20with-Kubernetes-326CE5.svg?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 🎯 Why this repo?

Most people learn Kubernetes by pasting YAML until the app comes up — and then have no idea what to do when a Pod is stuck on `ImagePullBackOff` or a Service returns nothing. This lab is built the other way around:

- **Every concept is explained in plain English first**, with the mental model behind it.
- **You write the manifests yourself** from a spec, then compare with a working solution.
- **A whole section is about things going wrong** — the broken Pods you'll actually see in real clusters and interviews.

No prior Kubernetes needed. If you've done the [Docker lab](https://github.com/shubhs248/docker-practice-lab), you're ready: Docker *builds* the box, Kubernetes *runs many boxes reliably*.

## 🧠 The one mental model to keep

Kubernetes is a **desired-state** system. You don't tell it "start this container". You hand it a YAML file that says *"I want 3 copies of this app running"* and a background **control loop** constantly compares **what you want** vs **what's actually running**, and fixes the difference. A Pod died? It makes a new one. You asked for 5 instead of 3? It adds 2.

Almost everything else in this lab is a variation of that single idea. Keep it in your head.

## 🗂️ What's inside

```
kubernetes-practice-lab/
├── README.md                      ← you are here
├── CHEATSHEET.md                  ← 1-page kubectl + manifest reminder
├── CONTRIBUTING.md                ← how to add your own exercises
├── INTERVIEW-QUESTIONS.md         ← the questions interviewers actually ask
├── 01-kubernetes-basics/          ← warm-up: the objects + kubectl muscle memory
│   └── README.md
├── 02-core-workloads/             ← write manifests: Pod, Deployment, Service, Config, Secret
│   ├── README.md
│   ├── exercise-1-pod-and-deployment.md
│   ├── exercise-2-service-and-expose.md
│   ├── exercise-3-config-and-secrets.md
│   └── solutions/
└── 03-operating-and-debugging/    ← scale, roll out, add health probes, FIX broken pods
    ├── README.md
    ├── exercise-1-scaling-and-rollouts.md
    ├── exercise-2-health-probes.md
    ├── exercise-3-debug-broken-pods.md
    ├── broken/                     ← intentionally broken manifests to diagnose & fix
    └── solutions/
```

## ✅ Requirements: a small local cluster

You need a Kubernetes cluster and `kubectl`. Any of these works — pick one (all are free and run on your laptop):

| Option | Best for | Start command |
|--------|----------|---------------|
| **Docker Desktop** | Already have Docker? Easiest. | Settings → Kubernetes → *Enable* |
| **kind** | Lightweight, scriptable | `kind create cluster` |
| **minikube** | Batteries-included, has a dashboard | `minikube start` |
| **k3d** | Tiny, fast (k3s in Docker) | `k3d cluster create` |

Then check it's alive:

```bash
kubectl version --short      # client + server
kubectl get nodes            # should list at least one Ready node
```

> 💡 **No cluster?** You can still read every explanation and *write* all the manifests — that alone is great interview prep. The YAML is the part you get tested on.

## 🚀 Quick start

```bash
# 1. Get the code
git clone https://github.com/shubhs248/kubernetes-practice-lab.git
cd kubernetes-practice-lab

# 2. Make sure your cluster is up
kubectl get nodes

# 3. Try a ready-made solution end to end
kubectl apply -f 02-core-workloads/solutions/deployment.yaml
kubectl apply -f 02-core-workloads/solutions/service.yaml
kubectl get pods,svc

# 4. Reach the app (NodePort or port-forward)
kubectl port-forward svc/hello 8080:80
# open http://localhost:8080

# 5. Clean up when done
kubectl delete -f 02-core-workloads/solutions/
```

> Everything uses the public `nginx` image, so there's **nothing to build or push** — it just works on any cluster.

## 🧭 Suggested path (about 2 hours)

| # | Part | What you practise | Time |
|---|------|-------------------|------|
| 1 | [Kubernetes Basics](01-kubernetes-basics/README.md) | the objects, `kubectl get/describe/logs/apply`, namespaces | 30 min |
| 2 | [Core Workloads](02-core-workloads/README.md) | Pod, Deployment, Service, ConfigMap, Secret | 45 min |
| 3 | [Operating & Debugging](03-operating-and-debugging/README.md) | scaling, rolling updates & rollback, probes, **fixing broken Pods** | 45 min |

## 📝 How each part works

- **Part 1** is a guided warm-up: read the explanation, run the `kubectl` commands, and build muscle memory for inspecting a cluster.
- **Part 2**: read the spec, **write the YAML yourself**, `kubectl apply` it, verify, then compare with `solutions/`.
- **Part 3**: scale and update a running app, add health checks, then diagnose three Pods that are deliberately broken (the part that makes you dangerous in interviews and on-call).

---

## 🎤 Prepping for an interview?

Open **[INTERVIEW-QUESTIONS.md](INTERVIEW-QUESTIONS.md)** — the Kubernetes questions interviewers actually ask (Pods vs Deployments, Services, probes, "a Pod is in CrashLoopBackOff, what do you do?"), in plain English with short answers you can say out loud.

---

## ⭐ Found this useful?

- **Star** ⭐ the repo so more people discover it.
- **Fork** 🍴 it and practise on your own copy.
- **Share** 🔗 it on LinkedIn and tag me — I would love to see your progress.
- **Contribute** 🤝 a new exercise or fix. See [CONTRIBUTING.md](CONTRIBUTING.md).

## 👋 About the author

Made with care by **Shubham Sharma**.

- GitHub: [github.com/shubhs248](https://github.com/shubhs248)
- LinkedIn: [linkedin.com/in/shubhs248](https://www.linkedin.com/in/shubhs248)

## 📜 License

MIT — free to use, fork, teach with, and share. A star ⭐ or a tag on LinkedIn is always appreciated!
