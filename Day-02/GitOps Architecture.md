# Understanding GitOps Architecture with Argo CD

GitOps has revolutionized how modern DevOps teams manage continuous delivery for Kubernetes-native applications. In this article, we dive deep into the **architecture of GitOps tools**, with a strong focus on **Argo CD**, one of the most widely adopted tools in this space.

This guide is part of an ongoing series on GitOps and Argo CD. If you haven’t checked out Day-01 yet, it’s highly recommended before proceeding.

---

## Introduction

GitOps is a modern way to do continuous delivery using Git as a single source of truth. Tools like **Argo CD**, **Flux CD**, and **Jenkins X** implement this principle to streamline application deployment and management on Kubernetes. This article explains the core **GitOps architecture**, specifically breaking down the microservices inside **Argo CD**, so you can better understand how GitOps tools maintain the state of applications in your Kubernetes clusters.

---

## Popular GitOps Tools in the Market

Here are some of the popular GitOps tools currently used in production:

- **Argo CD** – CNCF Graduated, 13k+ GitHub stars
- **Flux CD** – CNCF Graduated
- **Jenkins X** – GitOps support for CI/CD
- **Spinnaker** – Primarily focused on deployment automation, but can be used for GitOps in hybrid scenarios

Among these, **Argo CD** stands out due to its intuitive UI, strong community support, and active contributions from companies like **Intuit, Red Hat, BlackRock, Codefresh, and Equinix**.

---

## Brief History of Argo CD

- Developed by engineers at **Applatix**
- Open-sourced and later acquired by **Intuit**
- Now maintained as a **CNCF Graduated project**
- Part of a larger ecosystem that includes:
  - Argo CD
  - Argo Workflows
  - Argo Events
  - Argo Rollouts
  - Argo Notifications

---

## What Does a GitOps Tool Do?

At its core, a GitOps tool:

1. **Connects to a Git repository** (or any version control system).
2. **Pulls the desired state** of Kubernetes resources (like deployments, services, configs).
3. **Applies the state to the Kubernetes cluster**.
4. **Continuously monitors** for drifts between the Git repository and the actual cluster state.
5. **Performs reconciliation** if a difference is detected.

This provides **self-healing** and **automated rollback** capabilities that traditional CD pipelines using shell scripts or Jenkins pipelines often lack.

---

## Argo CD Architecture Breakdown

Let's explore the **microservice-based architecture** of Argo CD. Understanding this helps you grasp the internals of any GitOps tool.

### 1. **Repo Server**

- Connects to Git and **fetches the desired state** (manifests, Helm charts, etc.)
- Continuously monitors Git repositories for changes
- Works with various Git providers: GitHub, GitLab, Bitbucket, etc.

### 2. **Application Controller**

- Talks to the **Kubernetes API server**
- Checks the **actual state** of applications in the cluster
- Performs **reconciliation** by comparing the actual vs desired state
- Initiates synchronization if drift is detected

### 3. **API Server**

- Acts as the **frontend/backend server**
- Enables interaction through:
  - **CLI**
  - **Web UI**
  - **REST APIs**
- Supports:
  - Authentication and authorization
  - Role-Based Access Control (RBAC)
  - OpenID Connect (OIDC)

### 4. **Dex (OIDC Provider)**

- Lightweight identity provider used for **SSO integration**
- Acts as a bridge between Argo CD and external identity providers (e.g., Google, GitHub, LDAP)
- Enables secure enterprise authentication workflows

### 5. **Redis (Cache Layer)**

- Caches application states and metadata
- Ensures **stateful behavior** of the application controller
- Essential for system recovery and performance optimization

---

## Argo CD Architecture Diagram (Textual View)

```txt
+----------------+       +--------------------+
|   Git Repo     |       |  Kubernetes Cluster|
+--------+-------+       +----------+---------+
         |                          |
         v                          v
+----------------+       +------------------------+
|   Repo Server  |<----->|  Application Controller|
+----------------+       +------------------------+
         |                          |
         |                          v
         |                 +---------------------+
         |                 |   Redis (Caching)   |
         |                 +---------------------+
         |
         v
+----------------+       +---------------------+
|   API Server   |<----->|     Dex (SSO)       |
+----------------+       +---------------------+
        ^
        |
+----------------+
|    Web UI /    |
|     CLI Tool   |
+----------------+
```

## Key GitOps Principles Illustrated

- **Single Source of Truth**: Git stores all desired configurations.
- **Declarative Configurations**: Kubernetes manifests or Helm charts are versioned in Git.
- **Automated Reconciliation**: Argo CD ensures cluster state matches Git.
- **Self-Healing**: If drift occurs (e.g., manual changes to cluster), Argo CD reverts it.
- **Audit Trail**: Git history serves as an auditable change log.

---

## Real-World Use Case: Handling Admission Controllers

Consider this scenario:

- You have a `pod.yaml` in Git with no resource limits.
- Your Kubernetes cluster enforces limits via **admission controllers**.
- These controllers may mutate incoming requests, adding defaults or policies.

**What happens?**

- Argo CD detects a drift since the live state includes additional fields.
- It may **override** the live configuration with Git’s version, potentially causing failures if mandatory fields are missing.

**Solution:**

- Use **`ignoreDifferences`** setting in Argo CD to avoid reconciling certain fields.
- Use **Kustomize** overlays or strategic merging to handle mutations gracefully.

---

## Best Practices for Adopting GitOps

- Start small: Use GitOps for a subset of services before full migration.
- Store only declarative configurations in Git (avoid secrets in plain text).
- Use tools like **Sealed Secrets**, **Vault**, or **SOPS** for secure secret management.
- Integrate SSO with Argo CD using OIDC for secure access control.
- Enable audit logging to track sync history and user activities.
- Regularly test syncs in staging before applying changes to production.

---

## How to Install Argo CD

Argo CD can be installed via multiple methods:

- **YAML Manifests**:
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

- **Helm Chart**:
  ```bash
  helm repo add argo https://argoproj.github.io/argo-helm
  helm install argocd argo/argo-cd --namespace argocd --create-namespace
  ```

- **Kubernetes Operators**:
  - Use Argo CD operator to manage lifecycle declaratively

---

## What’s Next?

In the next part of this series, we’ll:

- Perform a **live demo of Argo CD**
- Deploy a sample application using GitOps
- Cover advanced GitOps scenarios:
  - How to onboard existing resources
  - Drift detection strategies
  - Interview-level questions on GitOps architecture

---

## Conclusion

Understanding the internal architecture of GitOps tools like **Argo CD** empowers you to build resilient, self-healing, and auditable continuous delivery systems. With strong community backing and CNCF graduation, Argo CD is poised to remain a leading tool in the GitOps space.

Whether you're a DevOps Engineer, Cloud Architect, or SRE, mastering GitOps is essential in the cloud-native era.
