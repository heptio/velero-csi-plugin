# Velero CSI plugins


This repository contains Velero plugins for snapshotting CSI backed PVCs using the [CSI _beta_ snapshot APIs][7].

These plugins are currently in beta as of the [Velero 1.4 release][1] and will reach GA shortly after the CSI volumesnapshotting APIs in upstream Kubernetes reach GA.

For a list of prerequisites and installation instructions, please refer to our documentation [here][2].


## Kinds of Plugins Included

### PVCBackupItemAction

A plugin of type BackupItemAction that backs up `persistentvolumeclaims` which are backed by CSI volumes.

This plugin will create a [CSI VolumeSnapshot][3] which in turn triggers the CSI driver to perform the snapshot operation on the volume.

### VolumeSnapshotBackupItemAction

A plugin of type BackupItemAction that backs up [`volumesnapshots.snapshot.storage.k8s.io`][3].

When invoked, this plugin will capture information about the underlying [`volumesnapshotcontent.snapshot.storage.k8s.io`][4] in the annotations of the volumesnapshots being backed up. This plugin will also return the underlying [`volumesnapshotcontent.snapshot.storage.k8s.io`][4] and the associated [`snapshot.storage.k8s.io.volumesnapshotclasses`][5] as additional resources to be backed up.

### VolumeSnapshotContentBackupItemAction

A plugin of type BackupItemAction that backs up [`volumesnapshotcontent.snapshot.storage.k8s.io`][4]. 

This plugin will look for snapshot delete operation secrets from the [annotations][6] on the VolumeSnapshotContent object being backed up.

### VolumeSnapshotClassBackupItemAction

A plugin of type BackupItemAction that backs up [`snapshot.storage.k8s.io.volumesnapshotclasses`][5].

This plugin will look for snapshot list operation secret from the [annotations][6] on the VolumeSnapshotClass object being backed up.

### PVCRestoreItemAction

A plugin of type RestoreItemAction that restores `persistentvolumeclaims` which were backed up by [PVCBackupItemAction](#PVCBackupItemAction).

This plugin will modify the spec of the `persistentvolumeclaim` being restored to use the VolumeSnapshot, created during backup, as the data source ensuring that the newly provisioned volume, to satisfy this claim, may be pre-populated using the VolumeSnapshot.

### VolumeSnapshotRestoreItemAction

A plugin of type RestoreItemAction that restores [`volumesnapshots.snapshot.storage.k8s.io`][3]. 

This plugin will use the annotations, added during backup, to create a [`volumesnapshotcontent.snapshot.storage.k8s.io`][4] and statically bind it to the volumesnapshot object being restored. The plugin will also set the necessary [annotations][6] if the original volumesnapshotcontent had snapshot deletion secrets associated with it. 

### VolumeSnapshotClassRestoreItemAction

A plugin of type RestoreItemAction that restores [`snapshot.storage.k8s.io.volumesnapshotclasses`][5]. 

This plugin will use the [annotations][6] on the object being restored to return, as additional items, any snapshot lister secret that is associated with the volumesnapshotclass.

## Building the plugins

To build the plugins, run

```bash
$ make
```

## Known shortcomings

We are tracking known limitations with the plugins [here][2]

[1]: https://github.com/vmware-tanzu/velero/releases
[2]: https://velero.io/docs/csi
[3]: https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots
[4]: https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents
[5]: https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/
[6]: https://github.com/kubernetes-csi/external-snapshotter/blob/master/pkg/utils/util.go#L59-L60
[7]: https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-cis-volume-snapshot-beta/
