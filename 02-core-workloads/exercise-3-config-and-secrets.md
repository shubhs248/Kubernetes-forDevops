# Exercise 2.3 — ConfigMaps & Secrets

**You write:** a ConfigMap that becomes the web page nginx serves (config as a *file*), and a Secret that shows up as *environment variables* inside the container.

---

## 🧠 Why this matters
Config doesn't belong inside your image — if it did, you'd rebuild the image for every environment change. Kubernetes keeps config separate:

- **ConfigMap** — non-secret settings. Inject it as **env vars** or **mount it as files**.
- **Secret** — same idea, but for sensitive values (passwords, tokens, API keys). Stored base64-encoded and can be encrypted at rest / access-controlled.

You'll do both injection styles here so the difference is concrete: the ConfigMap becomes a file you can *see* in the browser, and the Secret becomes env vars you can *print* from inside the Pod.

---

## 📋 The spec

**1) A ConfigMap** `hello-config` with one key:
- key `index.html`, value: any HTML, e.g. `<h1>Hello from a ConfigMap!</h1>`

**2) A Secret** `hello-secret` with two keys (use `stringData` so you write plain text, not base64):
- `APP_ENV: production`
- `API_KEY: super-secret-value`

**3) A Deployment** `hello-config-app` (image `nginx:1.27-alpine`) that:
- **mounts** the ConfigMap at `/usr/share/nginx/html` (so `index.html` replaces the default page), and
- pulls **both** Secret keys in as environment variables (`APP_ENV`, `API_KEY`).

## ✅ How to check
```bash
kubectl apply -f my-config.yaml      # or apply each file you wrote
kubectl get pods

# 1) See the ConfigMap-as-a-file in action
kubectl port-forward deploy/hello-config-app 8080:80
#   open http://localhost:8080  ->  your "Hello from a ConfigMap!" page

# 2) See the Secret-as-env-vars inside the container
kubectl exec deploy/hello-config-app -- printenv APP_ENV API_KEY
#   prints: production / super-secret-value
```
Compare with the files in [`solutions/`](solutions): `configmap.yaml`, `secret.yaml`, `deployment-config.yaml`.

## 💡 Hints
- **Mounting a ConfigMap as files:** define a `volumes:` entry of type `configMap` referencing `hello-config`, then a `volumeMounts:` on the container pointing at `/usr/share/nginx/html`. Each ConfigMap *key* becomes a *file* of that name.
- **Secret as env vars (one at a time):** under the container, use `env:` with `valueFrom: secretKeyRef: { name: hello-secret, key: API_KEY }`. To pull *all* keys at once, use `envFrom: [{ secretRef: { name: hello-secret } }]`.
- `stringData` lets you write secrets in plain text; Kubernetes base64-encodes them for you. (If you use `data` instead, you must base64-encode the values yourself.)
- A Secret is **not** encryption by itself — base64 is just encoding. The value of Secrets is access control + encryption-at-rest, not obscurity. Never commit real secrets to Git.

## 🧹 Clean up
```bash
kubectl delete -f my-config.yaml
```
