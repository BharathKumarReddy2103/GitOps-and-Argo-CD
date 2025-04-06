# Deep Dive into Argo CD: GitOps Hands-On for DevOps

## Introduction

In today’s fast-paced DevOps environments, **GitOps** has emerged as a preferred practice for managing infrastructure and applications through Git-based workflows. **Argo CD** is one of the most widely adopted tools for implementing GitOps in Kubernetes. This guide offers an in-depth, hands-on tutorial on Argo CD, covering everything from installation to deploying applications using both the UI and CLI. Whether you're a beginner or a practicing DevOps Engineer, this article will help you understand the practical implementation of Argo CD.

---

## Table of Contents

1. [What is Argo CD?](#what-is-argo-cd)
2. [Installation Methods](#installation-methods)
   - Using Plain Kubernetes Manifests
   - Using Helm Charts
   - Using Argo CD Operator
3. [Argo CD Architecture Overview](#argo-cd-architecture-overview)
4. [Hands-On: Installing Argo CD on Minikube](#hands-on-installing-argo-cd-on-minikube)
5. [Accessing Argo CD UI](#accessing-argo-cd-ui)
6. [Deploying Applications via UI](#deploying-applications-via-ui)
   - Plain YAML
   - Helm Charts
7. [Deploying Applications via CLI](#deploying-applications-via-cli)
8. [Best Practices & Tips](#best-practices--tips)
9. [Conclusion](#conclusion)

---

## What is Argo CD?

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. It enables teams to:

- Continuously sync Kubernetes manifests from Git repositories.
- Automatically apply updates based on Git commits.
- Visually manage and track deployment states.

---

## Installation Methods

### 1. Using Plain Kubernetes Manifests

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This method installs all Argo CD components including:
- API Server
- Application Controller
- Repo Server
- Dex (SSO)
- Redis
- UI Server

### 2. Using Helm Charts

Add the Argo Helm repo and install:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd --namespace argocd --create-namespace
```

### 3. Using Argo CD Operator

You can install Argo CD using the Operator pattern via tools like OperatorHub or Helm-based Operator. Refer to [Argo CD Operator GitHub](https://github.com/argoproj-labs/argocd-operator) for details.

---

## Argo CD Architecture Overview

Key components:

- **API Server**: Interfaces with users via CLI or UI.
- **Repo Server**: Fetches manifests from Git repositories.
- **Application Controller**: Performs reconciliation between Git state and live Kubernetes state.
- **Dex**: Provides SSO/OIDC integration.
- **Redis**: Manages caching and state.
- **ApplicationSet Controller**: Dynamically creates applications from templates.

---

## Hands-On: Installing Argo CD on Minikube

1. Ensure Minikube is running:

```bash
minikube start
minikube status
```

2. Install Argo CD using manifests:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. Watch pods come up:

```bash
kubectl get pods -n argocd -w
```

---

## Accessing Argo CD UI

1. Expose the Argo CD Server:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

2. Use Minikube tunnel to access the UI:

```bash
minikube service argocd-server -n argocd
```

3. Get the initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

Use `admin` as username and the decoded password to log in.

---

## Deploying Applications via UI

### Example Repository: [Argo CD Example Apps](https://github.com/argoproj/argocd-example-apps)

1. Click "Create Application"
2. Provide:
   - App Name: `my-first-app`
   - Repo URL: `https://github.com/argoproj/argocd-example-apps`
   - Path: `guestbook`
   - Destination: In-cluster
   - Namespace: `default`
3. Enable Auto-Sync (optional)
4. Click Create

Argo CD will automatically sync and deploy the `Deployment` and `Service` manifests.

---

### Deploying Helm Chart

Path: `helm-guestbook`

1. Use the same Git repo and change the path to `helm-guestbook`
2. Argo CD will detect Helm structure:
   - `Chart.yaml`
   - `values.yaml`
   - `templates/`

Argo CD renders Helm templates and applies them to the cluster.

---

## Deploying Applications via CLI

Install CLI:

```bash
brew install argocd
```

Log in:

```bash
argocd login <ARGOCD_SERVER> --username admin --password <PASSWORD> --insecure
```

Create app via CLI:

```bash
argocd app create guestbook \
--repo https://github.com/argoproj/argocd-example-apps \
--path guestbook \
--dest-server https://kubernetes.default.svc \
--dest-namespace default \
--sync-policy automated
```

Sync manually:

```bash
argocd app sync guestbook
```

---

## Best Practices & Tips

- Use separate namespaces for each controller (e.g., `istio`, `argocd`, etc.)
- Enable auto-sync with pruning and self-heal options for production
- Store secrets outside of Git or use tools like Sealed Secrets or External Secrets
- Use ApplicationSets for managing multiple clusters or apps
- Choose the appropriate folder structure (plain YAML, Helm, Kustomize) based on your team’s tooling

---

## Conclusion

Argo CD simplifies and automates Kubernetes deployments by embracing GitOps principles. Whether you use plain manifests, Helm charts, or Kustomize, Argo CD provides a powerful UI and CLI to manage your application lifecycle declaratively. By integrating Argo CD into your DevOps workflow, you ensure **consistency**, **auditability**, and **automation** across environments. Experiment with real-world applications using the official example repository, and master Argo CD both through UI and CLI.

Start small. Automate more. Embrace GitOps.

---

*If you found this article helpful, star the repository and follow for more hands-on DevOps and Cloud tutorials.*
