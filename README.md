# ArgoCD Configuration

This repository contains ArgoCD ApplicationSet definitions for managing a Kubernetes homelab environment using GitOps principles.

## Overview

This repository uses ArgoCD ApplicationSets to deploy and manage multiple applications across different namespaces in a Kubernetes cluster. All applications are configured to use the `argocd-vault-plugin-kustomize` plugin for secure secret management with HashiCorp Vault.

## Repository Structure

```
apps/
├── devops-tools.yaml   # ArgoCD and Renovate
├── media.yaml          # Media server applications
├── monitoring.yaml     # Observability stack
├── network.yaml        # Network infrastructure
├── p2p.yaml            # P2P applications
├── sec.yaml            # Security tools
├── storage.yaml        # Storage provisioners
└── vault.yaml          # HashiCorp Vault
```

## Features

All ApplicationSets are configured with:
- **Automated sync** - Changes are automatically deployed
- **Self-healing** - Cluster state is automatically corrected if it drifts
- **Pruning** - Removed resources are automatically cleaned up
- **Namespace creation** - Namespaces are created automatically if they don't exist
- **Vault integration** - Secrets are managed via argocd-vault-plugin-kustomize

## Prerequisites

- Kubernetes cluster
- ArgoCD installed in the `devops-tools` namespace
- ArgoCD Vault Plugin configured
- HashiCorp Vault configured and accessible

## Usage

To deploy all applications, apply the ApplicationSet definitions to your cluster:

```bash
kubectl apply -f apps/
```

To deploy a specific category:

```bash
kubectl apply -f apps/monitoring.yaml
```

## Monitoring

All applications can be monitored through the ArgoCD UI or CLI:

```bash
# View all applications
argocd app list

# Check application status
argocd app get <app-name>

# Sync an application manually
argocd app sync <app-name>
```

## Adding New Applications

To add a new application to an existing category:

1. Edit the appropriate ApplicationSet file (e.g., [`apps/monitoring.yaml`](apps/monitoring.yaml))
2. Add a new element to the `generators.list.elements` array
3. Ensure the corresponding directory exists in the source repository
4. Commit and push the changes

Example:

```yaml
- list:
    elements:
      - name: existing-app
      - name: new-app  # Add this line
```
