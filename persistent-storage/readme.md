# Storage : Persistent Volumes, Claims and Storage Classes

## Overview

For local k8s cluster you're going to be on a single-node setup (usually) so you'll be using static provisioning. You only need a Persistent Volume using `hostPath` for this (see `pv-hostpath.yaml`).

There are many options for your `PersistentVolume`, so this all depends on how you need kanidm to work. Once you get into a cloud platform, you might want to look into `Storage Classes` for dynamic provisioning of PV's.

## PersistentVolumes

### Local `hostPath`

create the hostPath Persistent Volume for your microK8s or minikube setup: `'microk8s.kubectl apply -f ./pv-hostpath.yaml'`.

### Cloud Provider Options

You can setup different PV's here based on the listed [Types of Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) on kubernetes documentation:

* GCEPersistentDisk
* AWSElasticBlockStore
* AzureFile
* AzureDisk
* CSI
* FC (Fibre Channel)
* FlexVolume
* Flocker
* NFS
* iSCSI
* RBD (Ceph Block Device)
* CephFS
* Cinder (OpenStack block storage)
* Glusterfs
* VsphereVolume
* Quobyte Volumes
* HostPath (Single node testing only -- local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
* Portworx Volumes
* ScaleIO Volumes
* StorageOS

## PersistentVolumesClaims

Your kanidm PVC is taken care of in `/kanidm-k8s/helm/kanidm/templates/pvc.yaml`

## Storage Classes

These are for dynamic PV provisioning and I need to complete this - example of what it looks like:

```yaml
 apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: staticManagedVolumeRetain
  provisioner: kubernetes.io/azure-disk
  parameters:
    kind: Managed
    storageaccounttype: Standard_LRS
  reclaimPolicy: Retain
```

Basically, you `kubectl apply -f ./storageClass.yaml` to create it, then in your `pvc.yaml` you would specify `staticManagedVolumeRetain` as the `storageClassName` and your claim would dynamically get itself a PV with a specified size and accessModes using this storage class, and mount it.
