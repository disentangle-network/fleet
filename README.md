# Disentangle Network Fleet

[![CI](https://github.com/disentangle-network/fleet/actions/workflows/ci.yml/badge.svg)](https://github.com/disentangle-network/fleet/actions/workflows/ci.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

GitOps deployment template for running Disentangle Network nodes across one or more Kubernetes clusters.

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [flux](https://fluxcd.io/flux/installation/)
- [helm](https://helm.sh/docs/intro/install/)
- [age](https://github.com/FiloSottile/age) + [sops](https://github.com/getsops/sops)
- [launch](https://github.com/disentangle-network/launch) (`brew install disentangle-network/tap/launch`)

## Quick Start

### 1. Create your fleet repo from this template

```bash
gh repo create my-org/fleet --template disentangle-network/fleet --private --clone
cd fleet
```

### 2. Configure credentials

```bash
launch setup
```

### 3. Get a Kubernetes cluster

Choose any path to a running cluster with a kubeconfig:

<details>
<summary><strong>Option A: OCI Always Free</strong></summary>

Provision a free ARM-based OKE cluster on Oracle Cloud:

```bash
brew install disentangle-network/tap/oci-tf-bootstrap
```

Clone and configure the infrastructure repo:

```bash
git clone https://github.com/LarsenClose/k8s-oci-foundation.git
cd k8s-oci-foundation

# Discover OCI resources and generate Terraform
oci-tf-bootstrap --always-free --output terraform-output

# Configure
cp environments/dev/terraform.tfvars.example environments/dev/terraform.tfvars
# Edit terraform.tfvars with your OCI and Cloudflare settings

# Review and apply
task init
task plan     # Review the plan
task apply    # Provision VCN, OKE cluster, ARM node pools
```

Your kubeconfig will be at `./kubeconfig`.

</details>

<details>
<summary><strong>Option B: Talos Linux (via Sidero Omni or bare metal)</strong></summary>

Follow the [Talos documentation](https://www.talos.dev/latest/introduction/getting-started/) to create a cluster. Your `talosconfig` and `kubeconfig` will be generated during the process.

</details>

<details>
<summary><strong>Option C: Any existing Kubernetes cluster</strong></summary>

Any cluster works -- k3s, EKS, GKE, AKS, kind, minikube. You just need a kubeconfig with cluster-admin access.

</details>

### 4. Add clusters to the fleet

```bash
cd fleet

# Add each cluster
launch cluster add oci-dev --infra cloud --arch arm64 --resources medium --nodes 3
launch cluster add my-talos --infra bare-metal --arch arm64 --resources small --nodes 3
```

### 5. Initialize secrets

```bash
launch secrets init --cluster oci-dev --provider age
launch secrets init --cluster my-talos --provider age
```

### 6. Bootstrap FluxCD

```bash
launch bootstrap --cluster oci-dev
launch bootstrap --cluster my-talos
```

FluxCD reconciles this repo and deploys disentangle-node pods to each cluster.

### 7. Verify

```bash
launch status
```

## Repository Structure

```
clusters/              Per-cluster FluxCD Kustomizations (created by launch cluster add)
apps/
  base/                HelmRelease for disentangle-node
  disentangle/         App overlay (inherits from base)
infrastructure/
  controllers/         Ingress, cert-manager, external-dns
  configs/             ClusterIssuers, NetworkPolicies
  secrets/             SOPS-encrypted infrastructure secrets
secrets/               Per-cluster SOPS-encrypted secrets
config.yaml.example    launch CLI config template
.sops.yaml             SOPS encryption rules
```

## Links

- [Disentangle Protocol](https://github.com/disentangle-network/protocol)
- [Helm Chart](https://github.com/disentangle-network/deploy)
- [Launch CLI](https://github.com/disentangle-network/launch)
- [Paper 1: TMC](https://doi.org/10.5281/zenodo.18671600)
- [Paper 2: Excitability Topology](https://doi.org/10.5281/zenodo.18695184)
