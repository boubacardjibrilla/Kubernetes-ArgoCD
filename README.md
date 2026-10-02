Kubernetes GitOps Deployment with Argo CD & kOps on AWS

This repository demonstrates a GitOps continuous delivery setup on AWS using Argo CD and Kubernetes provisioned via kOps.

🏗️ Architecture Overview

The GitOps workflow follows an automated declarative deployment pattern:

Developers/DevOps push Kubernetes manifest updates (deployment.yaml, service.yaml, ingress.yaml) to the GitHub repository.

Argo CD (Deployment Controller) continuously monitors the Git repository for changes.

Upon detecting a diff, Argo CD syncs and applies the manifests directly to the Kubernetes cluster (AWS managed via kOps).

Users and clients access the deployed applications and Argo CD UI through an AWS Elastic Load Balancer (ELB).

🛠️️ Tech Stack & Prerequisites

Infrastructure & Tools

Cloud Provider: AWS (EC2, S3, IAM, Route53)

Cluster Management: kOps (Kubernetes Operations)

Container Orchestration: Kubernetes (1 Master Node, 2 Worker Nodes)

Continuous Delivery (GitOps): Argo CD

Configuration Management: Kubernetes Declarative Manifests (yaml)

CLI Prerequisites

Before starting, ensure you have installed and configured:

AWS CLI (aws configure)

kubectl

kOps CLI

🚀 Step-by-Step Deployment Guide

1. Provision Kubernetes Cluster with kOps

Set up the S3 state store and create the cluster on AWS:

# Set up environment variables
export BUCKET_NAME=my-kops-state-store-bucket
export CLUSTER_NAME=mycluster.k8s.local

# Create S3 Bucket for state store
aws s3api create-bucket --bucket $BUCKET_NAME --region us-east-1

# Create the cluster configuration (1 Master, 2 Workers)
kops create cluster \
  --name=$CLUSTER_NAME \
  --state=s3://$BUCKET_NAME \
  --zones=us-east-1a \
  --node-count=2 \
  --node-size=t3.medium \
  --control-plane-size=t3.medium \
  --control-plane-zones=us-east-1a

# Build and apply cluster configuration
kops update cluster --name $CLUSTER_NAME --state=s3://$BUCKET_NAME --yes --admin


Verify cluster readiness:

kops validate cluster --state=s3://$BUCKET_NAME
kubectl get nodes


2. Install Argo CD on the Cluster

Create the argocd namespace and apply installation manifests:

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Expose the Argo CD API server via LoadBalancer:

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'


Retrieve the initial admin password:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


3. Connect Repository & Deploy Application

Access the Argo CD UI via the External LoadBalancer URL.

Log in with admin and the retrieved initial password.

Create a new Application targeting your repository (ar-deploy.git):

Application Name: my-app

Project: default

Sync Policy: Automatic (or Manual)

Repository URL: https://github.com/<your-username>/<your-repo>.git

Path: ./ (or folder containing manifests)

Cluster URL: https://kubernetes.default.svc

Namespace: default

📊 Application Status & Verification

Once synced, check the application status from kubectl or the Argo CD UI:

kubectl get pods -o wide
kubectl get svc


📂 Repository Structure

.
├── deployment.yaml                            # Deployment manifest for application pods
├── service.yaml                               # LoadBalancer/NodePort service definitions
├── ingress.yaml                               # Ingress rules (if applicable)
├── Architecture.png                           # Diagram illustrating GitOps workflow
├── ArgoDC.png                                 # Screenshot of Argo CD Dashboard
├── App-running.png                            # Screenshot of application running
└── guide_de_d_ploiement_et_r_solution_de_probl_mes.md  # Detailed deployment & troubleshooting guide


🧹 Cleanup

To avoid incurring AWS charges, tear down all resources when finished:

# Delete Kubernetes Cluster via kOps
kops delete cluster --name $CLUSTER_NAME --state=s3://$BUCKET_NAME --yes

# Delete S3 bucket
aws s3 rb s3://$BUCKET_NAME --force
