# Notes On Ceph

### ceph pools snapshot feature.

We can use ceph osd pool mksnap to take a snapshot of an entire pool in one operation.
We can also restore a complete pool when necessary

```
ceph osd pool mksnap
```

Pool Operation

```
ceph osd pool create node 128
ceph osd lspools
rados lspools
ceph osd pool ls detail
eph osd pool set node size 3
 ceph osd pool rename node node-pool
```

We will create an Object then take a snapshot of the pool. We will then proceed to remove the Object from the pool then restore the deleted Object from our pool snapshot.

```
echo "hardwork" > object01.txt

rados -p node-pool put object01 object01.txt

rados -p node-pool ls
object01

rados mksnap snapshot01 -p node-pool
created pool node-pool snap snapshot01

rados lssnap -p node-pool
1	snapshot01	2020.06.20 17:07:17
1 snaps

rados -p node-pool rm object01


rados rollback -p node-pool object01 snapshot01
rolled back pool node-pool to snapshot snapshot01

rados -p node-pool ls
object01

```

## Data Flow

```
echo "ceph" > /tmp/hello

ceph osd pool create node 64 64
pool 'node' created

ceph osd pool set node size 3
set pool 2 size to 3

rados -p node put object01 /tmp/hello

rados -p node ls
object01

ceph osd map node object01
osdmap e20 pool 'node' (2) object 'object01' -> pg 2.1f7eb951 (2.11) -> up ([2,0,1], p2) acting ([2,0,1], p2)


ceph osd find 2
{
    "osd": 2,
    "addrs": {
        "addrvec": [
            {
                "type": "v2",
                "addr": "172.31.25.153:6800",
                "nonce": 16024
            },
            {
                "type": "v1",
                "addr": "172.31.25.153:6801",
                "nonce": 16024
            }
        ]
    },
    "osd_fsid": "848d1578-f52a-4931-91b6-7a7f1ec4bde3",
    "host": "ip-172-31-25-153.us-east-2.compute.internal",
    "crush_location": {
        "host": "ip-172-31-25-153",
        "root": "default"
    }
}

 OSD resides on the host 172.31.25.153.
df -h | grep ceph-2


```

# Storage Provisioning with Ceph

Ceph provides us with multiple abstractions to access data stored in various forms. We can store data within a block device attached to our machine, on a locally mounted filesystem, or remotely accessible object storage using those abstractions

Ceph clients can be categorized under three distinct interfaces
RBD
CephFS
RGW

## Creating and Provisioning RADOS Block Devices

```
ceph tell mon.\* injectargs '--mon-allow-pool-delete=true'

ceph osd pool rm node node --yes-i-really-really-mean-it

ceph osd pool create rbd 64

https://ceph.io/pgcalc/

rbd create ceph-client1-rbd1 --size 10240

rbd ls

rbd info ceph-client1-rbd1

rbd -p rbd info ceph-client1-rbd1

To use the volume we will need to map it to a kernel block device on our client
operating system

Once mapped the device should show up as /dev/rbd0 on the client
node.

Mapping the RBD image to a virtual disk using the RBD kernel driver requires a
Linux kernel of at least version 2.6.32

we will first need to load the rbd kernel module.

modprobe rbd
lsmod | grep rbd

The next step is to map the image to a kernel device on our client node that we can treat just
like a local disk or other block device

rbd map ceph-client1-rbd1

This command might fail if the kernel you are using does not provide support for
certain image features (or attributes) that newer librbd images might possess


rbd info ceph-client1-rbd1

rbd feature disable ceph-client1-rbd1 exclusive-lock,object-map

rbd map ceph-client1-rbd1

rbd showmapped


fdisk -l /dev/rbd0

mkfs.xfs /dev/rbd0

mkdir /mnt/ceph-vol01

 mount /dev/rbd0 /mnt/ceph-vol1

echo "it works" > /mnt/ceph-vol1/rbd.txt

cat /mnt/ceph-vol1/rbd.txt

```
