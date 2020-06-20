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
