# Provisioning Windows VMs on OpenShift Virtualization using Ansible

This repository provides a set of Ansible roles designed to facilitate the provisioning and management of Windows virtual machines on OpenShift Virtualization. The roles help automate common tasks such as provisioning Windows VMs, managing VM power states, waiting for network readiness, enabling Remote Desktop connections, and configuring additional storage disks.

## Preparing Your Environment

To being, find a good working directory in your workstation and clone this repository using the following command:

```shell
git clone https://github.com/nasx/ocp-virt-windows.git
```

Next, we will create a Python Virtual Environment.

```shell
python -m venv /path/to/virtual/environment
```

Activate the virtual environment.

```shell
. /path/to/virtual/environment/bin/activate
```

Install required Python dependencies using `requirements.txt` in the root of the repository.

```shell
pip install -U -r requirements.txt
```

Finally, make sure you are in the root directory of the repository and install the required Ansible collections.

```shell
ansible-galaxy collection install -r collections/requirements.yaml
```

## Role: kubevirt-vm-power-state

Description: Power on, gracefully shutdown or force a VM off using `runStrategy` and/or API calls.

### Notes

Power state is mostly affected by setting the `runStrategy` on a `VirtualMachine` resource. `runStrategy` can be set to one of the following:

|Strategy|Description|
|:---|:---|
|Always|The VM is always running. KubeVirt will attempt to keep the VM running indefinitely. If the VM crashes or the node fails, KubeVirt will automatically restart it on the same or another node.|
|RerunOnFailure|The VM starts immediately and restarts only if it fails (exits with a non-zero exit code). If the VM is stopped gracefully (exits with a zero exit code), it will not restart.|
|Manual|The VM does not start automatically. You must explicitly start or stop the VM using `virtctl` or the API.|
|Halted|The VM is explicitly halted and will not run until the strategy is changed.|

The `kubevirt-vm-power-state` role will mostly use the run strategy to control power state. For example, if `run_strategy` is set to `Always` or `RerunOnFailure` and a VM is powered off the VM will start. If `run_strategy` is set to `Manual`, an API call will be used. The same logic can be used to power off a VM set to `Manual`. The API is also used when powering off a VM and `force` is set to `true` (this effectivly sets `gracePeriod` to `0` in the API request body).

### Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|vm_power_state|Dictionary|<span style="color:green">True</span>|All role specific vars defined in this dictionary (see below)|
|vm_power_state.force|Boolean|<span style="color:red">False</span>|When set to `true`, the VM will be force powered off (i.e. terminationGracePeriodSeconds will be set to 0).
|vm_power_state.name|String|<span style="color:green">True</span>|Name of the virtual machine.|
|vm_power_state.namespace|String|<span style="color:green">True</span>|Namespace containing the virtual machine.|
|vm_power_state.running|Boolean|<span style="color:green">True</span>|Should the VM be powered on or off.|
|vm_power_state.run_strategy|Enum|<span style="color:green">True</span>|The run strategy to use to control power state. When `running` is `true`, `run_strategy` must be one of `Always`, `RerunOnFailure`, or `Manual`. When `running` is set to  `false`, `run_strategy` must be one of `RerunOnFailure`, `Manual`, or `Halted`|

### Examples

```yaml
- name: Set run strategy to Manual and power on VM
  vars:
    vm_power_state:
      name: windows-vm
      namespace: windows-virtual-machines
      running: true
      run_strategy: Manual
  ansible.builtin.include_role:
    name: kubevirt-vm-power-state

- name: Set run strategy to RerunOnFailure and power off VM
  vars:
    vm_power_state:
      name: mssql01
      namespace: windows-virtual-machines
      running: false
      run_strategy: RerunOnFailure
  ansible.builtin.include_role:
    name: kubevirt-vm-power-state

- name: Set run strategy to Halted and force power off VM
  vars:
    vm_power_state:
      force: true
      name: web-server-01
      namespace: windows-virtual-machines
      running: false
      run_strategy: Halted
  ansible.builtin.include_role:
    name: kubevirt-vm-power-state
```

## Role: kubevirt-vm-wait-for-ip

Description: Wait for a `VirtualMachineInstance` to report an IP address for a specific interface.

### Notes

This role would typically be used during the provisioning process. Once a VM is deployed we can use it to wait/verify network connectivity before continuing with additional tasks that require direct network access to the Guest. 

### Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|vm_interface_name|String|<span style="color:green">True</span>|Name of the interface as defined in the guest OS|
|vm_ip|String|<span style="color:green">True</span>|IP to wait for
|vm_name|String|<span style="color:green">True</span>|Name of the virtual machine|
|vm_namespace|String|<span style="color:green">True</span>|Namespace containing the virtual machine|

### Examples

```yaml
- name: Wait for VM Network Connectivity
  vars:
    vm_interface_name: Ethernet Instance 0
    vm_ip: 172.16.10.100
    vm_name: mssql02
    nm_namespace: windows-workloads
  ansible.builtin.include_role:
    name: kubevirt-vm-wait-for-ip
```

## Role: windows-enable-rdp

Description: Enable Remote Desktop connections.

### Notes

This is a very basic/static role. When run, the following registry settings are applied. `UserAuthentication` is set to `1` for the path `HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp` and `fDenyTSConnections` is set to `0` for the path `HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server`. The service `TermService` (Remote Desktop Service) is enabled and set to automatically start on reboot, and a firewall rule is created to allow incomming connections on `3389/tcp`.

### Variables

None.

### Examples

```yaml
- name: Enable Remote Desktop Connections
  ansible.builtin.include_role:
    name: windows-enable-rdp
```

## Role: windows-setup-disks

Description: Configure (initialize, partition and format) additional disks defined at provisioning time.

### Notes

When configuring additional disks deployed with the `winvm` role, make sure the same `vm_additional_disks` variable is used. This is important because each disk's serial number is set to the name of the disk (i.e. DataVolume). This serial number attribute is discoverable using the `win_disk_facts` module, allowing us to map a DataVolume to a specific disk device within the OS.

Once a disk is identified the following writes occur:
* Initialized with a GPT partition table
* A single partition is created that consumes all free space on the disk
* Disk is formatted with NTFS

Since this role is typically used during the provisioning process, additional disks should be uninitialized. For added safety, none of the write actions will occur if any identified additional disk has a partition or an expected additional disk is missing.

### Custom Types

|Custom Type|Variable|Type|Required|Description|
|:---|:---|:---|:---|:---|
|Disks|bus|String|<span style="color:green">True</span>|Bus type to use. Should be set to one of `virtio` (preferred), `sata`, `scsi` or `usb`|
|Disks|name|String|<span style="color:green">True</span>|Name of the volume in the `VirtualMachineInstance` spec. Also used as the disk serial number (see notes)|
|Disks|size|String|<span style="color:green">True</span>|Size of the disk to be created|
|Disks|storage_class|String|<span style="color:green">True</span>|StorageClass used by the DataVolume (disk) to create PVC|
|Disks|drive_letter|String|<span style="color:green">True</span>|Drive letter to assign disk in Windows|

### Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|vm_additional_disks|List\<Disks\>|<span style="color:green">True</span>|List of dictionaries with each element representing a disk and its properties (see custom types above)|

### Examples

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
