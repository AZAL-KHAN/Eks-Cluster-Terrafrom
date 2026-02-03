# EKS Cluster using Terraform

This repository provisions an Amazon EKS cluster using **Terraform**.

## Features
- Kubernetes v1.34
- Managed Node Groups
- Auto Scaling (3–5 nodes)
- Public Subnets Only
- No NAT Gateway (Cost Optimized)

## Tech Stack
- Terraform
- AWS EKS
- IAM
- VPC

## 📁 Repository Structure
```
Eks-Cluster-Terrafrom/
├── main.tf            # Terraform & provider versions
├── provider.tf        # AWS provider configuration
├── variables.tf       # Input variables
├── vpc.tf             # VPC, subnets, routing (No NAT Gateway)
├── iam.tf             # IAM roles & policies for EKS
├── eks.tf             # EKS cluster & managed node group
├── outputs.tf         # Useful outputs
└── README.md
```

## Deployment

terraform init  
terraform apply

---

## Configure kubeconfig

Update local kubeconfig to access the cluster:

```
aws eks update-kubeconfig \
  --region eu-north-1 \
  --name notes-app-cluster
```

Verify:

```
kubectl get nodes
```

You should see all nodes in **Ready** state.

---

##  Install NGINX Ingress Controller (kubectl apply method)

Install NGINX Ingress Controller using the AWS-specific manifest:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.4/deploy/static/provider/aws/deploy.yaml
```

Verify:
```
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```
You should see a Service of type LoadBalancer with an external DNS.

---

##  Metrics Server (Already Installed Automatically)

For newer EKS versions (v1.27+), Metrics Server is installed automatically.

Verify:
```
kubectl top nodes
kubectl top pods -A
```

If metrics are visible, no manual installation is required.

---

##  Enable OIDC Provider (Required for AWS Add-ons)

Enable IAM OIDC provider for the cluster:

```
eksctl utils associate-iam-oidc-provider \
  --cluster notes-app-cluster \
  --region eu-north-1 \
  --approve
```

This is required for:

- EBS CSI Driver

- Other AWS-managed addons

---

##  Install Required EKS Add-ons
a) EBS CSI Driver (Required for MySQL / PVCs)

```
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster notes-app-cluster \
  --region eu-north-1
```

Verify:
```
eksctl get addon \
  --cluster notes-app-cluster \
  --region eu-north-1
```

Status should be:
```
aws-ebs-csi-driver   ACTIVE
```

## b) Verify StorageClasses

```
kubectl get storageclass
```

If **gp3** does not exist, create it:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  fsType: ext4
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
allowVolumeExpansion: true
```

Apply:
```
kubectl apply -f gp3-storage-class.yaml
```

---

##  Cluster is Ready for Application Deployment

At this point, the cluster supports:

- Ingress (public access)

- Metrics (HPA)

- Persistent storage (MySQL)

- Managed scaling

You can now deploy:

- MySQL StatefulSet

- Backend services

- Frontend

- Ingress resources

- HPA

## Delete the EKS Cluster (Clean Up)

To delete the cluster and all associated resources:

```
terraform destroy -auto-approve
```

⏱️ Takes ~10–15 minutes.

Verify:

```
aws eks list-clusters --region eu-north-1
```
or you can use this
```
eksctl get cluster --region eu-north-1
```

---

## Remove Cluster from kubeconfig

List contexts:

```
kubectl config get-contexts
```

Delete the EKS context:
```
kubectl config delete-context arn:aws:eks:eu-north-1:<ACCOUNT_ID>:cluster/notes-app-cluster
```

(Optional) Remove cluster entry:
```
kubectl config delete-cluster arn:aws:eks:eu-north-1:<ACCOUNT_ID>:cluster/notes-app-cluster
```

(Optional) Remove user entry:
```
kubectl config unset users.arn:aws:eks:eu-north-1:<ACCOUNT_ID>:cluster/notes-app-cluster
```