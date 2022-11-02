# The Basics of kOps Commands

## if you're going to create cluster in AWS, you will need IAM user with proper roles
aws iam create-group --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess --group-name kops

aws iam create-user --user-name kops

aws iam add-user-to-group --user-name kops --group-name kops

aws iam create-access-key --user-name kops

## configure AWS credentials and export them into shell environment
aws configure           # Use your new access and secret key here
aws iam list-users      # you should see a list of all your IAM users here

# Because "aws configure" doesn't export these vars for kops to use, we export them now
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)

## create your subdomain
ID=$(uuidgen) && aws route53 create-hosted-zone --name subdomain.example.com --caller-reference $ID | \
    jq .DelegationSet.NameServers

## note your parent hosted zone id
aws route53 list-hosted-zones | jq '.HostedZones[] | select(.Name=="example.com.") | .Id'

## supply the subdomain ns records into the parent hosted zone
aws route53 change-resource-record-sets \
 --hosted-zone-id <parent-zone-id> \
 --change-batch file://subdomain.json

## test your dns setup 
dig ns subdomain.example.com


## kOps create
kOps create cluster <clustername>

## kOps update
kOps update cluster --name <clustername>

Run the command with the --yes flag to apply your changes to the cluster.

## kOps get
kOps get clusters

## kOps delete
kOps delete cluster --name <clustername>

## kOps rolling-update
kOps rolling-update cluster --name <clustername>

## kOps validate
kOps validate cluster --wait <specified_time>

# How to Set Up Kubernetes on AWS using kOps
## Install kubectl.
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

## Give the executable permission to the downloaded file and move it to /usr/local/bin/
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

## Create an S3 Bucket using the AWS CLI where kOps will save all the cluster's state information.
aws s3api create-bucket \
--bucket my-kOps-bucket-1132 \
--region us-west-2 \
--create-bucket-configuration LocationConstraint=us-west-2
--acl public-read

## List the aws s3 bucket to see the bucket you have created
aws s3 ls

## Enable the version for the s3 bucket
aws s3api put-bucket-versioning \
--bucket my-kOps-bucket-1132 \
--versioning-configuration Status=Enabled

## Generate ssh key for which will be used by kOps for cluster login and password generation
ssh-keygen

## Expose the cluster name and s3 bucket as environment variables
export kOps_CLUSTER_NAME=kOpsdemo1.k8s.local
export kOps_STATE_STORE=s3://my-kOps-bucket-1132

## first check your region's availability zones
aws ec2 describe-availability-zones --region us-west-2

## create the cluster
kOps create cluster \
--cloud=aws \
--zones=eu-central-1a \
--node-count=1 \
--node-size=t2.micro \
--master-size=t2.micro \
--name=${kOps_CLUSTER_NAME}

## check the cluster is created
kOps get cluster

## Apply the specified cluster specifications to the cluster.
kOps update cluster \
--name kOpsdemo1.k8s.local \
--yes \
--admin

## validate the cluster
kOps validate cluster --wait 10m

## check the cluster specifications
kubectl get nodes
kubectl top nodes

## delete the cluster
kOps delete cluster --name kOpsdemo1.k8s.local --yes


