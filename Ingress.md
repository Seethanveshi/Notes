# 🚀 Kubernetes Ingress Setup on EKS (Focused Guide)

This guide covers:
- AWS Load Balancer Controller installation
- Ingress creation (Path-based + Host-based)
- Verification & testing

---

# 🔧 Step 1: Install AWS Load Balancer Controller

## 1. Associate IAM OIDC provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster <cluster-name> \
  --approve
```

## 2. Download IAM Policy
```bash
- curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

## 3. Create IAM Policy
```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json
```

## 4. Create IAM Service Account
```bash
eksctl create iamserviceaccount \
  --cluster <cluster-name> \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region ap-south-1
```

## 5. Install Controller using Helm

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

## 6. Verify Controller

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system deployment/aws-load-balancer-controller
kubectl get ingressclass
```





## 🌐 Step 2: Ingress (Path-Based Routing)

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip

spec:
  ingressClassName: alb
  rules:

  - host: nginx.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80

  - host: httpd.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: httpd-service
            port:
              number: 80
```

## Apply & Verify
```bash
kubectl apply -f ingress.yaml
kubectl get ingress
kubectl describe ingress app-ingress
```
