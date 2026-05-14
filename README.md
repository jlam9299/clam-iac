# clam-iac

Infrastructure-as-code for OpenShift clusters in the `clamie.au` environment, managed via GitOps using OpenShift GitOps (ArgoCD).

## How it works

1. **Bootstrap** — the operator and root ApplicationSet are applied manually once to a new cluster.
2. **ArgoCD takes over** — the ApplicationSet discovers components under `infra/` and `apps/` and reconciles them from Git automatically.
3. **Changes are made via Git** — open a PR, merge, ArgoCD syncs.

All manifests use [Kustomize](https://kustomize.io/) with a `base/` + `overlays/<cluster>/` layout so components can be customised per cluster without duplicating the base.

---

## Quick Start

### Prerequisites

- `oc` CLI authenticated to the target cluster as `cluster-admin`
- `kubectl` / `kustomize` (or use the `oc apply -k` built-in)

### Bootstrap the hub cluster (`ocp`)

```bash
oc apply -k bootstrap/openshift-gitops/overlays/hub/
```

This installs the OpenShift GitOps operator and creates the root ApplicationSet. Wait for all pods to be ready:

```bash
oc get pods -n openshift-gitops -w
```

ArgoCD will then begin reconciling all components listed in the ApplicationSet automatically. See [bootstrap/README.md](bootstrap/README.md) for the full bootstrap sequence.

---

## Directory Structure

```
clam-iac/
├── bootstrap/                        # Applied manually once per cluster
├── infra/                            # Cluster infrastructure — managed by ArgoCD
└── apps/                             # (future) Application workloads
```

### `bootstrap/`

One-time manual apply per cluster. Contains the OpenShift GitOps operator `Subscription` and the root `ApplicationSet` that hands control to ArgoCD. Nothing in this directory is reconciled by ArgoCD itself.

→ See [bootstrap/README.md](bootstrap/README.md)

### `infra/`

Cluster infrastructure configuration reconciled by ArgoCD. Each subdirectory is a discrete component following the `base/` + `overlays/<cluster>/` pattern.

| Component           | Description                                                                            |
| ------------------- | -------------------------------------------------------------------------------------- |
| `oauth`             | Configures LDAP authentication via FreeIPA (`idm.clamie.au`)                           |
| `openshift-console` | Sets the console hostname and enables console plugins (GitOps, monitoring, networking) |

### `apps/`

Application workloads will live here, following the same `base/` + `overlays/<cluster>/` structure as `infra/`.

---

## Adding a New Infrastructure Component

1. Create the directory structure:
   ```
   infra/<component>/
   ├── base/
   │   ├── kustomization.yaml
   │   └── <manifest>.yaml
   └── overlays/
       └── hub/
           └── kustomization.yaml
   ```
2. Add the component name to `spec.generators[0].list.elements` in `bootstrap/openshift-gitops/overlays/hub/infra-applicationset.yaml`.
3. Open a PR — ArgoCD will pick it up automatically after merge.

---

## Clusters

| Cluster | Role | Overlay         |
| ------- | ---- | --------------- |
| `ocp`   | Hub  | `overlays/hub/` |
