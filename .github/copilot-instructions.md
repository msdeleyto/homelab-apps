# Copilot Instructions

## Repository Overview

This repo contains ArgoCD `Application` and `ApplicationSet` definitions that drive a Kubernetes homelab via GitOps. All manifests in this repo point to `https://github.com/msdeleyto/homelab-manifests.git` (`main` branch) as the source of truth for the actual Kubernetes workloads.

## Architecture

```
prod/   # Production environment ApplicationSets/Applications
test/   # Test environment ApplicationSets/Applications
```

All resources are deployed into the `devops-tools` namespace in ArgoCD (`metadata.namespace: devops-tools`). Each file manages one logical group of apps (e.g., `network.yaml`, `monitoring.yaml`).

### Two resource kinds are used

- **`ApplicationSet`** — used for groups of apps sharing the same namespace and source path pattern. Uses a `list` generator with `elements` entries (each with a `name`). The template interpolates `{{name}}` into the `path` and `metadata.name`.
- **`Application`** — used for singleton or sequenced installs (e.g., `vault`, `external-secrets`) where sync-wave ordering matters.

### Source path convention

`ApplicationSet` paths follow `'./<group>/{{name}}'`, e.g., `./network/cilium`. Single `Application` sources use explicit paths like `./vault/vault`.

## Key Conventions

### Sync policy
All resources use `automated` sync with `prune: true` and `selfHeal: true`, plus `CreateNamespace=true` and `ServerSideApply=true` sync options.

### Namespace pod security labels
Every `managedNamespaceMetadata.labels` block includes at minimum:
```yaml
pod-security.kubernetes.io/audit: restricted
pod-security.kubernetes.io/audit-version: latest
```
Namespaces that can tolerate it also add `enforce: baseline`. Do **not** add `enforce: restricted` unless explicitly intended.

### Sync waves (Application only)
Bootstrap-order dependencies use `argocd.argoproj.io/sync-wave` annotations (negative values run first):
- `vault`: `-2`
- `external-secrets`: `-1`

### Prod vs test

`test/` contains a subset of prod's ApplicationSets — `media`, `monitoring`, `p2p`, and `sec` exist only in `prod/`. All files that exist in both environments should be kept identical.

### Adding a new app to an existing ApplicationSet
Add an entry to `generators.list.elements`:
```yaml
elements:
  - name: existing-app
  - name: new-app   # add here
```
The name must match a directory path in `homelab-manifests` under the relevant group folder.

### Adding a new ApplicationSet
Follow the existing pattern: use the group name as both the `metadata.name` and the destination namespace. Match the pod security label set used by similar namespaces in the same environment.

### Disabling an app temporarily
Comment out the element entry rather than deleting it (see `monitoring.yaml` where `crowdsec` is commented out).
