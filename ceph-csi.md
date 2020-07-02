# Integration of ceph-csi of kubernetes with ceph

ceph-csi, which dynamically provisions RBD images to back Kubernetes volumes and maps these RBD images as block devices (optionally mounting a file system contained within the image) on worker nodes running pods that reference an RBD-backed volume.

# ceph setup

`sudo ceph -s`

`sudo ceph osd pool create logs 64 64`

`sudo ceph osd pool get logs all`

`sudo rbd pool init logs`

## create a logsuser for pool named logs

`ceph auth get-or-create client.logsuser mon 'profile rbd' osd 'profile rbd pool=logs' mgr 'profile rbd pool=logs'`

or

`ceph --cluster ceph auth get-or-create client.logsuser mon 'allow r' osd 'allow rwx pool=logs'`

## get details for config and secret from following command

```
ceph mon dump
ceph auth list
ceph auth get-or-create client.logsuser
```

# kuberentes setup(assuming namespace csi)

kubectl apply -f ceph-csi/0-csi-config-map.yaml

kubectl apply -f ceph-csi/1-csi-rbd-secret.yam

kubectl create -f ceph-csi/2-csi-nodeplugin-rbac.yaml

kubectl create -f ceph-csi/3-csi-provisioner-rbac.yaml

kubectl create -f ceph-csi/4-ceph-csi-encryption-kms-config.yaml

kubectl create -f ceph-csi/5-csi-rbdplugin-provisioner.yaml

kubectl create -f ceph-csi/6-csi-rbdplugin.yaml

---

# testing

kubectl create -f ceph-csi/7-storageclass.yaml

kubectl create -f ceph-csi/8-raw-block-pvc.yaml

kubectl create -f ceph-csi/9-raw-block-pod.yaml

kubectl create -f ceph-csi/10-pvc.yaml

kubectl create -f ceph-csi/11-pod.yaml

kubectl create -f ceph-csi/12-ss.yaml

kubectl create -f ceph-csi/13a-service-cass.yaml

kubectl create -f ceph-csi/13b-ss-cass.yaml

kubectl exec -it cassandra-0 -- nodetool status
