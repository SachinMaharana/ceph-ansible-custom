kubectl create secret generic ceph-admin-secret --type="kubernetes.io/rbd" --from-literal=key='AQDzafte9KXLORAAtSPywJrEEZLF3nRqI+8pwg==' --namespace=kube-system

sudo ceph ceph osd pool create kube8 64

sudo ceph auth add client.kube8 mon 'allow r' osd 'allow rwx pool=kube8'

sudo ceph osd pool application enable kube8 rbd

sudo rbd pool init kube8

sudo ceph auth get-key client.kube8

kubectl create secret generic ceph-k8s-secret \
 --type="kubernetes.io/rbd" \
 --from-literal=key='AQDxbvtegv+DCBAAYlbnrFfivr9XlgaQQTT4lg==' \
 --namespace=kube-system

---

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
name: ceph-rbd
provisioner: ceph.com/rbd
parameters:
monitors: 172.31.17.69:6789, 172.31.18.216:6789, 172.31.25.28:6789
pool: kube8
adminId: admin
adminSecretNamespace: kube-system
adminSecretName: ceph-admin-secret
userId: kube8
userSecretNamespace: kube-system
userSecretName: ceph-k8s-secret
imageFormat: "2"
imageFeatures: layering

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
name: ceph-rbd-claim2
spec:
accessModes: - ReadWriteOnce
storageClassName: ceph-rbdd
resources:
requests:
storage: 1Gi

---

kind: Pod
apiVersion: v1
metadata:
name: rbd-test-pod
spec:
containers:

- name: rbd-test-pod
  image: busybox
  command: - "/bin/sh"
  args: - "-c" - "touch /mnt/RBD-SUCCESS && exit 0 || exit 1"
  volumeMounts: - name: pvc
  mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
  - name: pvc
    persistentVolumeClaim:
    claimName: ceph-rbd-claim1

apiVersion: v1
kind: Pod
metadata:
name: ceph-mysql
spec:
containers: - name: ceph-mysql
image: tutum/mysql
ports: - name: mysql-db
containerPort: 3306
volumeMounts: - name: mysql-pv
mountPath: /var/lib/mysql
volumes: - name: mysql-pv
persistentVolumeClaim:
claimName: ceph-rbd-claim2

## \$ cat <<EOF > csi-rbd-secret.yaml

apiVersion: v1
kind: Secret
metadata:
name: csi-rbd-secret
namespace: default
stringData:
userID: admin
userKey: QVFBNUx2eGVLdHV6T0JBQTVuRzJmUldqajhyWEVhL296WEhLREE9PQ==
EOF

## cat <<EOF > csi-rbd-sc.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
name: csi-rbd-sc
provisioner: rbd.csi.ceph.com
parameters:
clusterID: 24249032-c83e-4cb5-a25a-a620d5439a15
pool: kubernetes
csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
csi.storage.k8s.io/provisioner-secret-namespace: default
csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
csi.storage.k8s.io/node-stage-secret-namespace: default
reclaimPolicy: Delete
mountOptions:

- discard
  EOF

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: raw-block-pvc
spec:
accessModes:

- ReadWriteMany
  volumeMode: Block
  resources:
  requests:
  storage: 1Gi
  storageClassName: csi-rbd

## cat <<EOF > raw-block-pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: raw-block-pvc
spec:
accessModes: - ReadWriteOnce
volumeMode: Block
resources:
requests:
storage: 1Gi
storageClassName: csi-rbd-sc
EOF

[
{
"clusterID": "24249032-c83e-4cb5-a25a-a620d5439a15",
"monitors": [
"172.31.17.85:6789",
"172.31.19.133:6789",
"172.31.29.12:6789"
],
}
]

## cat <<EOF > csi-config-map.yaml

apiVersion: v1
kind: ConfigMap
data:
config.json: |-
[
{
"clusterID": "24249032-c83e-4cb5-a25a-a620d5439a15",
"monitors": [
"172.31.17.85:6789",
"172.31.19.133:6789",
"192.168.1.3:6789"
]
}
]
metadata:
name: ceph-csi-config

imageFeatures: layering

rbd feature disable kubernetes exclusive-lock,object-map

ceph auth get-or-create client.k8 mon 'profile rbd' osd 'profile rbd pool=k8' mgr 'profile rbd pool=k8'

AQByjPxex0yuDhAAti0AcrUEd3y6AgHG/1rz+A==

sudo ceph mon dump
dumped monmap epoch 1
epoch 1
fsid 24249032-c83e-4cb5-a25a-a620d5439a15
last_changed 2020-07-01 06:33:24.948922
created 2020-07-01 06:33:24.948922
min_mon_release 14 (nautilus)
0: [v2:172.31.17.85:3300/0,v1:172.31.17.85:6789/0] mon.ip-172-31-17-85
1: [v2:172.31.19.133:3300/0,v1:172.31.19.133:6789/0] mon.ip-172-31-19-133
2: [v2:172.31.29.12:3300/0,v1:172.31.29.12:6789/0] mon.ip-172-31-29-12

## cat <<EOF > csi-config-map.yaml

apiVersion: v1
kind: ConfigMap
data:
config.json: |-
[
{
"clusterID": "24249032-c83e-4cb5-a25a-a620d5439a15",
"monitors": [
"172.31.17.85:6789",
"172.31.19.133:6789",
"172.31.29.12:6789"
]
}
]
metadata:
name: ceph-csi-config
EOF
