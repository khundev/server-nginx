# Cluster Autoscaler
The following terms are used throughout this topic:

1. Kubernetes Cluster Autoscaler
  A core component of the Kubernetes control plane that makes scheduling and scaling decisions

2. AWS Cloud provider implementation
  An extension of the Kubernetes Cluster Autoscaler that implements the decisions of the Kubernetes Cluster Autoscaler by communicating with AWS products and services such as Amazon EC2.

3. Node groups
  Nodes that are found within a single node group might share several common properties such as labels and taints. However, they can still consist of more than one Availability Zone or instance type

4. Amazon EC2 Auto Scaling groups
  A feature of AWS that's used by the Cluster Autoscaler. Auto Scaling groups are suitable for a large number of use cases. Amazon EC2 Auto Scaling groups are configured to launch instances that automatically join their Kubernetes cluster

# Prerequisites
1. eks cluster
2. existing IAM/OIDC provider
3. Node groups with Auto Scaling groups tags. The Cluster Autoscaler requires the following tags on your Auto Scaling groups so that they can be auto-discovered
  In eksctl, these tags are automatically applied.
  If not using eksctl, must create the following tags for your node groups
    k8s.io/cluster-autoscaler/my-cluster  owned
    k8s.io/cluster-autoscaler/enabled true

# Create an IAM policy and role
Create an IAM policy that grants the permissions that the Cluster Autoscaler requires to use an IAM role

1. Save the following contents to a file that's named
cluster-autoscaler-policy.json
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/k8s.io/cluster-autoscaler/my-cluster": "owned"
                }
            }
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeAutoScalingGroups",
                "ec2:DescribeLaunchTemplateVersions",
                "autoscaling:DescribeTags",
                "autoscaling:DescribeLaunchConfigurations"
            ],
            "Resource": "*"
        }
    ]
}

No need to create this iam role if you create the cluster using eksctl
you can skip to step 2
Create the policy with the following command
  aws iam create-policy \
  --policy-name AmazonEKSClusterAutoscalerPolicy \
  --policy-document file://cluster-autoscaler-policy.json

2. You can create an IAM role and attach an IAM policy to it using eksctl
Run the following command if you created your Amazon EKS cluster with eksctl. If you created your node groups using the --asg-access option, then replace AmazonEKSClusterAutoscalerPolicy with the name of the IAM policy that eksctl created for you
eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=cluster-autoscaler \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AmazonEKSClusterAutoscalerPolicy \
  --override-existing-serviceaccounts \
  --approve

We recommend that, if you created your node groups using the --asg-access option, you detach the IAM policy that eksctl created and attached to the Amazon EKS node IAM role that eksctl created for your node groups.

# Deploy the Cluster Autoscaler

1. Download the Cluster Autoscaler YAML file.
curl -o cluster-autoscaler-autodiscover.yaml https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

2. Modify the YAML file and replace <YOUR CLUSTER NAME> with your cluster name. Also consider replacing the cpu and memory values as determined by your environment.

3. Apply the YAML file to your cluster
kubectl apply -f cluster-autoscaler-autodiscover.yaml

4. Annotate the cluster-autoscaler service account with the ARN of the IAM role that you created previously
kubectl annotate serviceaccount cluster-autoscaler \
  -n kube-system \
  eks.amazonaws.com/role-arn=arn:aws:iam::ACCOUNT_ID:role/AmazonEKSClusterAutoscalerRole

5. Patch the deployment to add the cluster-autoscaler.kubernetes.io/safe-to-evict annotation to the Cluster Autoscaler pods with the following command.
kubectl patch deployment cluster-autoscaler \
  -n kube-system \
  -p '{"spec":{"template":{"metadata":{"annotations":{"cluster-autoscaler.kubernetes.io/safe-to-evict": "false"}}}}}'

6. Edit the Cluster Autoscaler deployment with the following command.
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
replace with the following codes
   --balance-similar-node-groups
  --skip-nodes-with-system-pods=false

7. Set the Cluster Autoscaler image tag to the version that you recorded in the previous step with the following command.Replace 1.23.n with your own value.
kubectl set image deployment cluster-autoscaler \
  -n kube-system \
  cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v1.23.n

# View your Cluster Autoscaler logs
View your Cluster Autoscaler logs with the following command.
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler

# Karpenter
Amazon EKS supports the Karpenter open-source autoscaling project. See the Karpenter documentation to deploy it.

About Karpenter
Karpenter is a flexible, high-performance Kubernetes cluster autoscaler that helps improve application availability and cluster efficiency. Karpenter launches right-sized compute resources, (for example, Amazon EC2 instances), in response to changing application load in under a minute. Through integrating Kubernetes with AWS, Karpenter can provision just-in-time compute resources that precisely meet the requirements of your workload. Karpenter automatically provisions new compute resources based on the specific requirements of cluster workloads. These include compute, storage, acceleration, and scheduling requirements. Amazon EKS supports clusters using Karpenter, although Karpenter works with any conformant Kubernetes cluster.

How Karpenter works
Karpenter works in tandem with the Kubernetes scheduler by observing incoming pods over the lifetime of the cluster. It launches or terminates nodes to maximize application availability and cluster utilization. When there is enough capacity in the cluster, the Kubernetes scheduler will place incoming pods as usual. When pods are launched that cannot be scheduled using the existing capacity of the cluster, Karpenter bypasses the Kubernetes scheduler and works directly with your provider’s compute service, (for example, Amazon EC2), to launch the minimal compute resources needed to fit those pods and binds the pods to the nodes provisioned. As pods are removed or rescheduled to other nodes, Karpenter looks for opportunities to terminate under-utilized nodes. Running fewer, larger nodes in your cluster reduces overhead from daemonsets and Kubernetes system components and provides more opportunities for efficient bin-packing.

Prerequisites
Before deploying Karpenter, you must meet the following prerequisites:

An existing Amazon EKS cluster 

An existing IAM OIDC provider for your cluster. 

A user or role with permission to create a cluster.

AWS CLI

Installing or updating kubectl

Using Helm with Amazon EKS

You can deploy Karpenter using eksctl if you prefer