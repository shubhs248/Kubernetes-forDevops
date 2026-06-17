# 🖼️ Diagram Gallery — Kubernetes

Rendered diagrams for this lab in **light + dark**. They adapt to your GitHub theme below; grab the files directly for slides or LinkedIn.

- Light: `NN-name.png` / `.svg` · Dark: `NN-name-dark.png` / `.svg`
- Editable Mermaid source lives in [`src/`](src). Re-render from the repo root with `render-diagrams.ps1`.

## 🎨 Colour legend
| Colour | Means |
|--------|-------|
| 🔵 Cyan | client / desired state |
| 🟢 Teal / Green | controllers, pods, kubelets |
| 🟠 Amber | etcd / container |
| 🔴 Rose | restart / failure |

---

### Deployment → ReplicaSet → Pods
A Deployment manages a ReplicaSet, which keeps the desired number of Pods running.

<picture><source media="(prefers-color-scheme: dark)" srcset="01-deployment-hierarchy-dark.png"><img alt="Deployment hierarchy" src="01-deployment-hierarchy.png"></picture>

### Service routing via labels
<picture><source media="(prefers-color-scheme: dark)" srcset="02-service-routing-dark.png"><img alt="Service routing" src="02-service-routing.png"></picture>

### Probe lifecycle (startup / readiness / liveness)
<picture><source media="(prefers-color-scheme: dark)" srcset="03-probes-dark.png"><img alt="Probe lifecycle" src="03-probes.png"></picture>

### Cluster architecture
<picture><source media="(prefers-color-scheme: dark)" srcset="04-cluster-architecture-dark.png"><img alt="Kubernetes cluster architecture" src="04-cluster-architecture.png"></picture>

---

Made by **Shubham Sharma** · [GitHub](https://github.com/shubhs248) · [LinkedIn](https://www.linkedin.com/in/shubhs248)
