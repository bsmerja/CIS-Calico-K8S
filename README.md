# CIS-static-route integration in K8S - Calico-k8s implementation:

## BIGIP CIS Integration in K8S environment with static route, IPAM and CRD in OpenShift OVN-Kubernetes

### Prerequisite:

1. Install latest AS3 extention on BIG-IP.

  a) Download latest AS3 extention:
  ```
  https://github.com/F5Networks/f5-appsvcs-extension/releases
  ```
  b) Upload package at:  
  BIG-IP iApps  ››  Package Management LX, import and upload downloded file
  
  c) Check installation is Suucessfull
  ```
  https://(IP address of BIG-IP)/mgmt/shared/appsvcs/info
  ```

  Results should be like:
  
  {"version":"3.51.0","release":"5","schemaCurrent":"3.51.0","schemaMinimum":"3.0.0"}

2. Create partition "kubernetes"
3. Create a directory in /var/tmp/ on one of nodes of cluster where IPAM can create DB for saving persistent data
```
mkdir -p /var/tmp/cis_ipam
chmod 776 -R /var/tmp/cis_ipam
```

### BIGIP CIS Installation manually in separate namespace 

1. Create separate name space cis-system
```
kubectl create ns cis-system
```
2. Get hostname on which /var/tmp/cis_ipam directory is created, update "ipam-pv-and-pvc.yaml" and create Persistent Volume and Persistent Volume claim for ipam
```
kubectl get nodes -o yaml | grep "kubernetes.io/hostname" # to find hostname

kubectl create -f ipam-pv-and-pvc.yaml
```
3. Create Service Account for CIS and IPAM in cis-system name space:
```
kubectl create -f cis-service-account.yaml
kubectl create -f ipam-service-account.yaml
```
4. Create clusterole for CIS and IPAM:
```
kubectl create -f cis-clusterrole.yaml
kubectl create -f ipam-clusterrole.yaml
```
5. Create secret for BIG-IP admin username and apssword:
```
kubectl create secret generic bigip-login -n kube-system --from-literal=username=<bigip-admin-username> --from-literal=password=<bigip-admin-password>
```
6. Create clusterrolebinding for CIS and IPAM
```
kubectl create -f cis-clusterrolebinding.yaml
kubectl create -f ipam-clusterrolebinding.yaml
```
7. Create crds:
```
kubectl create -f cis-customresourcedefinitions.yaml
```
8. In case cluster does not have internet access, please download CIS and IPAM image and push in your private repo
```
docker pull f5networks/k8s-bigip-ctlr:latest
docker pull f5networks/f5-ipam-controller:latest
```
9. Update following files with respective private image
```
vim cis-staticroute-deployemnt-with-ipam.yaml
vim ipam-controller-deployment.yaml
```
10. Deploy CIS and ipam
```
kubectl create -f cis-staticroute-deployemnt-with-ipam.yaml
kubectl create -f ipam-controller-deployment.yaml
```
### BIGIP static route should be updated automatically, based Calico-K8s

<img width="1430" alt="image" src="https://github.com/bsmerja/ocp-cis-static-route/assets/49276353/e738da4c-6f6b-4cbc-aeab-6a34db564292">

### Deploy sample Application to and VirtualServer configuration

1. Create sample application deployment:
```
kubectl create -f sample-app.yaml
```
3. Create service type ClusterIP:
```
kubectl create -f sample-clusterip-service.yaml
```
5. Create VirtualServer:
```
kubectl create -f sample-app-vs.yaml
```

### Check BIG-IP configuration crd VS and pool created with default pool assigned to newly created VS:

VS Config:
<img width="1620" alt="image" src="https://github.com/bsmerja/CIS-Calico-K8S/assets/49276353/33f0d651-3574-4c02-9985-12717850dbc1">

Pool Config:

<img width="1604" alt="image" src="https://github.com/bsmerja/CIS-Calico-K8S/assets/49276353/4257e627-3364-44d8-8861-3e5b8d354f52">

Resource assign:

<img width="1183" alt="image" src="https://github.com/bsmerja/ocp-cis-static-route/assets/49276353/8df2dd9b-b916-44ab-b840-dec3b0e29eb0">
