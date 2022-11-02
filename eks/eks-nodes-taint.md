# Node taints on managed node groups

Amazon EKS supports configuring Kubernetes taints through managed node groups. Taints and tolerations work together to ensure that pods aren't scheduled onto inappropriate nodes.

One or more taints can be applied to a node. This marks that the node shouldn't accept any pods that don't tolerate the taints. Tolerations are applied to pods and allow, but don't require, the pods to schedule onto nodes with matching taints.

Kubernetes node taints can be applied to new and existing managed node groups using the AWS Management Console or through the Amazon EKS API.

aws eks create-nodegroup \
 --cli-input-json '
{
  "clusterName": "my-cluster",
  "nodegroupName": "node-taints-example",
  "subnets": [
     "subnet-1234567890abcdef0",
     "subnet-abcdef01234567890",
     "subnet-021345abcdef67890"
   ],
  "nodeRole": "arn:aws:iam::111122223333:role/AmazonEKSNodeRole",
  "taints": [
     {
         "key": "dedicated",
         "value": "gpuGroup",
         "effect": "NO_SCHEDULE"
     }
   ]
}'

Maximum of 50 taints are allowed for one node group.

Taints can be updated after you create the node group using the UpdateNodegroupConfig API.

