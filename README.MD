Reference
```
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.5/deploy/installation/
```
```
https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/
```

1. IAM OIDC Provider. This is the integration between AWS IAM and then Kuberentes RBAC.
```
eksctl utils associate-iam-oidc-provider \
    --region ap-south-1 \
    --cluster eks-cluster \
    --approve
```
2. Download IAM Policy. EKS should create AWS ALB dynamically. So EKS should have the access of AWS EC2.

```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
3. Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```
4. Provide access to kubernetes cluster to access this IAM Policy.
```
eksctl create iamserviceaccount \
--cluster=eks-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::315069654700:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region ap-south-1 \
--approve
```
5. Enable AWS Helm repo to install Ingress controller resources.
```
helm repo add eks https://aws.github.io/eks-charts
```
6. Install AWS ALB controller.
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=eks-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```