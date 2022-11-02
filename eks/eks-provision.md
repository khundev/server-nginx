# 1. EKS cluster using AWS CLI
aws eks create-cluster \
  --region region-code \
  --name my-cluster \
  --kubernetes-version 1.23 \
  --role-arn arn:aws:iam::111122223333:role/myAmazonEKSClusterRole \
  --resources-vpc-config subnetIds=subnet-ExampleID1,subnet-ExampleID2,securityGroupIds=sg-ExampleID1

# 2. describe your cluster 
aws eks describe-cluster \
    --region region-code \
    --name my-cluster \
    --query "cluster.status"

# 3. additional step to update new cluster's kube config in your workstation
aws eks update-kubeconfig --region region-code --name my-cluster

# 4. check cluster service 
kubectl get svc

# 1 .EKS cluster using AWS eksctl
eksctl create cluster --name my-cluster --region region-code --version 1.23 --vpc-private-subnets subnet-ExampleID1,subnet-ExampleID2 --without-nodegroup