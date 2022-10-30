# Add IAM users or roles to your Amazon EKS cluster

1. Determine which credentials kubectl is using to access your cluster
cat ~/.kube/config

2. You can see which other roles or users currently have access to your cluster with the following command:
kubectl describe -n kube-system configmap/aws-auth

3. Make sure that you have existing Kubernetes roles and rolebindings or clusterroles and clusterrolebindings that you can map IAM users or roles to.
View your existing Kubernetes roles or clusterroles. Roles are scoped to a namespace, but clusterroles are scoped to the cluster.

  kubectl get roles -A
  kubectl get clusterroles
  kubectl describe role role-name -n kube-system
  kubectl describe clusterrole cluster-role-name
  kubectl get rolebindings -A
  kubectl get clusterrolebindings
  kubectl describe rolebinding role-binding-name -n kube-system
  kubectl describe clusterrolebinding cluster-role-binding-name


4. Edit the aws-auth ConfigMap. You can use a tool such as eksctl to update the ConfigMap
  View the current mappings in the ConfigMap
  eksctl get iamidentitymapping --cluster my-cluster --region=region-code

  Add a mapping for a role
  eksctl create iamidentitymapping \
    --cluster my-cluster \
    --region=region-code \
    --arn arn:aws:iam::111122223333:role/my-role \
    --group eks-console-dashboard-full-access-group \
    --no-duplicate-arns

  Add a mapping for a user
    eksctl create iamidentitymapping \
    --cluster my-cluster \
    --region=region-code \
    --arn arn:aws:iam::111122223333:user/my-user \
    --group eks-console-dashboard-restricted-access-group \
    --no-duplicate-arns

  View the mappings in the ConfigMap again
    eksctl get iamidentitymapping --cluster my-cluster --region=region-code

# Apply the aws-authConfigMap to your cluster

  The aws-auth ConfigMap is automatically created and applied to your cluster when you create a managed node group or when you create a node group using eksctl. It is initially created to allow nodes to join your cluster, but you also use this ConfigMap to add role-based access control (RBAC) access to IAM users and roles.

1. Check to see if you have already applied the aws-auth ConfigMap.
  kubectl describe configmap -n kube-system aws-auth

2. Download, edit, and apply the AWS authenticator configuration map
  curl -o aws-auth-cm.yaml https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/aws-auth-cm.yaml

3. Open the file with a text editor. Replace <ARN of instance role (not instance profile)> with the Amazon Resource Name (ARN) of the IAM role associated with your nodes, and save the file

4. Apply the configuration. This command may take a few minutes to finish.
  kubectl apply -f aws-auth-cm.yaml


