# Errors and Fixes

## a) Perma) Permission Denied on kubeconfigission Denied on kubeconfig

## ErrorError

```
unable to write kubeconfig: unable to read existing kubeconfig file "/etc/rancher/k3s/k3s.yaml": permission denied
```
## CauseCause

```
Your machine had k3s installed, which created a kubeconfig at /etc/rancher/k3s/k3s.yaml.
eksctl tried to merge the EKS kubeconfig into this file.
File owned by root → normal user could not read/write → permission denied.
```
## FixFix

```
# Use a user-owned kubeconfig file
mkdir -p ~/.kube
aws eks update-kubeconfig --region us-east-1 --name game-app --kubeconfig ~/.kube/config
```
```
# Export kubeconfig path
export KUBECONFIG=~/.kube/config
```
 Now kubectl commands target the EKS cluster safely without touching k3s config.

## b) Invalid exec plugin API versionb) Invalid exec plugin API version

## ErrorError

```
error: exec plugin: invalid apiVersion "client.authentication.k8s.io/v1alpha1"
```
## CauseCause

```
EKS kubeconfig used v1alpha1 for AWS IAM authentication.
Newer kubectl versions (≥1.25) dropped support for v1alpha1 → only accept v1beta1 or v1.
Happens when AWS CLI / AWS IAM Authenticator is outdated.
```
## FixesFixes

Option 1: Upgrade AWOption 1: Upgrade AWS CLI (recomS CLI (recommmended)ended)

```
sudo apt update
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update
```
```
# Update kubeconfig again
aws eks update-kubeconfig --region us-east-1 --name game-app --kubeconfig ~/.kube/config
```
```
Updated AWS CLI to v2 → kubeconfig now generates apiVersion: v1beta1.
```
Option 2: MOption 2: Manual patchanual patch

```
# Before
apiVersion: client.authentication.k8s.io/v1alpha
```
```
# After
apiVersion: client.authentication.k8s.io/v1beta
```

 Solved compatibility issue with new kubectl.

## c) ResourceNotFoundException (Cluster Not Found)c) ResourceNotFoundException (Cluster Not Found)

### ErrorError

```
unable to describe cluster control plane: operation error EKS: DescribeCluster,
StatusCode: 404 ... ResourceNotFoundException: No cluster found for name: game-app.
```
### CauseCause

```
eksctl tried to call DescribeCluster on game-app.
AWS responded 404 Not Found → cluster not found in your current AWS account + region.
```
### WWhy this happenshy this happens

```
1. WWrong AWrong AWS RegionS Region
If you created the cluster in us-east-1, but eksctl defaults to another region, it won’t find it.
eksctl only searches in one region at a time.
```
### FixFix

```
eksctl utils associate-iam-oidc-provider --cluster game-app --region us-east-1 --approve
```
## d) ALB Ingress Has No Addressd) ALB Ingress Has No Address

### ErrorError

```
kubectl get ingress -n game-
```
```
NAME CLASS HOSTS ADDRESS PORTS AGE
ingress-2048 alb * <none> 80 63m
```
When describing the ingress:

```
kubectl describe ingress ingress-2048 -n game-
```
The AWS Load Balancer Controller logs show:

```
OperationNotPermitted: This AWS account currently does not support creating load balancers.
Account restriction: Cannot create ALB
Failed deploy model due to operation error Elastic Load Balancing v2: CreateLoadBalancer
StatusCode: 400, OperationNotPermitted
```
### CauseCause

This happens in new AWS accounts with no billing history.
AWS restricts creating Application Load Balancers (ALB)Application Load Balancers (ALB) and Network Load Balancers (NLB)Network Load Balancers (NLB) until billing history is established.

### FixFix

```
Opened an AWS Support case regarding ALB creation restriction.
AWS confirmed the restriction is temporary and will be lifted automatically as billing usage builds up.
After some time, ALB creation started working and the ADDRESS field in kubectl get ingress showed the assigned ALB DNS name.
```
 Solved by waiting for AWSolved by waiting for AWS to lift new-account restrictions.S to lift new-account restrictions.


