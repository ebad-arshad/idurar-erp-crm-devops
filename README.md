# IDURAR ERP/CRM - Full Stack GitOps Pipeline

This repository contains the DevOps transformation of the IDURAR ERP/CRM (MERN Stack) application. It demonstrates a complete automated lifecycle using **Jenkins** for Continuous Integration and **ArgoCD** for GitOps-based Continuous Delivery.



## 🏗 Project Structure

The project is organized into three specialized branches to maintain a clean separation between application code and infrastructure manifests:

* **`master`**: Application source code (React/Node.js) and the **Jenkinsfile**.
* **`k8s`**: Kubernetes manifests (Deployments & Services).
* **`argocd`**: ArgoCD Application definitions and environment-specific configurations.

---

## 🛠 Tech Stack

- **CI Tool:** Jenkins (Pipeline-as-Code)
- **CD Tool:** ArgoCD (GitOps)
- **Containerization:** Docker
- **Orchestration:** Kubernetes
- **Version Control:** Git (GitHub)
- **Registry:** DockerHub

---

## 🔄 The Pipeline Workflow

### 1. Continuous Integration (CI) - Jenkins
The build process is managed by Jenkins. Upon a code push to the `master` branch:
1.  **Build:** Jenkins triggers the build process using the project `Jenkinsfile`.
2.  **Dockerize:** Frontend and Backend images are created using optimized Dockerfiles.
3.  **Version Tagging:** Images are tagged using a specific **x.x** versioning scheme (e.g., `1.1`, `1.2`) instead of `latest` to ensure deployment traceability.
4.  **Push:** Images are pushed to DockerHub.



### 2. Continuous Delivery (CD) - ArgoCD
Deployment is handled via the GitOps methodology:
1.  **Source of Truth:** ArgoCD monitors the `k8s` branch for any changes to the manifests.
2.  **Automated Sync:** When a new image version is updated in the manifests, ArgoCD detects the out-of-sync status.
3.  **Deployment:** ArgoCD pulls the versioned images from DockerHub and applies the state to the Kubernetes cluster.



---

## 🚀 How to Replicate

### Prerequisites
- Jenkins server with Docker and Pipeline plugins installed.
- Kubernetes cluster with ArgoCD installed.
- DockerHub account and credentials configured in Jenkins.

### Step-by-Step
1. **Jenkins Job:** Create a Pipeline job in Jenkins pointing to the `master` branch.
2. **K8s Manifests:** Ensure the `k8s` branch contains the correct service and deployment files.
3. **ArgoCD App:** ```bash
   argocd app create idurar-erp \
     --repo [https://github.com/ebad-arshad/idurar-erp-crm-devops.git](https://github.com/ebad-arshad/idurar-erp-crm-devops.git) \
     --path . \
     --dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) \
     --revision argocd
