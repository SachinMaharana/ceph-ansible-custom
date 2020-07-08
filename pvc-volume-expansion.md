# Increasing Volume Size of PV backed by RBD Image on the fly using ceph-csi driver

## Apply StorageClass

<p>Deploy the following storageclass. Make sure to change required values.</p>

```
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: custom-sc
provisioner: rbd.csi.ceph.com
parameters:
  clusterID: c135b7eb-197d-4db7-9a18-efa966b42d2c
  pool: kube
  imageFeatures: layering
  csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
  csi.storage.k8s.io/provisioner-secret-namespace: csi
  csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
  csi.storage.k8s.io/node-stage-secret-namespace: csi
  csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
  csi.storage.k8s.io/controller-expand-secret-namespace: csi
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  - discard
EOF
```

## Apply PVC and Pod.

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: custom-sc
EOF
```

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: csi-rbd-demo-pod
spec:
  containers:
    - name: web-server
      image: nginx
      volumeMounts:
        - name: mypvc
          mountPath: /var/lib/www/html
  volumes:
    - name: mypvc
      persistentVolumeClaim:
        claimName: rbd-pvc
        readOnly: false
EOF
```

## Update PVC

<p>Change storage size in spec.resources.requests.storage using below command</p>

```
kubectl edit pvc rbd-pvc
```

<p>Use the following command to see events for debugging</p>

```
kubectl describe pvc rbd-pvc
```

<p>Volume is Updated Now</p>

```
kubectl get pvc

kubectl get pv | grep {pvc-volume}

kubectl exec -it csi-rbd-demo-pod -- sh
df -h
```
