# End-to-End DevOps Project: From IaC to CI/CD on Kubernetes

This repository demonstrates a complete, automated DevOps project, covering the lifecycle from infrastructure provisioning to CI/CD deployment on Kubernetes. It is designed to be repeatable, idempotent, and real-world ready.

## Project Goal

Build a fully automated DevOps environment from scratch, simulating a modern software delivery lifecycle using a containerized toolset. The focus is on automation, idempotency, and best practices.

## Architecture & Workflow

The workflow is fully automated:

1. Developer pushes code to Gitea.
2. Gitea webhook triggers the Jenkins pipeline.
3. Jenkins checks out the code, runs tests, and builds the application.
4. Kaniko builds a Docker image inside Kubernetes (avoiding Docker-in-Docker).
5. The image is pushed to a private Nexus registry.
6. Jenkins deploys the updated application to the Kubernetes cluster.

## Technology Stack

| Category | Tool | Purpose |
|----------|------|---------|
| Infrastructure as Code | Ansible | Idempotent provisioning of Kubernetes cluster |
| Orchestration | Kubernetes v1.30.14 | Cluster for running services and apps |
| Container Runtime | containerd | Cluster's CRI |
| Network Plugin | Calico | Cluster networking (CNI) |
| CI/CD Server | Jenkins | Orchestrates build/test/deploy pipeline |
| Artifact Registry | Nexus | Hosts private Docker images |
| Source Control | Gitea | Self-hosted Git server |
| Database | MySQL | Backend for Gitea |
| Container Builder | Kaniko | Builds images securely inside Kubernetes |
| Package Manager | Helm | Simplifies Jenkins/Nexus deployment |

## Prerequisites

- 1 CentOS Linux 7 VM (1 master)
- Ansible installed on your local/control node
- SSH key-based access from control node to all K8s nodes

## Steps to Run the Project
```bash
### 1. Provision the Cluster

Update the `inventory.ini` file with your node IP addresses, then run:
ansible-playbook k8s.yml
This sets up Kubernetes prerequisites , containerd, and Calico ,Helm.

### 2. Deploy the CI/CD Toolchain
Deploy Gitea & MySQL:
kubectl apply -f gitee.yml/

### 3. Deploy Jenkins & nexus:
---Note this Local cluster so i create a local storage class first and need for a local provisioner -----
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl apply -f sc.yml
helm repo add jenkins https://charts.jenkins.io
helm repo add sonatype https://sonatype.github.io/helm3-charts/
helm repo update
helm install jenkins jenkins/jenkins -n jenkins-ns  --set controller.nodePort=30000
helm install nexus sonatype/nexus-repository-manager -n nexus --set nexusProxy.service.type=NodePort --set nexusProxy.service.nodePort=30050 --set persistence.enabled=true

### 4. Configure the Pipeline
Gitea: Create a repository for the sample application and configre webhook .
---Note expose the ssh port to connect to the repo EX: git clone ssh://git@10.10.70.84:30062/ahmed_zeid/DevOps.git
kubectl apply -f  nexus-svc.yml # To expose the ssh
Nexus: Create a Docker hosted repository.
---Note expose the HTTP API port to connect to the registry.
--- Note 2: you can get the password to access the nexus EX:kubectl exec -n nexus -it nexus-nexus-repository-manager-6d5c4db9f9-abcde -- cat /nexus-data/admin.password

Jenkins:
Create a Pipeline job pointing to the Jenkinsfile in Gitea.
Configure Gitea webhook for push events.
Add credentials for Nexus and Kubernetes.

### 4. Trigger the First Build
1-Push your application code to Gitea. Jenkins will automatically:

2-Run tests

3-Build the Docker image using Kaniko

4-Push to Nexus

5-Deploy the application to Kubernetes
-----------------------------------------------------------------------------------------------------------

├── ansible-playbooks/      # Ansible playbooks for K8s, containerd, Calico
├── gitea-manifests/        # Deployment/Service YAMLs for Gitea/MySQL
├── helm-charts/            # Custom values for Jenkins/Nexus
├── sample-application/     # Sample Node.js/Python/Go app
│   └── Dockerfile          # Dockerfile for Kaniko build
├── Jenkinsfile             # Main declarative pipeline
└── README.md               # Project documentation
----------------------------------------------------------------------------------------------------------
## Attached Repository Contents

This repository includes all the essential components used to build and automate the environment:

1. **Kubernetes Manifests** — All YAML files used to deploy and configure the cluster resources.  
2. **Ansible Directory** — Contains the Ansible configuration files and playbooks for setting up Kubernetes, containerd, Calico, and Helm.  
3. **Application Directory** — Includes the sample application source code, its `Dockerfile`, and the `Jenkinsfile` used for the CI/CD pipeline.

Thanks !!!!!!!
Ahmed Ibrahim Zeid
LinkedIn: www.linkedin.com/in/ahmed-zeid-a64b21204

