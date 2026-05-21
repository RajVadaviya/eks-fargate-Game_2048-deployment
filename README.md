# Deploying 2048 Game on Amazon EKS using Fargate and AWS Load Balancer Controller

This project demonstrates deploying the 2048 game application on Amazon EKS using Kubernetes, AWS Fargate, and AWS Load Balancer Controller.

The application is exposed publicly using an AWS Application Load Balancer (ALB) through Kubernetes Ingress.

---

# Project Overview

In this project, I:

- Installed and configured AWS CLI
- Installed kubectl
- Installed eksctl
- Created an Amazon EKS Cluster
- Configured AWS Fargate Profile
- Deployed the 2048 application on Kubernetes
- Configured IAM OIDC Provider
- Created IAM Policy for AWS Load Balancer Controller
- Installed AWS Load Balancer Controller using Helm
- Exposed the application using Kubernetes Ingress and AWS ALB

---

# Architecture

```text
User
   ↓
AWS Application Load Balancer (ALB)
   ↓
Kubernetes Ingress
   ↓
Kubernetes Service
   ↓
2048 Application Pods running on AWS Fargate
```

---

# Technologies Used

- AWS EKS
- AWS Fargate
- Kubernetes
- kubectl
- eksctl
- Helm
- AWS Load Balancer Controller
- IAM OIDC Provider
- AWS ALB
- Docker

---

# Prerequisites

Before starting:

- AWS Account
- IAM User with AdministratorAccess
- AWS CLI installed and configured
- kubectl installed
- eksctl installed
- Helm installed

---

# Step 1: Install kubectl

Download kubectl:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Make it executable:

```bash
chmod +x kubectl
```

Move to system path:

```bash
sudo mv kubectl /usr/local/bin/
```

Verify installation:

```bash
kubectl version --client
```

---

# Step 2: Install AWS CLI

Verify installation:

```bash
aws --version
```

Configure AWS CLI:

```bash
aws configure
```

Verify AWS connection:

```bash
aws sts get-caller-identity
```

---

# Step 3: Install eksctl

Download and install eksctl:

```bash
curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```

```bash
sudo mv /tmp/eksctl /usr/local/bin
```

Verify installation:

```bash
eksctl version
```

---

# Step 4: Create Amazon EKS Cluster with Fargate

```bash
eksctl create cluster \
  --name demo-cluster-1 \
  --region us-east-1 \
  --fargate
```

Update kubeconfig:

```bash
aws eks update-kubeconfig \
  --name demo-cluster-1 \
  --region us-east-1
```

Verify cluster:

```bash
kubectl get nodes
```

---

# Step 5: Create Fargate Profile

```bash
eksctl create fargateprofile \
  --cluster demo-cluster-1 \
  --region us-east-1 \
  --name alb-sample-app \
  --namespace game-2048
```

---

# Step 6: Deploy 2048 Application

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

Verify pods:

```bash
kubectl get pods -n game-2048
```

Verify service:

```bash
kubectl get svc -n game-2048
```

Verify ingress:

```bash
kubectl get ingress -n game-2048
```

---

# Step 7: Configure IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster-1 \
  --approve
```

---

# Step 8: Create IAM Policy for AWS Load Balancer Controller

Download IAM policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

Create IAM policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

# Step 9: Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster-1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<AWS-ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

---

# Step 10: Install Helm

Install Helm using Snap:

```bash
sudo snap install helm --classic
```

Verify installation:

```bash
helm version
```

---

# Step 11: Install AWS Load Balancer Controller

Add Helm repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
```

Update Helm repository:

```bash
helm repo update eks
```

Install AWS Load Balancer Controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster-1 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<YOUR-VPC-ID>
```

Verify deployment:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

Verify pods:

```bash
kubectl get pods -n kube-system
```

---

# Step 12: Access the Application

Get ingress URL:

```bash
kubectl get ingress -n game-2048
```

Example Output:

```text
k8s-game2048-ingress2-xxxxxxxx.us-east-1.elb.amazonaws.com
```

Open the URL in browser to access the 2048 game.

---

# Kubernetes Resources Created

- Namespace
- Deployment
- Pods
- Service
- Ingress
- Fargate Profile
- IAM Service Account
- ALB Controller

---

# Learning Outcomes

Through this project, I learned:

- Kubernetes fundamentals
- Amazon EKS cluster setup
- AWS Fargate integration
- Kubernetes Ingress
- AWS Application Load Balancer
- IAM OIDC Provider
- AWS Load Balancer Controller
- Helm package management
- Kubernetes resource management using kubectl

---

# Screenshots

## 2048 Game Running on Browser

<img width="1919" height="1030" alt="image" src="https://github.com/user-attachments/assets/3c64497f-3fa2-4530-8c0b-6e353e4712f6" />


---

# Cleanup

Delete cluster to avoid AWS charges:

```bash
eksctl delete cluster \
  --name demo-cluster-1 \
  --region us-east-1
```

---

# Reference

Project inspired from:

:contentReference[oaicite:0]{index=0}

---

# Author

Raj Vadaviya

Learning Cloud & DevOps 🚀
