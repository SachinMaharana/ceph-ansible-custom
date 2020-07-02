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

```
ceph auth list

ceph auth get-or-create client.logsuser
```

## get details for config and secret from following command

```
ceph mon dump
ceph auth list
ceph auth get-or-create client.logsuser
```

# kuberentes setup(assuming namespace csi)

kubectl apply -f ceph-csi/ceph-csi-config.yaml

kubectl apply -f ceph-csi/csi-rbd-secret.yam

kubectl create -f ceph-csi/csi-nodeplugin-rbac.yaml

kubectl create -f ceph-csi/csi-provisioner-rbac.yaml

kubectl create -f ceph-csi/ceph-csi-encryption-kms-config.yaml

kubectl create -f ceph-csi/csi-rbdplugin-provisioner.yaml

kubectl create -f ceph-csi/csi-rbdplugin.yaml

---

# testing

kubectl create -f ceph-csi/storageclass.yaml

kubectl create -f ceph-csi/raw-block-pvc.yaml

kubectl create -f ceph-csi/raw-block-pod.yaml

kubectl create -f ceph-csi/pvc.yaml

kubectl create -f ceph-csi/pod.yaml
