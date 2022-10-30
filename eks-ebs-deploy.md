# Deploy a sample application and verify that the CSI driver is working
You can test the CSI driver functionality with a sample application

1. Deploy a sample application that uses the external snapshotter to create volume snapshots
2. Deploy a sample application that uses volume resizing

This procedure uses the Dynamic volume provisioning example from the Amazon EBS Container Storage Interface (CSI) driver GitHub repository to consume a dynamically provisioned Amazon EBS volume.

git clone https://github.com/kubernetes-sigs/aws-ebs-csi-driver.git
cd aws-ebs-csi-driver/examples/kubernetes/dynamic-provisioning/
kubectl apply -f manifests/
kubectl describe storageclass ebs-sc
kubectl get pods --watch
kubectl get pv
kubectl describe pv pvc-37717cd6-d0dc-11e9-b17f-06fad4858a5a
kubectl exec -it app -- cat /data/out.txt
kubectl delete -f manifests/
