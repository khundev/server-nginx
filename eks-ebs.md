# Storage classes

kubectl get storageclass

Create an AWS storage class manifest file for your storage class. The following gp2-storage-class.yaml example defines a storage class called gp2 that uses the Amazon EBS gp2 volume type.

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gp2
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4 

kubectl create -f gp2-storage-class.yaml
kubectl get storageclass

Choose a storage class and set it as your default by setting the storageclass.kubernetes.io/is-default-class=true annotation.
kubectl annotate storageclass gp2 storageclass.kubernetes.io/is-default-class=true

kubectl get storageclass

# Amazon EBS CSI driver

The Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver allows Amazon Elastic Kubernetes Service (Amazon EKS) clusters to manage the lifecycle of Amazon EBS volumes for persistent volumes.

Here are some things to consider about using the Amazon EBS CSI driver.

The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf. For more information, see Creating the Amazon EBS CSI driver IAM role for service accounts.

You can run the Amazon EBS CSI controller on Fargate, but you can't mount volumes to Fargate pods.

Alpha features of the Amazon EBS CSI driver aren't supported on Amazon EKS clusters.

The Amazon EBS CSI driver isn't installed when you first create a cluster. To use the driver, you must add it as an Amazon EKS add-on or as a self-managed add-on.

# Creating the Amazon EBS CSI driver IAM role for service accounts
The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf.

When the plugin is deployed, it creates and is configured to use a service account that's named ebs-csi-controller-sa. The service account is bound to a Kubernetes clusterrole that's assigned the required Kubernetes permissions.

Prerequisites
An existing cluster.

1.19 requires eks.7 or later.
1.20 requires eks.3 or later.
1.21 requires eks.3 or later.

To create your Amazon EBS CSI plugin IAM role with eksctl
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster my-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole

If you use a custom KMS key for encryption on your Amazon EBS volumes, customize the IAM role as needed
  {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": ["custom-key-id"],
      "Condition": {
        "Bool": {
          "kms:GrantIsForAWSResource": "true"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": ["custom-key-id"]
    }
  ]
}

Create the policy. You can change KMS_Key_For_Encryption_On_EBS_Policy to a different name
aws iam create-policy \
  --policy-name KMS_Key_For_Encryption_On_EBS_Policy \
  --policy-document file://kms-key-for-encryption-on-ebs.json

Attach the IAM policy to the role with the following command. Replace 111122223333 with your account ID
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::111122223333:policy/KMS_Key_For_Encryption_On_EBS_Policy \
  --role-name AmazonEKS_EBS_CSI_DriverRole

# Managing the Amazon EBS CSI driver as an Amazon EKS add-on
To improve security and reduce the amount of work, you can manage the Amazon EBS CSI driver as an Amazon EKS add-on

Prerequisites
1. existing cluster

Adding the Amazon EBS CSI add-on
aws eks describe-addon-versions --addon-name aws-ebs-csi-driver
eksctl create addon --name aws-ebs-csi-driver --cluster my-cluster --service-account-role-arn arn:aws:iam::111122223333:role/AmazonEKS_EBS_CSI_DriverRole --force

Updating the Amazon EBS CSI driver as an Amazon EKS add-on
To update the Amazon EBS CSI add-on using eksctl
Check the current version of your Amazon EBS CSI add-on. Replace my-cluster with your cluster name.
eksctl get addon --name aws-ebs-csi-driver --cluster my-cluster
eksctl update addon --name aws-ebs-csi-driver --version v1.11.4-eksbuild.1 --cluster my-cluster --force

Removing the Amazon EBS CSI add-on
eksctl delete addon --cluster my-cluster --name aws-ebs-csi-driver --preserve