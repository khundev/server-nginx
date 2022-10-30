# Amazon EFS CSI driver
The Amazon EFS Container Storage Interface (CSI) driver provides a CSI interface that allows Kubernetes clusters running on AWS to manage the lifecycle of Amazon EFS file systems.

For detailed descriptions of the available parameters and complete examples that demonstrate the driver's features, see the Amazon EFS Container Storage Interface (CSI) driver project on GitHub.

Considerations

The Amazon EFS CSI Driver isn't compatible with Windows-based container images.

You can't use dynamic persistent volume provisioning with Fargate nodes, but you can use static provisioning.

Dynamic provisioning requires 1.2 or later of the driver. You can statically provision persistent volumes using version 1.1 of the driver on any supported Amazon EKS cluster version.

# Create an IAM policy and role
Create an IAM policy and assign it to an IAM role. The policy will allow the Amazon EFS driver to interact with your file system.

Create an IAM policy that allows the CSI driver's service account to make calls to AWS APIs on your behalf.

curl -o iam-policy-example.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/docs/iam-policy-example.json
aws iam create-policy \
    --policy-name AmazonEKS_EFS_CSI_Driver_Policy \
    --policy-document file://iam-policy-example.json

Create an IAM role and attach the IAM policy to it. Annotate the Kubernetes service account with the IAM role ARN and the IAM role with the Kubernetes service account name. You can create the role using eksctl or the AWS CLI

eksctl create iamserviceaccount \
    --cluster my-cluster \
    --namespace kube-system \
    --name efs-csi-controller-sa \
    --attach-policy-arn arn:aws:iam::111122223333:policy/AmazonEKS_EFS_CSI_Driver_Policy \
    --approve \
    --region region-code

# Install the Amazon EFS driver

helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/

# Create an Amazon EFS file system
vpc_id=$(aws eks describe-cluster \
    --name my-cluster \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

vpc_id=$(aws eks describe-cluster \
    --name my-cluster \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

vpc_id=$(aws eks describe-cluster \
    --name my-cluster \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

vpc_id=$(aws eks describe-cluster \
    --name my-cluster \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

file_system_id=$(aws efs create-file-system \
    --region region-code \
    --performance-mode generalPurpose \
    --query 'FileSystemId' \
    --output text)

kubectl get nodes

aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$vpc_id" \
    --query 'Subnets[*].{SubnetId: SubnetId,AvailabilityZone: AvailabilityZone,CidrBlock: CidrBlock}' \
    --output table

aws efs create-mount-target \
    --file-system-id $file_system_id \
    --subnet-id subnet-EXAMPLEe2ba886490 \
    --security-groups $security_group_id

# Deploy a sample application
# dynamic

aws efs describe-file-systems --query "FileSystems[*].FileSystemId" --output text

curl -o storageclass.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/examples/kubernetes/dynamic_provisioning/specs/storageclass.yaml

Edit the file. Find the following line, and replace the value for fileSystemId with your file system ID. Then deploy the storage class

kubectl apply -f storageclass.yaml

curl -o pod.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/examples/kubernetes/dynamic_provisioning/specs/pod.yaml

kubectl apply -f pod.yaml

kubectl get pods -n kube-system | grep efs-csi-controller

kubectl logs efs-csi-controller-74ccf9f566-q5989 \
    -n kube-system \
    -c csi-provisioner \
    --tail 10

kubectl get pv

kubectl get pvc

kubectl get pods -o wide

kubectl exec efs-app -- bash -c "cat data/out"







