# Bootstrap

This directory contains the manifests required to bootstrap OpenShift GitOps (ArgoCD) onto a cluster. It is the **only part of this repository that is applied manually** — everything else is managed by ArgoCD after bootstrap is complete.

All manifests are structured for use with `kustomize`. Apply them using `oc apply -k <path>`.

---

## Directory Structure

```
bootstrap/
└── openshift-gitops/
    ├── base/
    │   ├── operator/       # OpenShift GitOps operator Subscription
    │   └── instance/       # ArgoCD instance customisation (placeholder)
    └── overlays/
        └── hub/            # Hub cluster overlay — apply this to bootstrap the hub cluster
            └── infra-applicationset.yaml
```

---

## Subdirectories

### `openshift-gitops/base/operator/`

Contains the `Subscription` CR that installs the **OpenShift GitOps operator** from the Red Hat operator catalog. The operator is installed into `openshift-operators`, which already has a global `OperatorGroup` — no additional group configuration is required.

Once applied, the operator automatically creates a default ArgoCD instance in the `openshift-gitops` namespace.

### `openshift-gitops/base/instance/`

Reserved for customisation of the default ArgoCD instance. Currently a placeholder — populate this directory when you need to override operator defaults such as resource limits, RBAC, or SSO integration.

### `openshift-gitops/overlays/hub/`

The hub cluster overlay. Extends the base and adds the `infra-applicationset.yaml` ApplicationSet that hands cluster configuration management over to ArgoCD.

The `infra-applicationset.yaml` ApplicationSet uses a **list generator** to enumerate infrastructure components. Each element maps to a kustomize overlay at `infra/<name>/overlays/hub/`. To onboard a new infrastructure component, add its name to the `spec.generators[].list.elements` list.

---

## Bootstrap Sequence

Follow these steps in order. Do not proceed to the next step until the previous one is healthy.

### Step 1 — Install the OpenShift GitOps Operator and apply the root ApplicationSet

```bash
oc apply -k bootstrap/openshift-gitops/overlays/hub/
```

Wait for the operator and default ArgoCD instance to become ready:

```bash
oc get pods -n openshift-gitops -w
```

All pods should reach `Running` status before proceeding.

### Step 2 — Verify ArgoCD is accessible

Retrieve the ArgoCD console route:

```bash
oc get route openshift-gitops-server -n openshift-gitops
```

Log in and confirm the instance is healthy. ArgoCD will immediately begin reconciling all infrastructure components listed in the ApplicationSet.

---

## Notes

- The operator `Subscription` is intentionally included in the bootstrap apply rather than managed by ArgoCD, to avoid ArgoCD blocking operator upgrades.
- To re-bootstrap a cluster from scratch, follow the steps above. The rest of the repository state is restored automatically by ArgoCD once the ApplicationSet syncs.
- This bootstrap directory is **hub cluster only**. Spoke clusters are onboarded separately and do not require manual bootstrap steps.
