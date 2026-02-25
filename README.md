# k8s-argo-cd-practice

A learning repository for practicing GitOps with ArgoCD on a self-managed Kubernetes cluster built with kubeadm on GCP.

## What is GitOps?

GitOps is a way of managing Kubernetes applications where Git is the single source of truth for the desired cluster state. Instead of running `kubectl apply` by hand, you commit Kubernetes manifests to a Git repository and a tool (ArgoCD in this case) watches that repository and automatically syncs the cluster to match.

Benefits include:
- Every change is a Git commit with an author, timestamp, and message
- Full audit trail of who changed what and when
- Easy rollbacks by reverting a Git commit
- Self-healing — if someone manually changes something in the cluster, ArgoCD automatically reverts it back to what Git says it should be
- No one needs direct kubectl access to deploy

## Cluster Setup

This repo is used with a 3-node Kubernetes cluster running on GCP Compute Engine:
- 1 control plane node (e2-standard-2)
- 2 worker nodes (e2-medium)
- Kubernetes v1.35 installed via kubeadm
- Calico for pod networking (CNI)
- ArgoCD for GitOps-based continuous deployment

## Repository Structure
```
k8s-argo-cd-practice/
└── apps/
    └── nginx/
        ├── deployment.yaml   # nginx Deployment with 2 replicas
        └── service.yaml      # ClusterIP Service exposing port 80
```

## Apps

### nginx-argo
A simple nginx deployment used to demonstrate ArgoCD's core GitOps workflow including:
- Automatic sync from Git to cluster
- Self-healing when cluster state drifts from Git
- Auto-pruning when resources are removed from Git

## ArgoCD

ArgoCD is installed in the `argocd` namespace and accessed via kubectl port-forward:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open https://localhost:8080 in your browser.

## Key ArgoCD Concepts Demonstrated

**Automated Sync** — ArgoCD polls this repo every 3 minutes and automatically applies any changes it detects, no manual intervention needed.

**Auto-prune** — If a manifest is deleted from this repo, ArgoCD deletes the corresponding resource from the cluster.

**Self-heal** — If someone manually modifies a resource in the cluster (via kubectl), ArgoCD detects the drift and reverts it back to match Git.

## Usage

To make a change to the nginx deployment, edit the manifests in `apps/nginx/` and push to main. ArgoCD will detect the change and sync automatically within a few minutes.

To force an immediate sync:
```bash
argocd app sync nginx-argo
```

To check application status:
```bash
argocd app get nginx-argo
```

To check sync history:
```bash
argocd app history nginx-argo
```