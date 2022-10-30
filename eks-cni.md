# Amazon EKS VPC and subnet requirements and considerations

When you create a cluster, you specify a VPC and at least two subnets that are in different Availability Zones. This topic provides an overview of Amazon EKS specific requirements and considerations for the VPC and subnets that you use with your cluster

# VPC requirements and considerations

The VPC must have a sufficient number of IP addresses available for the cluster, any nodes, and other Kubernetes resources that you want to create. If the VPC that you want to use doesn't have a sufficient number of IP addresses, try to increase the number of available IP addresses. You can do this by associating additional Classless Inter-Domain Routing (CIDR) blocks with your VPC. You can associate private (RFC 1918) and public (non-RFC 1918) CIDR blocks to your VPC either before or after you create your cluster. It can take a cluster up to five hours for a CIDR block that you associated with a VPC to be recognized.

You can conserve IP address utilization by using a transit gateway with a shared services VPC

The VPC must have DNS hostname and DNS resolution support. Otherwise, nodes can't register to your cluster

The VPC might require VPC endpoints using AWS PrivateLink

If you created a cluster with Kubernetes 1.14 or earlier, Amazon EKS added the following tag to your VPC:
kubernetes.io/cluster/my-cluster    owned

# Subnet requirements and considerations
When you create a cluster, Amazon EKS creates 2–4 elastic network interfaces in the subnets that you specify. These network interfaces enable communication between your cluster and your VPC. These network interfaces also enable Kubernetes features such as kubectl exec and kubectl logs. Each Amazon EKS created network interface has the text Amazon EKS cluster-name in its description.

Amazon EKS can create its network interfaces in any subnet that you specify when you create a cluster. You can't change which subnets Amazon EKS creates its network interfaces in after your cluster is created. When you update the Kubernetes version of a cluster, Amazon EKS deletes the original network interfaces that it created, and creates new network interfaces. These network interfaces might be created in the same subnets as the original network interfaces or in different subnets than the original network interfaces. To control which subnets network interfaces are created in, you can limit the number of subnets you specify to only two when you create a cluster.

The subnets that you specify when you create a cluster must meet the following requirements:

The subnets must each have at least six IP addresses for use by Amazon EKS. However, we recommend at least 16 IP addresses.

The subnets must use IP address based naming. Amazon EC2 resource-based naming isn't supported with Amazon EKS.

The subnets can be a public or private. However, we recommend that you specify private subnets, if possible. A public subnet is a subnet with a route table that includes a route to an internet gateway, whereas a private subnet is a subnet with a route table that doesn't include a route to an internet gateway.

You can deploy nodes and Kubernetes resources to the same subnets that you specify when you create your cluster. However, this isn't necessary. This is because you can also deploy nodes and Kubernetes resources to subnets that you didn't specify when you created the cluster. If you deploy nodes to different subnets, Amazon EKS doesn't create cluster network interfaces in those subnets. Any subnet that you deploy nodes and Kubernetes resources to must meet the following requirements:

The subnets must have enough available IP addresses to deploy all of your nodes and Kubernetes resources to.

The subnets must use IP address-based naming. Amazon EC2 resource-based naming isn't supported with Amazon EKS.

If you need inbound access from the internet to your pods, make sure to have at least one public subnet with enough available IP addresses to deploy load balancers and ingresses to. You can deploy load balancers to public subnets. Load balancers can load balance to pods in private or public subnets. We recommend deploying your nodes to private subnets, if possible

If you plan to deploy nodes to a public subnet, the subnet must auto-assign IPv4 public addresses or IPv6 addresses. If you deploy nodes to a private subnet that has an associated IPv6 CIDR block, the private subnet must also auto-assign IPv6 addresses

If the subnet that you deploy a node to is a private subnet and its route table doesn't include a route to a network address translation (NAT) device (IPv4) or an egress-only gateway (IPv6), add VPC endpoints using AWS PrivateLink to your VPC. VPC endpoints are needed for all the AWS services that your nodes and pods need to communicate with. Examples include Amazon ECR, Elastic Load Balancing, Amazon CloudWatch, AWS Security Token Service, and Amazon Simple Storage Service (Amazon S3). The endpoint must include the subnet that the nodes are in. Not all AWS services support VPC endpoints

When a Kubernetes cluster that's version 1.18 and earlier was created, Amazon EKS added the following tag to all of the subnets that were specified.

kubernetes.io/cluster/my-cluster	shared

# Creating a VPC for your Amazon EKS cluster
You can use Amazon Virtual Private Cloud (Amazon VPC) to launch AWS resources into a virtual network that you've defined. This virtual network closely resembles a traditional network that you might operate in your own data center. However, it comes with the benefits of using the scalable infrastructure of Amazon Web Service

An Amazon EKS cluster, nodes, and Kubernetes resources are deployed to a VPC. If you want to use an existing VPC with Amazon EKS, that VPC must meet the requirements that are described in Amazon EKS VPC and subnet requirements and considerations.



