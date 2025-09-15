Perfect question 👍 — **`CrashLoopBackOff`** is another very common Kubernetes Pod error.

It means:
👉 The container **starts → crashes → restarts → crashes again** → kubelet keeps retrying with **backoff** (waiting longer between retries).

---

## 🔎 **Common Causes**

1. **Application bug**

   * App exits immediately (non-zero exit code).
   * Bad code/config inside the container.

2. **Configuration errors**

   * Wrong ENV vars, missing config files, invalid flags.
   * Wrong command/entrypoint in Dockerfile or manifest.

3. **Dependency issues**

   * App requires DB/API but can’t connect → crashes.
   * Network policies blocking access.

4. **Port / Binding issues**

   * App fails to bind to required port.

5. **Resource limits**

   * OOMKilled (out of memory).
   * CPU throttling.

6. **Probes misconfigured**

   * Liveness probe failing → kubelet restarts the Pod.

---

## 🛠 **Troubleshooting Steps**

### 1. Check Pod events

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for **Last State** under `Containers:`:

* `Terminated: ExitCode: 1`
* `OOMKilled`
* `Error`

---

### 2. View container logs

```bash
kubectl logs <pod-name> -n <namespace>
# If multiple containers:
kubectl logs <pod-name> -c <container-name>
```

➡ Logs usually tell you *why* the container is crashing.

---

### 3. Check if Pod restarts immediately

To see recent crash logs (before restart):

```bash
kubectl logs <pod-name> --previous
```

---

### 4. Check probes

If livenessProbe fails → Pod restarts continuously.
Look at Pod spec:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

👉 If misconfigured, fix path/port or increase `initialDelaySeconds`.

---

### 5. Check resources

* Describe Pod → look for `OOMKilled`.
* If yes, increase memory/CPU limits in Deployment:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

---

### 6. Debug inside Pod

If app exits too fast, run in debug mode:

```bash
kubectl run tmp-shell --rm -it --image=busybox -- bash
```

Or override entrypoint:

```bash
kubectl run debug --image=<your-image> -it --restart=Never --command -- sh
```

---

✅ **Summary:**

* `CrashLoopBackOff` = container keeps crashing after start.
* **Check logs** (`kubectl logs` / `--previous`).
* **Check events** (`kubectl describe pod`).
* Common fixes: correct config/env vars, fix probes, increase resources, wait for dependencies.

---

Do you want me to make you a **CrashLoopBackOff troubleshooting flowchart** (step-by-step decision tree) so you can use it quickly in interviews or production?
