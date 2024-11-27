# Role: windows-setup-disks

Description: Configure (initialize, partition and format) additional disks defined at provisioning time.

## Notes

When configuring additional disks deployed with the `winvm` role, make sure the same `vm_additional_disks` variable is used. This is important because each disk's serial number is set to the name of the disk (i.e. DataVolume). This serial number attribute is discoverable using the `win_disk_facts` module, allowing us to map a DataVolume to a specific disk device within the OS.

Once a disk is identified the following writes occur:
* Initialized with a GPT partition table
* A single partition is created that consumes all free space on the disk
* Disk is formatted with NTFS

Since this role is typically used during the provisioning process, additional disks should be uninitialized. For added safety, none of the write actions will occur if any identified additional disk has a partition or an expected additional disk is missing.

## Custom Types

|Custom Type|Variable|Type|Required|Description|
|:---|:---|:---|:---|:---|
|Disks|`bus`|String|True|Bus type to use. Should be set to one of `virtio` (preferred), `sata`, `scsi` or `usb`|
|Disks|`drive_letter`|String|True|Drive letter to assign disk in Windows|
|Disks|`name`|String|True|Name of the volume in the `VirtualMachineInstance` spec. Also used as the disk serial number (see notes)|
|Disks|`size`|String|True|Size of the disk to be created|
|Disks|`storage_class`|String|True|StorageClass used by the DataVolume (disk) to create PVC|

## Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|`vm_additional_disks`|List\<Disks\>|True|List of dictionaries with each element representing a disk and its properties (see custom types above)|

## Examples

```yaml
- name: Configure Additional Disks
  vars:
    vm_additional_disks:
      - bus: virtio
        drive_letter: E
        name: sql1
        size: 100Gi
        storage_class: odf-storagecluster-ceph-rbd-virtualization
      - bus: virtio
        drive_letter: F
        name: sql2
        size: 50Gi
        storage_class: odf-storagecluster-ceph-rbd-virtualization
  ansible.builtin.include_role:
    name: windows-setup-disks
```
