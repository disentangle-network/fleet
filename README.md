# Disentangle Network Fleet

GitOps deployment template for running Disentangle Network nodes across one or more Kubernetes clusters.

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [flux](https://fluxcd.io/flux/installation/)
- [helm](https://helm.sh/docs/intro/install/)
- [age](https://github.com/FiloSottile/age) + [sops](https://github.com/getsops/sops)
- [launch](https://github.com/disentangle-network/launch) (`brew install disentangle-network/tap/launch`)

For OCI Always Free deployments, also install:
- [oci-cli](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)
- [OpenTofu](https://opentofu.org/docs/intro/install/)
- [oci-tf-bootstrap](https://github.com/LarsenClose/oci-tf-bootstrap) (`brew install disentangle-network/tap/oci-tf-bootstrap`)

## Quick Start

### 1. Use this template

Click **"Use this template"** on GitHub, or:

```bash
gh repo create my-org/fleet --template disentangle-network/fleet --private --clone
cd fleet
```

### 2. Configure credentials

```bash
launch setup
```

This walks you through OCI, Cloudflare, GitHub, and SOPS age key configuration.

### 3. Add your clusters

```bash
# OCI Always Free cluster (provisioned by launch)
launch cluster add oci-dev --infra cloud --arch arm64 --resources medium --nodes 3

# Existing clusters (Talos, k3s, managed K8s, etc.)
launch cluster add my-cluster --infra bare-metal --arch arm64 --resources small --nodes 3
```

### 4. Initialize secrets

```bash
launch secrets init --cluster oci-dev --provider age
launch secrets init --cluster my-cluster --provider age
```

### 5. Provision infrastructure (OCI only)

```bash
launch infra plan     # Review the plan
launch infra apply    # Provision VCN, OKE cluster, node pools
```

### 6. Bootstrap FluxCD

```bash
launch bootstrap --cluster oci-dev
launch bootstrap --cluster my-cluster
```

FluxCD will reconcile this repo and deploy disentangle-node pods to each cluster.

### 7. Verify

```bash
launch status
kubectl --context oci-dev get pods -n disentangle
```

## Repository Structure

```
clusters/           # Per-cluster FluxCD Kustomizations (created by launch cluster add)
apps/
  base/             # HelmRelease for disentangle-node
  disentangle/      # App overlay (inherits from base)
infrastructure/
  controllers/      # Ingress, cert-manager, external-dns
  configs/          # ClusterIssuers, NetworkPolicies
  secrets/          # SOPS-encrypted infrastructure secrets
secrets/            # Per-cluster SOPS-encrypted secrets
config.yaml.example # launch CLI config template
.sops.yaml          # SOPS encryption rules
```

## Customization

### Node count and resources

Edit `clusters/<name>/cluster-settings.yaml` or pass values during `launch cluster add`:

```bash
launch cluster add my-cluster --nodes 5 --resources large
```

### Adding infrastructure controllers

Uncomment and configure resources in `infrastructure/controllers/kustomization.yaml`.

## Links

- [Disentangle Protocol](https://github.com/disentangle-network/protocol)
- [Helm Chart](https://github.com/disentangle-network/deploy)
- [Launch CLI](https://github.com/disentangle-network/launch)
- [Paper 1: TMC](https://doi.org/10.5281/zenodo.18671600)
- [Paper 2: Excitability Topology](https://doi.org/10.5281/zenodo.18695184)
