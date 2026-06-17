# 🤝 Contributing

Thanks for thinking about helping out! This is a learning project, so contributions of every size are welcome — even fixing a typo.

If this repo helped you, the easiest way to support it is to **star** ⭐ it, **fork** 🍴 it, and **share** 🔗 it on LinkedIn so others can find it too.

## Ways you can help

- **Fix something** — a typo, a broken link, or a manifest that does not apply cleanly.
- **Add a new exercise** — another workload type (StatefulSet, Job, CronJob, Ingress), or a new "broken Pod" to debug.
- **Improve the wording** — make an explanation clearer or simpler. Explanations are the whole point of this lab.
- **Add a broken-manifest scenario** — realistic failures are the most valuable contributions here.

## How to contribute (step by step)

1. **Fork** this repo to your own GitHub account.
2. **Clone** your fork:
   ```bash
   git clone https://github.com/<your-username>/kubernetes-practice-lab.git
   cd kubernetes-practice-lab
   ```
3. **Create a branch**:
   ```bash
   git switch -c add-cronjob-exercise
   ```
4. **Make your change** and test it (see the checklist below).
5. **Commit** and **push**, then open a **Pull Request**:
   ```bash
   git add .
   git commit -m "Add a CronJob exercise"
   git push -u origin add-cronjob-exercise
   ```

## Adding an exercise

Keep the same simple style:

- Add an `exercise-*.md` brief that explains the **goal**, the **why**, and a clear **spec**.
- Add a working manifest in the part's `solutions/` folder.
- For debugging scenarios, add a broken manifest in `broken/` plus the fixed version in `solutions/`, and explain *what symptom* it produces.
- Use plain, simple English and prefer public images (`nginx`, `busybox`) so it works on any cluster with nothing to build.

## Checklist before you open a PR

- [ ] Manifests are valid: `kubectl apply --dry-run=client -f <file>` passes.
- [ ] They actually run on a local cluster (kind / minikube / Docker Desktop).
- [ ] A "broken" scenario reproduces the symptom you describe, and the fix resolves it.
- [ ] Explanations say **why**, not just **what**.
- [ ] If you added a new part, you linked it from the main `README.md`.

## Code of conduct

Be kind and helpful. Assume good intent and keep feedback friendly.

---

Made by **Shubham Sharma** · [GitHub](https://github.com/shubhs248) · [LinkedIn](https://www.linkedin.com/in/shubhs248)
