# Self-managed nodes

A cluster contains one or more Amazon EC2 nodes that pods are scheduled on. Amazon EKS nodes run in your AWS account and connect to the control plane of your cluster through the cluster API server endpoint. You deploy one or more nodes into a node group. A node group is one or more Amazon EC2 instances that are deployed in an Amazon EC2 Auto Scaling group.

All instances in a node group must have the following characteristics:
1. Be the same instance type
2. Be running the same Amazon Machine Image (AMI)
3. Use the same Amazon EKS node IAM role

A cluster can contain several node groups. If each node group meets the previous requirements, the cluster can contain node groups that contain different instance types and host operating systems. Each node group can contain several nodes.

Amazon EKS provides specialized Amazon Machine Images (AMI) that are called Amazon EKS optimized AMIs. The AMIs are configured to work with Amazon EKS and include Docker, kubelet , and the AWS IAM Authenticator. The AMIs also contain a specialized bootstrap script that allows it to discover and connect to your cluster's control plane automatically

If you restrict access to the public endpoint of your cluster using CIDR blocks, we recommend that you also enable private endpoint access. This is so that nodes can communicate with the cluster. Without the private endpoint enabled, the CIDR blocks that you specify for public access must include the egress sources from your VPC.

If you launch self-managed nodes manually, add the following tag to each node. For more information
kubernetes.io/cluster/my-cluster  owned

# Launching self-managed Amazon Linux nodes
This topic describes how you can launch Auto Scaling groups of Linux nodes that register with your Amazon EKS cluster. After the nodes join the cluster, you can deploy Kubernetes applications to them. You can also launch self-managed Amazon Linux 2 nodes with eksctl or the AWS Management Console

Prerequisites
1. An existing Amazon EKS cluster
2. The Amazon VPC CNI plugin

eksctl create nodegroup \
  --cluster my-cluster \
  --name al-nodes \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --ssh-access \
  --managed=false \
  --ssh-public-key my-key

eksctl create nodegroup --help

# Launching self-managed Bottlerocket nodes
This topic describes how to launch Auto Scaling groups of Bottlerocket nodes that register with your Amazon EKS cluster. Bottlerocket is a Linux-based open-source operating system from AWS that you can use for running containers on virtual machines or bare metal hosts. After the nodes join the cluster, you can deploy Kubernetes applications to them

This procedure only works for clusters that were created with eksctl.

cat >bottlerocket.yaml <<EOF
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: region-code
  version: '1.23'

iam:
  withOIDC: true

nodeGroups:
  - name: ng-bottlerocket
    instanceType: m5.large
    desiredCapacity: 3
    amiFamily: Bottlerocket
    ami: auto-ssm
    iam:
       attachPolicyARNs:
          - arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
          - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
          - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
          - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
    ssh:
        allow: true
        publicKeyName: my-ec2-keypair-name
EOF

eksctl create nodegroup --config-file=bottlerocket.yaml

kubectl edit -n kube-system daemonset kube-proxy

      containers:
      - command:
        - kube-proxy
        - --v=2
        - --config=/var/lib/kube-proxy-config/config
        - --conntrack-max-per-core=0
        - --conntrack-min=0

        