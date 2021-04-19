# Kubernetes Volume Management

In today's business model, data is highly fundamental for any business. There is the need to consume data that are already available and to create data to become available. Containers running inside Pods can either consume or/and produce data, and this data might be available inside and outside the Pods. Kubernetes provides a way to carry about data management and to attach **Volume** for Pods, the **PersistentVolume** and **PersistentVolumeClaim**

### Volumes

Once Pod is ephemeral, all data stored within the Pod are lost if the container crashes. Even when the **kubelet** restarts the Pod, data are not going to be recovered. Also, when there is the need to share data among Pods. These problems are overcoming with [**Volumes**](https://kubernetes.io/docs/concepts/storage/volumes/), a storage abstraction that allow various storage technologies to be used by Kubernetes. **Volumes is basically a mount point on the container's file system and a path in the OS file system**. The OS file system is linked to the container's mount point. So, Volumes provide a way to save data even when the container crashes.

```yaml
spec:
  template:
    spec:
      containers:
        - name: mysql
          image: mysql:latest
          volumeMounts:	# mounts the volume containers
            - name: mysql-volume
              mountPath: /var/lib/mysql	# mount point on container
      volumes:			# creates a volume in SO
        - name: mysql-volume
          hostPath:
            path: /mnt/mysql-v			# local path
```

Once a Volume is created and linked to a Pod, all containers within the Pod share that Volume. If any container restarts, the Volume is still in live. However, ephemeral volumes are deleted if **the Pod** is deleted. So, if the Pod crashes, its Volume crashes too.

The directory which is mounted inside the Pod depends on the [VolumeType](https://kubernetes.io/docs/concepts/storage/volumes/#volume-types), which determines the size, access modes, file type an so on. There are a lot of different volume types:

- **emptyDir**: this Volume is an empty one that is created when the container is scheduled on the worker node. If the Pod crashes, this Volume crashes too.
- **hostPath**: a path in the local SO is shared with the container as its volume. All data are read/written inside this path. If the Pod crashes, **this Volume is still available** in the host path.
- **[gcePersistentDisk](https://cloud.google.com/compute/docs/disks/), [awsElasticBlockStore](https://aws.amazon.com/ebs/), [azureDisk](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/managed-disks-overview) and [azureFile](https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md)**: these volume types mount a Volume for Google Compute Engine, AWS, Azure Data and Azure File, respectively.
- **cephfs**: this volume type mounts an existing **[CephFS](https://ceph.io/ceph-storage/)** volume into a Pod. This volume type unmount the Pod's volume when it closes, and saves all the volume's contents.
- [nfs](https://en.wikipedia.org/wiki/Network_File_System), [iSCSI](https://en.wikipedia.org/wiki/ISCSI), secret and configMap: these volume types are the ones used to mount the respectively volume type into Pod.

To provide a way persistent data even when the Pod crashed is achieve using Persistent Volumes.

### PersistentVolumes (PV)

Volumes are not recommended to use directly once this Kubernetes object is destroyed when a  Pod is deleted and all data are lost. So, to overcome this problem there is the [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV), which stores all data in the local disk and, even when Pod crashes, data remains unchanged. PV provides a way to manage and consume persistent storage. PV provides **an abstraction layer between the administrator and the user/application**, on which the details about the storage is hidden and managed by the administrator, whereas the user/application just consumes and writes in this storage. The best part: **PV has a life cycle independently of any Pod**.

PV can be:

- **statically**: this PV is created by the administration, which specifies the details of the storage and users consumes this PV.
- **dynamically**: when the there are no one PV that fit the user's PVC, Kubernetes can create a PV dynamically based on the **[StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)** resource. A StorageClass enables administrators to describe the "classes" of storage they offer. A PV is created through a dynamic request in a StorageClass.

There are a lot of [Volume Types](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes) available to manage storage using PersistentVolume, such as GCEPersistentDisk, AWSElasticBlockStore, AzureFile, AzureDisk, CephFS, NFS and iSCSI.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 500Mi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: local-storage
  local:
    path: /mnt/mysql-pv
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

However, the persistent is just achieve through a [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (PVC). A PVC is the way a user/application requires a persistent volume. PVC requests specific size, access mode, volume type and so on. **PVC consumes a PV**.

### PersistentVolumeClaims (PVC)

A [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (PVC) is a request for storage by a user. Users may specify some attributes to match the PVC with a PV, for example, the type, the access mode, the size, and so on. There are three access modes available: **ReadWriteOnce** (read-write by a single node), **ReadOnlyMany** (read-only by any nodes), and **ReadWriteMany** (read-write by many nodes).

In the following figure we can see how the stuff works. First of all, the administrator creates a PV, and more than one can be available. This PV has its specifics, such as file type, size and access mode. Then, the client creates a PVC specifying the desired PV. Once the desired PV is found, it is **bound** to a PVC and the client can use this PV. Finally, the Pod receives the PV and shares it with all of its containers.

![pv-pvc](https://courses.edx.org/assets/courseware/v1/fcf7dcbb00433275fbbaa7bd5a8d400e/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/pvc2.png)

When the Pod does not use the PV anymore, it can be released. The PV may have in `persistentVolumeReclaimPolicy` one of the following policies:

- **reclaimed**: the administrator reclaims the PV.
- **deleted**: in this case, both data and volume are deleted.
- **recycled**: only data is deleted, but the PV comes back to PV Pool.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 500Mi
  storageClassName: slow
  selector:
    matchLabels:
      release: stable
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

### StorageClass

[StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/).

### Container Storage Interface (CSI)

In the past, different container orchestrators, such as Kubernetes, Mesos, Docker and Cloud Foundry, used to have their own Volume management way. This become so hard to storage vendors develop and maintain a lot of orchestrators plugins. The [Container Storage Interface](https://kubernetes.io/docs/concepts/storage/volumes/#csi) (CSI) is the standard interface for container orchestration on which all plugins developed based in CSI is compatible with several different storage vendors.

----

You should now be able to:

- Explain the need for persistent data management.
- Compare Kubernetes Volume types.
- Discuss PersistentVolumes and PersistentVolumeClaims.