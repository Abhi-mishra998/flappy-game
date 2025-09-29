# 🐦 Flappy Bird Game on Amazon EKS  

This project demonstrates how to deploy a **Flappy Bird game** on **Amazon Elastic Kubernetes Service (EKS)** using **AWS Fargate** and the **AWS Load Balancer Controller** for Ingress.  

It is designed as a **DevOps/DevSecOps learning project** showcasing skills in:  
- Kubernetes (EKS)  
- AWS Cloud (EKS, IAM, Fargate, ALB)  
- Helm & eksctl  
- CI/CD-ready workloads  

---

## 🚀 Architecture  

(https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/images/eks-alb.png)  

**Components:**  
- **EKS Cluster** – Managed Kubernetes cluster on AWS.  
- **Fargate Profile** – Serverless compute for pods.  
- **AWS Load Balancer Controller** – Manages ALBs for Kubernetes Ingress.  
- **Flappy Bird Game App** – Containerized game deployed as a Kubernetes Deployment + Service + Ingress.  

---

## 🛠️ Prerequisites  

- AWS Account with IAM Admin permissions  
- AWS CLI, kubectl, eksctl, and helm installed  
- Docker (optional, if you want to build your own image)  

---

## 📦 Deployment Steps (All-in-One)  

Copy and run the following commands in order 👇  

```bash
# 1. Create EKS Cluster with Fargate
eksctl create cluster \
  --name Abhishek-cluster \
  --region ap-south-1 \
  --fargate

# 2. Associate IAM OIDC Provider
eksctl utils associate-iam-oidc-provider \
  --cluster Abhishek-cluster \
  --approve \
  --region ap-south-1

# 3. Download and create an IAM Policy for ALB Controller
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# 4. Create IAM Service Account for ALB Controller
eksctl create iamserviceaccount \
  --cluster Abhishek-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# 5. Install AWS Load Balancer Controller via Helm
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=Abhishek-cluster

# 6. Deploy the Flappy Bird Game
kubectl create namespace flappy-game
kubectl apply -f https://raw.githubusercontent.com/Abhi-mishra998/flappy-bird-k8s/main/flappy-bird.yaml

# 7. Verify pods, services, and ingress
kubectl get pod -n kube-system
kubectl get pods -n kube-system -w
kubectl get svc -n kube-system
kubectl get ingress -n flappy-game

# 8. (Optional) Re-run OIDC association if needed
eksctl utils associate-iam-oidc-provider \
  --cluster Abhishek-cluster \
  --approve \
  --region ap-south-1

# 9. (Optional) Create IAM policy from local file (Windows path example)
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://C:\Users\USER\Downloads\EKS-Project\flappy-bird-k8s\iam_policy.json

---
## Accessing the Game
Once the Ingress is created, an AWS Application Load Balancer (ALB) will be provisioned.
Get the ALB DNS name:
kubectl get ingress -n flappy-game
Open the DNS URL in your browser → Enjoy Flappy Bird 🎮"
