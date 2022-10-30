# Vertical Pod Autoscaler

The Kubernetes Vertical Pod Autoscaler automatically adjusts the CPU and memory reservations for your pods to help "right size" your applications. This adjustment can improve cluster resource utilization and free up CPU and memory for other pods.

Prerequisites

You have an existing Amazon EKS cluster. If you don't, see Getting started with Amazon EKS.
You have the Kubernetes Metrics Server installed. For more information, see Installing the Kubernetes Metrics Server.
You are using a kubectl client that is configured to communicate with your Amazon EKS cluster.
OpenSSL 1.1.1 or later installed on your device.

# Deploy the Vertical Pod Autoscaler

git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/
./hack/vpa-down.sh
./hack/vpa-up.sh
kubectl get pods -n kube-system

# Test your Vertical Pod Autoscaler installation
kubectl apply -f examples/hamster.yaml
kubectl get pods -l app=hamster
kubectl describe pod hamster-c7d89d6db-rglf5
kubectl get --watch pods -l app=hamster
kubectl describe pod hamster-c7d89d6db-jxgfv
kubectl describe vpa/hamster-vpa
kubectl delete -f examples/hamster.yaml

# Horizontal Pod Autoscaler
The Kubernetes Horizontal Pod Autoscaler automatically scales the number of pods in a deployment, replication controller, or replica set based on that resource's CPU utilization. This can help your applications scale out to meet increased demand or scale in when resources are not needed, thus freeing up your nodes for other applications. When you set a target CPU utilization percentage, the Horizontal Pod Autoscaler scales your application in or out to try to meet that target.

The Horizontal Pod Autoscaler is a standard API resource in Kubernetes that simply requires that a metrics source (such as the Kubernetes metrics server) is installed on your Amazon EKS cluster to work. You do not need to deploy or install the Horizontal Pod Autoscaler on your cluster to begin scaling your applications.

Prerequisites

You have an existing Amazon EKS cluster.
You have the Kubernetes Metrics Server installed. 
You are using a kubectl client that is configured to communicate with your Amazon EKS cluster.

# Run a Horizontal Pod Autoscaler test application
Deploy a simple Apache web server application with the following command.
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml

Create a Horizontal Pod Autoscaler resource for the php-apache deployment.
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
kubectl get hpa

Create a load for the web server by running a container.
kubectl run -i \
    --tty load-generator \
    --rm --image=busybox \
    --restart=Never \
    -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

kubectl get hpa php-apache
kubectl get hpa
kubectl delete deployment.apps/php-apache service/php-apache horizontalpodautoscaler.autoscaling/php-apache

# Network load balancing on Amazon EKS
Network traffic is load balanced at L4 of the OSI model. To load balance application traffic at L7, you deploy a Kubernetes ingress, which provisions an AWS Application Load Balancer

An AWS Network Load Balancer can load balance network traffic to pods deployed to Amazon EC2 IP and instance targets or to AWS Fargate IP targets

Prerequisites

Before you can load balance network traffic using the AWS Load Balancer Controller, you must meet the following requirements.

1. Have an existing cluster

2. Have the AWS Load Balancer Controller deployed on your cluster

3. At least one subnet. If multiple tagged subnets are found in an Availability Zone, the controller chooses the first subnet whose subnet ID comes first lexicographically. The subnet must have at least eight available IP addresses

4. The configuration of your load balancer is controlled by annotations that are added to the manifest for your service. Service annotations are different when using the AWS Load Balancer Controller than they are when using the AWS cloud provider load balancer controller

5. When using the Amazon VPC CNI plugin for Kubernetes, the AWS Load Balancer Controller can load balance to Amazon EC2 IP or instance targets and Fargate IP targets. When using Alternate compatible CNI plugins, the controller can only load balance to instance target

6. If you want to add tags to the load balancer when or after it's created, add the following annotation in your service specification
service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags

7. You can assign Elastic IP addresses to the Network Load Balancer by adding the following annotation
service.beta.kubernetes.io/aws-load-balancer-eip-allocations: eipalloc-xxxxxxxxxxxxxxxxx,eipalloc-yyyyyyyyyyyyyyyyy

# Create a network load balancer
You can create a network load balancer with IP or instance targets.

1. ip as target type
service.beta.kubernetes.io/aws-load-balancer-type: "external"
service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"

2. instances as target type
service.beta.kubernetes.io/aws-load-balancer-type: "external"
service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "instance"

If you want to create a Network Load Balancer in a public subnet to load balance to Amazon EC2 nodes
service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"

# (Optional) Deploy a sample application
1. Prerequisites
At least one public or private subnet in your cluster VPC.
Have the AWS Load Balancer Controller deployed on your cluster

2. To deploy a sample application
kubectl create namespace nlb-sample-app

3. apiVersion: apps/v1
kind: Deployment
metadata:
  name: nlb-sample-app
  namespace: nlb-sample-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: public.ecr.aws/nginx/nginx:1.21
          ports:
            - name: tcp
              containerPort: 80

4. kubectl apply -f sample-deployment.yaml

5. apiVersion: v1
kind: Service
metadata:
  name: nlb-sample-service
  namespace: nlb-sample-app
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: external
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
    service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: LoadBalancer
  selector:
    app: nginx

6. kubectl apply -f sample-service.yaml

7. kubectl get svc nlb-sample-service -n nlb-sample-app

8. curl k8s-default-samplese-xxxxxxxxxx-xxxxxxxxxxxxxxxx.elb.region-code.amazonaws.com

9. kubectl delete namespace nlb-sample-app

# Application load balancing on Amazon EKS
When you create a Kubernetes ingress, an AWS Application Load Balancer (ALB) is provisioned that load balances application traffic.

ALBs can be used with pods that are deployed to nodes or to AWS Fargate. You can deploy an ALB to public or private subnets

Application traffic is balanced at L7 of the OSI model. To load balance network traffic at L4, you deploy a Kubernetes service of the LoadBalancer type. This type provisions an AWS Network Load Balancer

Prerequisites
1. Have an existing cluster

2. Have the AWS Load Balancer Controller deployed on your cluster

3. At least two subnets in different Availability Zones
    kubernetes.io/cluster/my-cluster shared or owned

4. If you're using the AWS Load Balancer Controller version 2.1.1 or earlier, subnets must be tagged in the format that follows
    kubernetes.io/cluster/my-cluster shared or owned

5. To share an application load balancer across multiple service resources using IngressGroups
    alb.ingress.kubernetes.io/group.name: my-group
    alb.ingress.kubernetes.io/group.order: '10'

6.  Deploy a sample application
At least one public or private subnet in your cluster VPC.
Have the AWS Load Balancer Controller deployed on your cluster

public
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/examples/2048/2048_full.yaml

private
curl -o 2048_full.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/examples/2048/2048_full.yaml

kubectl apply -f 2048_full.yaml

7. After a few minutes, verify that the ingress resource was created with the following command
kubectl get ingress/ingress-2048 -n game-2048

8. delete the manifests
kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/examples/2048/2048_full.yaml
kubectl delete -f 2048_full.yaml

