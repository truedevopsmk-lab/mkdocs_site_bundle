A clean, practical, reusable **ArgoCD Troubleshooting Cheat Sheet** that you can save, share, or drop into any repo/wiki.
Designed specifically for **real-world GitOps debugging**, including Crossplane, Helm, Kustomize, and SyncWave issues.

---

# 🚀 **ArgoCD Troubleshooting Cheat Sheet (Reusable, Production-Ready)**

## 🟢 **1. Check App Status**

See sync + health + revision + errors.

```
argocd app get <app-name>
```

You’re looking for:

* **Sync Status:** Synced
* **Health:** Healthy
* **OperationState:** Succeeded

If not → scroll to the events/errors.

---

## 🟣 **2. List All Managed Resources**

Identify which resource is failing or OutOfSync.

```
argocd app resources <app-name>
```

Filter by unhealthy:

```
argocd app resources <app-name> --health Unhealthy
```

---

## 🔵 **3. Check Differences (Drift)**

Compare live state vs Git:

```
argocd app diff <app-name>
```

Useful for detecting:

* local edits made via kubectl
* prune mismatch
* overwritten fields
* manual deletes

---

## 🟡 **4. Force Sync / Retry**

When ArgoCD refuses to sync due to drift or invalid state:

```
argocd app sync <app-name> --force
```

Adds extra useful flags:

```
argocd app sync <app-name> --force --prune --retry-limit 5
```

---

## 🔴 **5. Refresh Cache**

Similar to UI “HARD REFRESH”. Forces ArgoCD to re-fetch manifests & cluster state.

```
argocd app refresh <app-name>
```

Hard refresh:

```
argocd app refresh <app-name> --hard
```

---

## 🔥 **6. Wait Until the App is Healthy**

Automates waiting for nodes, CRDs, pods, or dependencies.

```
argocd app wait <app-name>
```

Wait for both sync + health:

```
argocd app wait <app-name> --sync --health
```

---

## ⚙️ **7. Check Detailed Sync Operation**

Shows the exact operation log (failed resource, diff, retry reason).

```
argocd app get <app-name> --show-operations
```

Same but more verbose:

```
argocd app get <app-name> --show-operation-events
```

---

## 📌 **8. View Sync History**

Great for tracking failures, retries, and version changes.

```
argocd app history <app-name>
```

Rollback to a previous revision:

```
argocd app rollback <app-name> <ID>
```

---

## 🧩 **9. Get Controller Logs (when UI hides real errors)**

Application controller:

```
kubectl logs -n argocd deploy/argocd-application-controller
```

Repo server (Git fetch/manifest generation issues):

```
kubectl logs -n argocd deploy/argocd-repo-server
```

Server (API/UI issues):

```
kubectl logs -n argocd deploy/argocd-server
```

---

## 🧨 **10. Common Real-World Failures & Fixes**

### ❌ *"one or more synchronization tasks are not valid"*

**Fix:**

* Add `SkipDryRunOnMissingResource=true`
* Add sync-waves
* Ensure CRDs exist before dependent resources

### ❌ OutOfSync but no diff

**Fix:** Hard refresh

```
argocd app refresh <app-name> --hard
```

### ❌ Crossplane resources fail during dry-run

**Fix:**
Add:

```
argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
```

### ❌ Kustomize "invalid prune"

**Fix:** Use structured fields in `kustomization.yaml`
or add:

```
- PruneLast=true
```

### ❌ Helm hooks not running

**Fix:**
Ensure:

```
helm.sh/hook-weight: "-1"
```

---

## 🧭 **11. Debug Sync-Wave Ordering (Advanced)**

Check resources sorted by sync-wave:

```
argocd app resources <app-name> --group-by syncwave
```

---

## 🧹 **12. Clean Up Failed / Stuck Resources**

Sometimes ArgoCD gets stuck due to finalizers or old CRDs.

Delete a stuck resource:

```
kubectl delete <kind> <name> --force --grace-period=0
```

Delete old provider CRDs (Crossplane):

```
kubectl delete provider.pkg.crossplane.io <provider-name>
```

---

## 🛡 **13. Healthy GitOps Best Practices**

Use these always:

**In Application.yaml**

```
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
    - PruneLast=true
    - ApplyOutOfSyncOnly=true
    - SkipDryRunOnMissingResource=true
```

**In manifests**

```
argocd.argoproj.io/sync-wave: "<number>"
```

---

## 🧲 **14. Crossplane-Specific Commands**

Show provider status:

```
kubectl get providers.pkg.crossplane.io
```

Show managed resources owned by Crossplane:

```
kubectl get managed
```

Describe bucket:

```
kubectl describe bucket <name>
```

---

# 🏁 **Summary: Your Go-To ArgoCD Debug Commands**

```
argocd app get <app>
argocd app resources <app>
argocd app diff <app>
argocd app sync <app> --force
argocd app refresh <app> --hard
argocd app wait <app> --sync --health
argocd app history <app>
```

---
