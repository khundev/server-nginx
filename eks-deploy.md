# Deploy a sample application
Prerequisites
1. An existing Kubernetes cluster with at least one node
2. Kubectl installed on your computer
3. Kubectl configured to communicate with your cluster


kubectl create namespace eks-sample-app

apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-sample-linux-deployment
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: eks-sample-linux-app
  template:
    metadata:
      labels:
        app: eks-sample-linux-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
                - arm64
      containers:
      - name: nginx
        image: public.ecr.aws/nginx/nginx:1.21
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent
      nodeSelector:
        kubernetes.io/os: linux

kubectl apply -f eks-sample-deployment.yaml

apiVersion: v1
kind: Service
metadata:
  name: eks-sample-linux-service
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  selector:
    app: eks-sample-linux-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

kubectl apply -f eks-sample-service.yaml
kubectl get all -n eks-sample-app
kubectl -n eks-sample-app describe service eks-sample-linux-service
kubectl -n eks-sample-app describe pod eks-sample-linux-deployment-65b7669776-m6qxz
kubectl exec -it eks-sample-linux-deployment-65b7669776-m6qxz -n eks-sample-app -- /bin/bash
curl eks-sample-linux-service
cat /etc/resolv.conf
kubectl delete namespace eks-sample-app