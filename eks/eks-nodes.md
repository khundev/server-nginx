# Managed node groups
Amazon EKS managed node groups automate the provisioning and lifecycle management of nodes (Amazon EC2 instances) for Amazon EKS Kubernetes clusters.

With Amazon EKS managed node groups, you don’t need to separately provision or register the Amazon EC2 instances that provide compute capacity to run your Kubernetes applications. 

You can create, automatically update, or terminate nodes for your cluster with a single operation. Node updates and terminations automatically drain nodes to ensure that your applications stay available.

Every managed node is provisioned as part of an Amazon EC2 Auto Scaling group that's managed for you by Amazon EKS. Every resource including the instances and Auto Scaling groups runs within your AWS account. Each node group runs across multiple Availability Zones that you define.

Nodes launched as part of a managed node group are automatically tagged for auto-discovery by the Kubernetes cluster autoscaler. You can use the node group to apply Kubernetes labels to nodes and update them at any time.

There are no additional costs to use Amazon EKS managed node groups, you only pay for the AWS resources you provision. These include Amazon EC2 instances, Amazon EBS volumes, Amazon EKS cluster hours, and any other AWS infrastructure. There are no minimum fees and no upfront commitments.

# Managed node groups concepts
Amazon EKS managed node groups create and manage Amazon EC2 instances for you.

Every managed node is provisioned as part of an Amazon EC2 Auto Scaling group that's managed for you by Amazon EKS. Moreover, every resource including Amazon EC2 instances and Auto Scaling groups run within your AWS account.

The Auto Scaling group of a managed node group spans every subnet that you specify when you create the group.

Amazon EKS tags managed node group resources so that they are configured to use the Kubernetes Cluster Autoscaler.

You can use a custom launch template for a greater level of flexibility and customization when deploying managed nodes. If you deploy using a launch template, you can also use a custom AMI

Amazon EKS follows the shared responsibility model for CVEs and security patches on managed node groups. When managed nodes run an Amazon EKS optimized AMI, Amazon EKS is responsible for building patched versions of the AMI when bugs or issues are reported. We can publish a fix. However, you're responsible for deploying these patched AMI versions to your managed node groups. When managed nodes run a custom AMI, you're responsible for building patched versions of the AMI when bugs or issues are reported and then deploying the AMI

Amazon EKS managed node groups can be launched in both public and private subnets. If you launch a managed node group in a public subnet on or after April 22, 2020, the subnet must have MapPublicIpOnLaunch set to true for the instances to successfully join a cluster

When using VPC endpoints in private subnets, you must create endpoints for com.amazonaws.region.ecr.api, com.amazonaws.region.ecr.dkr, and a gateway endpoint for Amazon S3

You can create multiple managed node groups within a single cluster. For example, you can create one node group with the standard Amazon EKS optimized Amazon Linux 2 AMI for some workloads and another with the GPU variant for workloads that require GPU support

If your managed node group encounters an Amazon EC2 instance status check failure, Amazon EKS returns an error message to help you to diagnose the issue

Amazon EKS adds Kubernetes labels to managed node group instances. These Amazon EKS provided labels are prefixed with eks.amazonaws.com

Amazon EKS automatically drains nodes using the Kubernetes API during terminations or updates. Updates respect the pod disruption budgets that you set for your pods

There are no additional costs to use Amazon EKS managed node groups. You only pay for the AWS resources that you provision.

If you want to encrypt Amazon EBS volumes for your nodes, you can deploy the nodes using a launch template. To deploy managed nodes with encrypted Amazon EBS volumes without using a launch template, encrypt all new Amazon EBS volumes created in your account

# Managed node group capacity types
1. On-Demand
2. Spot

# Creating a managed node group
Prerequisites
1. An existing Amazon EKS cluster
2. CNI plugin for Kubernetes

eksctl version
eksctl create nodegroup --help

With a launch template
The launch template must already exist and must meet the requirements specified in Launch template configuration basics.

We recommend blocking pod access to IMDS if the following conditions are true:

You plan to assign IAM roles to all of your Kubernetes service accounts so that pods only have the minimum permissions that they need.

No pods in the cluster require access to the Amazon EC2 instance metadata service (IMDS) for other reasons, such as retrieving the current AWS Region.

If you want to block pod access to IMDS, then specify the necessary settings in the launch template.

Copy the following contents to your device. Replace the example values and then run the modified command to create the eks-nodegroup.yaml file

cat >eks-nodegroup.yaml <<EOF
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: my-cluster
  region: region-code
managedNodeGroups:
- name: my-mng
  launchTemplate:
    id: lt-id
    version: "1"
EOF

eksctl create nodegroup --config-file eks-nodegroup.yaml

# Update a node group version
eksctl upgrade nodegroup --name=node-group-name --cluster=my-cluster

upgrade a node group to the same version as the control plane's Kubernetes version
eksctl upgrade nodegroup \
  --name=node-group-name \
  --cluster=my-cluster \
  --kubernetes-version=1.23

# Deleting a managed node group
When you delete a managed node group, Amazon EKS first sets the minimum, maximum, and desired size of your Auto Scaling group to zero. This then causes your node group to scale down. Before each instance is terminated, Amazon EKS sends a signal to drain the pods from that node and then waits a few minutes. If the pods haven't drained after a few minutes, Amazon EKS lets Auto Scaling continue the termination of the instance. After every instance is terminated, the Auto Scaling group is deleted.

eksctl delete nodegroup \
  --cluster my-cluster \
  --region region-code \
  --name my-mng
