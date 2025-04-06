# Deploying Applications to Multiple Kubernetes Clusters Using Argo CD:

## Introduction

Managing deployments across multiple Kubernetes clusters can be complex and error-prone, especially when dealing with different environments like development, QA, staging, and production. In this article, we explore how to simplify and standardize this process using **Argo CD**, a GitOps-based continuous delivery tool.

We'll dive into multi-cluster deployments, understand the benefits of GitOps, compare Hub-Spoke and Standalone models, and walk through a complete hands-on setup of deploying applications to multiple clusters using Argo CD.

---

## What is Argo CD and Why GitOps?

**Argo CD** is a declarative, GitOps continuous delivery tool for Kubernetes. Instead of relying on traditional shell scripts, Ansible playbooks, or CI plugins for deployment, Argo CD pulls the desired state directly from Git and applies it to your Kubernetes clusters.

### Benefits of GitOps and Argo CD:
- **Declarative and version-controlled deployments**
- **Automatic drift detection** and self-healing from unauthorized changes
- **Improved security** by limiting manual access to production
- **Auditable change history** via Git commits
- **Simplified rollback** using Git history

---

## Real-World Problem with Traditional CD Pipelines

In a traditional CI/CD pipeline (e.g., using Jenkins), deployment often happens via custom shell scripts, Ansible, or plugins. While effective, this approach has drawbacks:

- Manual changes on clusters go undetected
- No unified source of truth for deployment configurations
- Difficult to track unauthorized modifications
- No auto-healing mechanism to revert undesirable changes

**GitOps** solves this by treating Git as the single source of truth for both application and infrastructure configurations.

---

## Multi-Cluster Deployment with Argo CD

### Why Multi-Cluster?

In real-world organizations, different teams or features often require their own isolated Kubernetes clusters:
- dev, qa, staging, production environments
- feature-specific clusters for isolated testing
- tenant-based architectures in SaaS environments

Deploying the same application across these clusters (e.g., a `guestbook` app) becomes a common need.

---

## Hub-Spoke vs Standalone Argo CD Deployment Models

### Standalone Model

- Each cluster has its **own Argo CD instance**
- Manages only the resources of that cluster
- Good for decentralized teams with isolated responsibilities

### Hub-Spoke Model (Used in This Demo)

- One **central Argo CD instance** (Hub) manages deployments across **multiple clusters** (Spokes)
- Easier for centralized DevOps teams
- Simplifies upgrades, backup, and disaster recovery
- Requires high availability for scalability

---

## Hands-On: Multi-Cluster Argo CD Setup

### Step 1: Create Three EKS Clusters

Using `eksctl`, create:
- `hub-cluster`: for installing Argo CD
- `spoke-cluster-1`: target cluster 1
- `spoke-cluster-2`: target cluster 2

```bash
eksctl create cluster --name hub-cluster ...
eksctl create cluster --name spoke-cluster-1 ...
eksctl create cluster --name spoke-cluster-2 ...
```

### Step 2: Install Argo CD on the Hub Cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 3: Expose Argo CD UI via NodePort

Edit the Argo CD server service:

```bash
kubectl edit svc argocd-server -n argocd
# Change type from ClusterIP to NodePort
```

### Step 4: Enable HTTP (Insecure) Access

Edit the Argo CD CMD config map:

```bash
kubectl edit configmap argocd-cmd-params-cm -n argocd
# Add:
# data:
#   server.insecure: "true"
```

Restart Argo CD server to apply changes.

### Step 5: Access Argo CD UI

Use the public IP of any EKS node:

```
http://<node-public-ip>:<nodePort>
```

Retrieve the initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

Login as `admin`.

---

## Step 6: Add Spoke Clusters to Argo CD

Install Argo CD CLI and authenticate:

```bash
argocd login <ARGO_CD_UI_IP>:<port> --username admin --password <password> --insecure
```

Get available contexts:

```bash
kubectl config get-contexts
```

Add clusters:

```bash
argocd cluster add <spoke-cluster-context-1>
argocd cluster add <spoke-cluster-context-2>
```

Refresh Argo CD UI → Settings → Clusters to verify.

---

## Step 7: Deploy Application to Multiple Clusters

### Option 1: Manually Create Argo CD Applications for Each Cluster

Navigate to **Applications > Create New Application**:

Fill in:
- **Name**: `guestbook-spoke1`
- **Repo URL**: `<your-public-repo-url>`
- **Path**: `manifests/guestbook`
- **Destination Cluster**: `spoke-cluster-1`
- **Namespace**: `default`
- **Sync Policy**: Automatic

Repeat the same for `spoke-cluster-2`.

### Option 2 (Coming Soon): ApplicationSets

Use Argo CD **ApplicationSets** to create multiple applications dynamically. Ideal for managing 10+ clusters with a single config.

---

## Demo: Auto Sync & Auto Heal

1. Edit a config map value directly in Git.
2. Argo CD detects the change and **auto-syncs** the new version to the cluster.
3. Now, simulate a **manual change** to the same config map using `kubectl`.
4. Argo CD **detects the drift** and automatically **reverts it** to match Git.

This is GitOps in action — no manual overrides go unnoticed.

---

## Best Practices

- Always configure **Argo CD with HTTPS** in production.
- Keep all Kubernetes manifests in version-controlled Git repos.
- Set up **RBAC** policies in Argo CD for cluster access control.
- Enable **webhooks** or periodic sync for faster reconciliation.
- Use **ApplicationSets** for large-scale multi-cluster deployments.

---

## Conclusion

Deploying applications across multiple Kubernetes clusters can be simplified and made reliable using **Argo CD and GitOps principles**. Whether you're a solo DevOps engineer or part of a centralized platform team, Argo CD provides the observability, automation, and safety net required for modern cloud-native deployments.

By implementing the **Hub-Spoke** model and adopting **Git as the source of truth**, you ensure consistency, traceability, and resilience across all environments.
