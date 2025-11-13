# AWS EKS Three-Tier Application Deployment 🚀

This repository demonstrates how to deploy a **three-tier web application** (frontend, backend, and database) on **Amazon EKS** using the **AWS Load Balancer Controller** for ingress and **EBS CSI driver** for persistent storage.

---

## 🧩 Project Overview

- **Cluster Setup:** Amazon EKS Cluster created using `eksctl`
- **Storage:** AWS EBS CSI Driver for persistent volumes
- **Ingress:** AWS Application Load Balancer (ALB) Controller
- **Deployment:** Three-tier application deployed via Helm
- **Namespace:** `robot-shop`
- **Access:** Application exposed using Load Balancer DNS

---

## 📸 Screenshots

- Pods status → `screenshots/pods_running.png`
- Deployments status → `screenshots/deployments_running.png`
- Ingress configuration → `screenshots/ingress_configuration.png`
- Application running via Load Balancer → `screenshots/application_running.png`

## 🛠️ Tools & Technologies Used

- **Amazon EKS**
- **AWS Load Balancer Controller**
- **EBS CSI Driver**
- **Helm**
- **kubectl**
- **eksctl**
- **AWS IAM Roles & Policies**
- **Kubernetes (Pods, Deployments, Services, Ingress)**

---

## 🚀 Deployment Steps

### 1. Create an EKS Cluster
```bash
eksctl create cluster --name demo-cluster-three-tier-1 --region us-east-1
```
### 2. Associate IAM OIDC Provider
``` bash
export cluster_name=<CLUSTER-NAME>
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
### 3. Create IAM Policy for AWS Load Balancer Controller
``` bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
### 4. Create Service Account for ALB Controller
``` bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
### 5. Install AWS Load Balancer Controller using Helm
``` bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
Check the deployment:
``` bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
### 6. Create IAM Role for EBS CSI Driver
``` bash
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <YOUR-CLUSTER-NAME> \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
### 7. Install EBS CSI Driver Addon
``` bash
eksctl create addon --name aws-ebs-csi-driver \
  --cluster <YOUR-CLUSTER-NAME> \
  --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole \
  --force
```
### 8. Clone the Repository
``` https://github.com/satheesh-kamadani/aws-eks-three-tier-app.git
cd aws-eks-three-tier-app/EKS/helm
```
### 9. Deploy the Application
``` bash
kubectl create ns robot-shop
helm install robot-shop --namespace robot-shop .
```
Validation
``` bash
kubectl get pods -n robot-shop
kubectl get svc -n robot-shop
kubectl get ingress -n robot-shop
```
Get the Load Balancer DNS:
``` bash
kubectl get ingress -n robot-shop
``` 
Access the app in your browser using the LoadBalancer DNS URL.
