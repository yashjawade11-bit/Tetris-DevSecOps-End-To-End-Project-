# End-to-End DevSecOps on AWS EKS: Tetris Deployment

A hands-on DevSecOps project that provisions AWS infrastructure with Terraform and deploys a Tetris web game to Amazon EKS through a Jenkins CI/CD pipeline with security scanning, using ArgoCD for GitOps-style delivery.

> **Credit:** This project is based on the open-source work by [Aman Pathak](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-DevSecOps-Tetris-Project), licensed under Apache-2.0. See the [Acknowledgments](#acknowledgments) and [My Changes](#my-changes) sections.

---

## Architecture Overview

1. **Terraform** provisions a Jenkins server on EC2 and an EKS cluster on AWS.
2. **Jenkins** runs the CI pipeline: checkout, code analysis, dependency scan, build, image scan, push.
3. **ArgoCD** watches the Git repository and syncs the Kubernetes manifests to the EKS cluster.
4. A new app version (V2) is rolled out by updating the image tag in the manifests, and ArgoCD applies it.

```
Developer -> GitHub -> Jenkins (build + scan) -> Docker registry
                                                      |
                       GitHub (manifests) -> ArgoCD -> EKS cluster -> Tetris app
```

## Tech Stack

| Area | Tool |
|------|------|
| Cloud | AWS (EC2, EKS, IAM, VPC) |
| Infrastructure as Code | Terraform |
| CI/CD | Jenkins |
| GitOps / CD | ArgoCD |
| Containers | Docker |
| Orchestration | Kubernetes |
| Code quality | SonarQube |
| Dependency scanning | OWASP Dependency-Check |
| Image scanning | Trivy |

## Repository Structure

```
.
├── Jenkins-Server-TF/       # Terraform: Jenkins server on EC2
├── EKS-TF/                  # Terraform: EKS cluster
├── Jenkins-Pipeline-Code/   # Jenkinsfiles for CI/CD
├── Manifest-file/           # Kubernetes manifests (Deployment, Service)
├── Tetris-V1/               # Tetris app, version 1
├── Tetris-V2/               # Tetris app, version 2
├── assets/                  # Diagrams and images
└── README.md
```

## Prerequisites

- An AWS account with permissions to create EC2, EKS, IAM, and VPC resources
- AWS CLI configured (`aws configure`)
- Terraform installed
- kubectl installed
- A Docker Hub (or other registry) account
- A GitHub account

> **Cost warning:** EKS and EC2 resources are billed hourly. Run `terraform destroy` when you are finished.

## Setup Steps

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### 2. Provision the Jenkins server

```bash
cd Jenkins-Server-TF
terraform init
terraform plan
terraform apply
```

Install and configure on the server: Jenkins, Docker, Trivy, SonarQube, Terraform, kubectl, AWS CLI.

### 3. Configure Jenkins

- Unlock Jenkins and install the suggested plugins.
- Add required plugins: Docker, SonarQube Scanner, OWASP Dependency-Check, Terraform, Pipeline Stage View.
- Add credentials: GitHub, Docker registry, AWS, SonarQube token.
- Configure tools: JDK, Node.js, SonarQube Scanner, Dependency-Check, Docker.

### 4. Create the EKS cluster

Use the pipeline in `Jenkins-Pipeline-Code/` or run directly:

```bash
cd EKS-TF
terraform init
terraform apply
aws eks update-kubeconfig --region <region> --name <cluster-name>
kubectl get nodes
```

### 5. Run the CI pipeline

Create a Jenkins pipeline job pointing at the Jenkinsfile. Stages:

1. Checkout source
2. SonarQube code analysis and quality gate
3. OWASP Dependency-Check
4. Trivy filesystem scan
5. Docker build and push
6. Trivy image scan
7. Update the image tag in the manifests

### 6. Install ArgoCD and deploy

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Create an ArgoCD application pointing at `Manifest-file/` in this repository, with automated sync enabled.

### 7. Access the application

```bash
kubectl get svc
```

Open the LoadBalancer URL in a browser to play Tetris.

### 8. Roll out V2

Update the image tag in `Manifest-file/` to the V2 image. ArgoCD detects the change and syncs it to the cluster.

## Cleanup

```bash
cd EKS-TF && terraform destroy
cd ../Jenkins-Server-TF && terraform destroy
```

## Screenshots

<!-- Add your own screenshots here after deploying -->

- Jenkins pipeline run: `assets/my-jenkins-pipeline.png`
- SonarQube report: `assets/my-sonarqube.png`
- Trivy scan results: `assets/my-trivy.png`
- ArgoCD application view: `assets/my-argocd.png`
- Running application: `assets/my-app.png`

## My Changes

<!-- Replace this with what you actually changed or added. Be specific and honest. -->

- [ ] Example: replaced Jenkins with GitHub Actions
- [ ] Example: added Prometheus and Grafana monitoring
- [ ] Example: updated Terraform provider versions
- [ ] Example: fixed issues I hit during deployment (describe them)

## What I Learned

<!-- Write 3-5 points in your own words. -->

-
-
-

## Acknowledgments

This project is based on [End-to-End-Kubernetes-DevSecOps-Tetris-Project](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-DevSecOps-Tetris-Project) by Aman Pathak, used under the Apache-2.0 license. The original step-by-step guide is available in the author's [Medium blog post](https://amanpathakdevops.medium.com/devsecops-mastery-a-step-by-step-guide-to-deploying-tetris-on-aws-eks-with-jenkins-and-argocd-3adcf21b3120).

## License

Apache-2.0. See the [LICENSE](LICENSE) file.