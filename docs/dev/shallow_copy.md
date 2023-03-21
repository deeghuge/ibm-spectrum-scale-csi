# Snapshots as read-only volumes

CSI spec do not have concept of mounting a snapshot. The only way is to create
new volume by copying content of snapshot and then mount that volume for
workloads. 

IBM Storage Scale exposes snapshots as special, read-only directories
located in `<fileset>/.snapshots`. IBM Storage Scale CSI can already provision 
writable volumes with snapshots as their data source, where snapshot contents 
are cloned to the newly created volume. However, cloning a snapshot to 
volume is a very expensive operation as the data needs to be fully copied. 
When the need is to only read snapshot contents, snapshot cloning is extremely
inefficient and wasteful.

This proposal describes a way for IBM Storage Scale CSI to expose snapshots as 
shallow, read-only volumes, without needing to clone the underlying snapshot 
data.


## Design

Key points:

* Volume source is a snapshot, volume access mode is `*_READER_ONLY`.
* No actual new volume are created in Storage Scale.
* Volume Handle must contain all the required details to find snapshot in 
a fileset for various operations 


### Controller operations

Care must be taken when handling life-times of relevant storage resources. When
a shallow volume is created, what would happen if:

* _Parent volume of the snapshot is removed while the shallow volume still
  exists?_

  Deletion of volume will fails since snapshot exist for it

* _Source snapshot from which the shallow volume originates is removed while
  that shallow volume still exists?_

  We need to make sure this doesn't happen and some book-keeping is necessary.


#### Book-keeping for shallow volumes

As mentioned above, this is to protect shallow volumes, should their source
snapshot be requested for deletion.

VolumeCreation : Reference count will be added to Snapshot

SnapshotDeletion : Delete snapshot if no reference to shallow volume otherwise
Fail with error saying shallow volume exist.

VolumeDeletion : Reference count will be removed from Snapshot


#### `CreateVolume`

A read-only volume with snapshot source would be created under these conditions:

1. `CreateVolumeRequest.volume_content_source` is a snapshot,
1. `CreateVolumeRequest.volume_capabilities[*].access_mode` is any of read-only
   volume access modes.
1. StorageClass parameters
    VolBackendFs : fsName
    ShallowCopy: true
    Version:1 

Things to look out for:

* _What's the volume size?_

  It can be Zero or anything as this is not actually consuming any storage but 
  we have to keep cloning in mind for size

### `DeleteVolume`

Update the snapshot reference.

### `CreateSnapshot`

Not supported

### `ControllerExpandVolume`

Not supported

### `VolumeClone`

Can be supported

### `Subdir/fsGroup/SELinux`

Will fail if dir does not exist since snapshot is readOnly


### `NodePublishVolume`, `NodeUnpublishVolume`

Bind mount snapshot path in volumeHandle to kubelet path

### `NodeGetVolumeStats`

Not supported

### Volume Handle
VolumeHandle: 0;3;<clusterID>;<fsuid>;<independent fileset name>;<snapshot name>;<>;<Complete Path> 
