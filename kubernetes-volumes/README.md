# Kubernetes Volumes

On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers. 

One problem occurs when a container crashes or is stopped. Container state is not saved so all of the files that were created or modified during the lifetime of the container are lost. 

Kubernetes supports many types of volumes. A Pod can use any number of volume types simultaneously.

- Ephemeral volume types have a lifetime of a pod, but persistent volumes exist beyond the lifetime of a pod. 
- When a pod ceases to exist, Kubernetes destroys ephemeral volumes; however, Kubernetes does not destroy persistent volumes. 
- For any kind of volume in a given pod, data is preserved across container restarts.


## Some Important types of Volumes

- AWS EBS third party storage driver (aws-ebs-csi-driver)
- azure file third party storage driver (azuredisk-csi-driver)
- configMap
- emptyDir
- hostPath
- local
- secret


## PersistentVolume and PersistentVolumeClaim

`PersistentVolume`

- A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. 
- It is a resource in the cluster just like a node is a cluster resource. 
- PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV.

`PersistentVolumeClaim`

- PVC is a request for storage by a user. 
- It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. 
- Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany, ReadWriteMany, or ReadWriteOncePod)

### Persistent volume provisioning

`Static`

A cluster administrator creates a number of PVs. They carry the details of the real storage, which is available for use by cluster users.

`Dynamic`

- When none of the static PVs the administrator created match a user's PersistentVolumeClaim, the cluster may try to dynamically provision a volume specially for the PVC. 
- This provisioning is based on StorageClasses

### Volume Reclaiming

When a user is done with the volume, they can remove the PVC parameter from their Deployment/Pod. The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim.

Currently, volumes can either be Retained, Recycled, or Deleted.

