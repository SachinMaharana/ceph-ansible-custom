# ceph-ansible

Ansible playbooks for Ceph, the distributed filesystem.

Please refer to our hosted documentation here: http://docs.ceph.com/ceph-ansible/master/

You can view documentation for our `stable-*` branches by substituting `master` in the link
above for the name of the branch. For example: http://docs.ceph.com/ceph-ansible/stable-3.0/

# Documentation

Please refer the following repo to create a VM's for ceph-cluster creation in AWS. As this is for testing and learning creation of ceph cluster, this setup shouldn't be used for prodcution purpose.

```
https://github.com/SachinMaharana/ceph-cluster-vms
```

The above terraform creates a total of 7 VM's , 3 for mons, 3 for osds and 1 for client. A gp2 disk of size 30GB volume is attacthed to osds.

## Get the instance information from the following commands

`` aws ec2 describe-instances --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value|[0],State.Name,PrivateIpAddress,PublicIpAddress,PublicDnsName,BlockDeviceMappings[*].DeviceName]' --output text | column -t | grep running ``

## SSH in one of the mons and get ceph cluster details

```
ssh centos@ec2-3-15-185-202.us-east-2.compute.amazonaws.com

sudo su -

ceph -s
```

## Find the public IP of active mgrs from `ceph -s` and go to IP:8443 to access ceph dashboard.

## Use Block Device

To add a client node, add a client host group in hosts file. And execute the following to start client playbook(optional. already added in terraform file.)

`ansible-playbook site.yml -i hosts --limit clients`

### Login to Client Node

```
ssh centos@ec2-18-220-152-238.us-east-2.compute.amazonaws.com

sudo ceph osd pool create rbd 8
pool 'rbd' created

sudo rbd pool init rbd

sudo rbd create rbd01 --size 10G --image-feature layering

sudo rbd ls -l
NAME SIZE PARENT FMT PROT LOCK
rbd01 10 GiB 2

sudo rbd map rbd01
/dev/rbd0

rbd showmapped
id pool namespace image snap device
0 rbd rbd01 - /dev/rbd0

sudo mkfs.xfs /dev/rbd0

sudo mount /dev/rbd0 /mnt

df -hT
Filesystem Type Size Used Avail Use% Mounted on
/dev/xvda1 xfs 8.0G 1.4G 6.7G 17% /
devtmpfs devtmpfs 1.9G 0 1.9G 0% /dev
tmpfs tmpfs 1.9G 0 1.9G 0% /dev/shm
tmpfs tmpfs 1.9G 17M 1.9G 1% /run
tmpfs tmpfs 1.9G 0 1.9G 0% /sys/fs/cgroup
tmpfs tmpfs 379M 0 379M 0% /run/user/1000
/dev/rbd0 xfs 10G 33M 10G 1% /mnt
```

## Use As Filesystem

To add a mds, add a mds host group in hosts file. And execute the following to start mds playbook

`ansible-playbook site.yml -i hosts --limit mdss`

### Login to MDS Node.

`ssh centos@ec2-3-15-185-202.us-east-2.compute.amazonaws.com`

```
sudo ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]

sudo ceph mds stat
cephfs:1 {0=ip-172-31-30-12=up:active} 2 up:standby

sudo ceph fs status cephfs
cephfs - 0 clients
======
+------+--------+-----------------+---------------+-------+-------+
| Rank | State | MDS | Activity | dns | inos |
+------+--------+-----------------+---------------+-------+-------+
| 0 | active | ip-172-31-30-12 | Reqs: 0 /s | 10 | 13 |
+------+--------+-----------------+---------------+-------+-------+
+-----------------+----------+-------+-------+
| Pool | type | used | avail |
+-----------------+----------+-------+-------+
| cephfs_metadata | metadata | 96.0k | 27.4G |
| cephfs_data | data | 0 | 27.4G |
+-----------------+----------+-------+-------+
+------------------+
| Standby MDS |
+------------------+
| ip-172-31-24-182 |
| ip-172-31-24-86 |
+------------------+
MDS version: ceph version 14.2.9 (581f22da52345dba46ee232b73b990f06029a2a0) nautilus (stable)
```

### Get admin key in MDS node.

`ceph-authtool -p /etc/ceph/ceph.client.admin.keyring > admin.key`

## On Client Node:

`ssh centos@ec2-18-220-152-238.us-east-2.compute.amazonaws.com`

```
yum -y install ceph-fuse
```

Copy the admin key above(obtained in MDS node) in client node.

```
chmod 600 admin.key
```

```
mount -t ceph 172.31.30.12:6789:/ /mnt -o name=admin,secretfile=admin.key
```

```
df -hT
172.31.30.12:6789:/ ceph 28G 0 28G 0% /mnt
```
