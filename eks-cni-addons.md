# Installing the AWS Load Balancer Controller add-on

The AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster. The controller provisions the following resources:

An AWS Application Load Balancer (ALB) when you create a Kubernetes Ingress.

An AWS Network Load Balancer (NLB) when you create a Kubernetes service of type LoadBalancer. In the past, the Kubernetes network load balancer was used for instance targets, but the AWS Load balancer Controller was used for IP targets. With the AWS Load Balancer Controller version 2.3.0 or later, you can create NLBs using either target type

Prerequisites
1. An existing Amazon EKS cluster
2. An existing AWS Identity and Access Management (IAM) OpenID Connect (OIDC) provider for your cluster
3. make sure that your Amazon VPC CNI plugin for Kubernetes, kube-proxy, and CoreDNS add-ons are at the minimum versions listed in Service account tokens.
4. Familiarity with AWS Elastic Load Balancing
5. Familiarity with Kubernetes service and ingress resources

# To deploy the AWS Load Balancer Controller to an Amazon EKS cluster

1. Create an IAM policy.
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/install/iam_policy.json

2. Create an IAM policy using the policy downloaded in the previous step
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

3. Create an IAM role
eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name "AmazonEKSLoadBalancerControllerRole" \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

4. If you don't currently have the AWS ALB Ingress Controller for Kubernetes installed, or don't currently have the 0.1.x version of the AWS Load Balancer Controller installed with Helm
helm delete aws-alb-ingress-controller -n kube-system
helm delete aws-load-balancer-controller -n kube-system

5. Install the AWS Load Balancer Controller using Helm V3 or later or by applying a Kubernetes manifest.
helm repo add eks https://aws.github.io/eks-charts
helm repo update

6. If your nodes don't have access to Amazon EKS Amazon ECR image repositories, then you need to pull the following container image and push it to a repository that your nodes have access to
602401143452.dkr.ecr.region-code.amazonaws.com/amazon/aws-load-balancer-controller:v2.4.4

7. Install the AWS Load Balancer Controller. If you're deploying the controller to Amazon EC2 nodes that have restricted access to the Amazon EC2 instance metadata service (IMDS), or if you're deploying to Fargate, then add the following flags to the helm command that follows:
--set region=region-code
--set vpcId=vpc-xxxxxxxx

If you're deploying to any AWS Region other than us-west-2, then add the following flag to the helm command,
--set image.repository=602401143452.dkr.ecr.region-code.amazonaws.com/amazon/aws-load-balancer-controller

8. Replace my-cluster with your own.
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 

9. kubectl get deployment -n kube-system aws-load-balancer-controller

# Managing the CoreDNS add-on

CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS. When you launch an Amazon EKS cluster with at least one node, two replicas of the CoreDNS image are deployed by default, regardless of the number of nodes deployed in your cluster. The CoreDNS pods provide name resolution for all pods in the cluster. The CoreDNS pods can be deployed to Fargate nodes if your cluster includes an AWS Fargate profile with a namespace that matches the namespace for the CoreDNS deployment

Prerequisites
1. An existing Amazon EKS cluster
2.  Amazon VPC CNI plugin for Kubernetes and kube-proxy add-ons are at the minimum versions listed in Service account tokens.

1. Adding the CoreDNS Amazon EKS add-on
eksctl create addon --name coredns --cluster my-cluster --force

2. Updating the CoreDNS Amazon EKS add-on
eksctl get addon --name coredns --cluster my-cluster
eksctl update addon --name coredns --version v1.8.7-eksbuild.3 --cluster my-cluster --force

3. Removing the CoreDNS Amazon EKS add-on
eksctl delete addon --cluster my-cluster --name coredns --preserve

4. Updating the CoreDNS self-managed add-on
kubectl describe deployment coredns --namespace kube-system | grep Image | cut -d "/" -f 3

5. Retrieve your current CoreDNS image:
kubectl get deployment coredns --namespace kube-system \
    -o=jsonpath='{$.spec.template.spec.containers[:1].image}'

6. If you're updating to CoreDNS 1.8.3 or later, then you need to add the endpointslices permission to the system:coredns Kubernetes clusterrole
kubectl edit clusterrole system:coredns -n kube-system

7. Update the CoreDNS add-on by replacing 602401143452 and region-code with the values from the output returned in a previous step.Replace 1.8.7-eksbuild.2 with your cluster's recommended CoreDNS version or later
kubectl set image --namespace kube-system deployment.apps/coredns \
    coredns=602401143452.dkr.ecr.region-code.amazonaws.com/eks/coredns:v1.8.7-eksbuild.2

# Managing the kube-proxy add-on

Kube-proxy maintains network rules on each Amazon EC2 node. It enables network communication to your pods. Kube-proxy is not deployed to Fargate nodes

There are two types of the kube-proxy container image available for each Kubernetes version:

Default – This type is based on a Debian-based Docker image that is maintained by the Kubernetes upstream community.

Minimal – This type is based on a minimal base image maintained by Amazon EKS Distro, which contains minimal packages and doesn't have shells. 

kubectl get daemonset kube-proxy --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}' | cut -d : -f 2

Prerequisites
1. An existing Amazon EKS cluster.
2. make sure that your Amazon VPC and CoreDNS add-ons are at the minimum versions listed in Service account tokens.

1. Adding the kube-proxy Amazon EKS add-on
eksctl create addon --name kube-proxy --cluster my-cluster --force

2. Updating the kube-proxy Amazon EKS add-on
eksctl get addon --name kube-proxy --cluster my-cluster
eksctl update addon --name kube-proxy --version v1.23.8-eksbuild.2 --cluster my-cluster --force

3. Removing the kube-proxy Amazon EKS add-on
eksctl delete addon --cluster my-cluster --name kube-proxy --preserve

4. Updating the kube-proxy self-managed add-on
kubectl get daemonset kube-proxy --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}'
kubectl set image daemonset.apps/kube-proxy -n kube-system kube-proxy=602401143452.dkr.ecr.region-code.amazonaws.com/eks/kube-proxy:v1.23.8-eksbuild.2
kubectl edit -n kube-system daemonset/kube-proxy
kubectl edit -n kube-system daemonset/kube-proxy

# Installing the Calico network policy engine add-on

1. To install Calico with Helm
helm repo add projectcalico https://docs.projectcalico.org/charts
helm repo update                         
helm install calico projectcalico/tigera-operator --version v3.21.4
kubectl get all -n tigera-operator
kubectl get all -n calico-system

# Stars policy demo

1. To run the Stars policy demo
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/00-namespace.yaml
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/01-management-ui.yaml
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/02-backend.yaml
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/03-frontend.yaml
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/04-client.yaml

2. View all pods on the cluster.
kubectl get pods -A

3. kubectl port-forward service/management-ui -n management-ui 9001

4. Apply the following network policies to isolate the services from each other:
kubectl apply -n stars -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/default-deny.yaml
kubectl apply -n client -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/default-deny.yaml

5. Apply the following network policies to allow the management user interface to access the services:
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/allow-ui.yaml
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/allow-ui-client.yaml

6. Apply the following network policy to allow traffic from the front-end service to the back-end service:
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/backend-policy.yaml

7. Apply the following network policy to allow traffic from the client to the front-end service.
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/frontend-policy.yaml

8. delete all manifests
kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/04-client.yaml
kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/03-frontend.yaml
kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/02-backend.yaml
kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/01-management-ui.yaml
kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/00-namespace.yaml

