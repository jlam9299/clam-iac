# Bootstrap

This directory contains the manifests required to bootstrap OpenShift GitOps (ArgoCD) onto a cluster. It is the **only part of this repository that is applied manually** — everything else is managed by ArgoCD after bootstrap is complete.

All manifests are structured for use with `kustomize`. Apply them using `oc apply -k <path>`.

---

## Directory Structure

```
bootstrap/
├── operator/           # OpenShift GitOps operator subscription
├── instance/           # ArgoCD instance customisation (placeholder)
└── applicationsets/    # Root ApplicationSets that hand control to ArgoCD
```

---

## Subdirectories

### `openshift-gitops-operator/`

Contains the `Subscription` CR that installs the **OpenShift GitOps operator** from the Red Hat operator catalog. The operator is installed into the `openshift-operators` namespace, which already has a global `OperatorGroup` — no additional group configuration is required.

Once applied, the operator will automatically create a default ArgoCD instance in the `openshift-gitops` namespace.

> This subscription is managed **manually** and is not reconciled by ArgoCD.

---

### `instance/`

Reserved for customisation of the ArgoCD instance created by the operator. The operator provisions a default instance in `openshift-gitops` which is sufficient for most cases.

Populate this directory when you have a specific reason to override operator defaults — for example, to configure resource limits, RBAC, or SSO integration.

> Currently a placeholder. No manifests need to be applied here during initial bootstrap.

---

### `applicationsets/`

Contains the root `ApplicationSet` CRs that ArgoCD uses to dynamically generate and manage `Application` CRs across the cluster. These are applied once manually to hand control of the cluster configuration and workloads over to ArgoCD.

| ApplicationSet        | Manages                                             |
| --------------------- | --------------------------------------------------- |
| `infrastructure.yaml` | Kustomize overlays under `infra/<resource>/overlays/` |
| `apps.yaml`           | Kustomize overlays under `apps/<resource>/overlays/`           |

Once applied, ArgoCD will reconcile all managed resources from Git automatically.

---

## Bootstrap Sequence

Follow these steps in order. Do not proceed to the next step until the previous one is healthy.

### Step 1 — Install the OpenShift GitOps Operator

```bash
oc apply -k bootstrap/openshift-gitops-operator/
```

Wait for the operator and default ArgoCD instance to become ready:

```bash
oc get pods -n openshift-gitops -w
```

All pods should reach `Running` status before proceeding.

---

### Step 2 — Verify ArgoCD is accessible

Retrieve the ArgoCD console route:

```bash
oc get route openshift-gitops-server -n openshift-gitops
```

Log in and confirm the instance is healthy before proceeding.

---

### Step 3 — Apply the root ApplicationSets

```bash
oc apply -k bootstrap/applicationsets/
```

ArgoCD will immediately begin reconciling all applications and cluster configuration defined in the repository. Monitor the ArgoCD console or CLI to confirm applications sync successfully.

---

## Notes

- The operator subscription is intentionally kept outside of ArgoCD management. Operator lifecycle is managed manually to avoid ArgoCD inadvertently blocking operator upgrades.
- If you need to re-bootstrap a cluster from scratch, follow the steps above in order. The rest of the repository state will be restored automatically by ArgoCD once Step 3 is complete.
- This bootstrap directory is **hub cluster only**. Spoke clusters (e.g. HCP clusters) are onboarded via ACM and do not require manual bootstrap steps.