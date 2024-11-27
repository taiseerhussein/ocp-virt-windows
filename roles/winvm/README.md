# Role: winvm

Description: Provision a Windows Virtual Machine customized using sysprep (`unattend.xml`).

## Notes

This role will customize a Windows virtual machine using unattend.xml. Settings include a static network configuration, setting the computer name (i.e. hostname), setting the Administrator password, configuring WinRM via a Power Shell script and setting other OOBE properties like locale, accepting EULA, etc.

> [!NOTE]
> The `unattend.xml` file is stored in a `ConfigMap` in the same `Namespace` as the virtual machine. A randomly generated Administrator password is set in the `unattend.xml` and later changed to the value in `winvm_admin_password` after the VM is booted.

> [!IMPORTANT]  
> When using `winvm_additional_disks`, this role only adds the disks (`DataVolumes`) to the virtual machine. To setup the disks (i.e. initialize, partition and format) be sure to pass the same `winvm_additional_disks` var to the `windows-setup-disk` role.

## Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|`winvm_ad_admin_password`|String|False|Password for Active Directory Administrator account|
|`winvm_ad_admin_user`|String|False|Username used to join the VM to the Domain|
|`winvm_ad_domain`|String|False|Domain to join|
|`winvm_ad_domain_join`|Boolean|False|When set to `true`, join the VM with Active Directory Domain. Default is `false`.|
|`winvm_ad_domain_server`|String|False|IP address or hostname of a Domain Controller|
|`winvm_additional_disks`|[List\<Disks\>](https://github.com/nasx/ocp-virt-windows/blob/main/roles/windows-setup-disks/README.md#custom-types)|False|List of additional disks to add to VM (see [`winvm-setup-disks`](https://github.com/nasx/ocp-virt-windows/tree/main/roles/windows-setup-disks) for details)|
|`winvm_admin_password`|String|True|Password to set for the local Administrator account|
|`winvm_cpu_cores`|String|True|Number of cores to set for the VM|
|`winvm_cpu_sockets`|String|True|Number of sockets to set for the VM|
|`winvm_cpu_threads`|String|True|Number of threads to set for the VM|
|`winvm_dns_domain`|String|True|DNS domain to set for the VM|
|`winvm_dns_server`|String|True|DNS server to set for the VM|
|`winvm_gateway`|String|True|Network gateway|
|`winvm_interface_name`|String|True|Name of the network interface in the VM template disk as seen from Network Connections|
|`winvm_ip`|String|True|Static IP address to assign the VM|
|`winvm_ip_cidr`|String|True|CIDR block used for static ip address|
|`winvm_name`|String|True|The name of the `VirtualMachine` resource. This string will be converted to uppercase and become the VM Computer Name|
|`winvm_namespace`|String|True|The Kubernetes `Namespace` that should contain the `VirtualMachine` resource|
|`winvm_network_name`|String|True|The name of the `NetworkAttachmentDefinition` that `winvm_interface_name` will connect to|
|`winvm_ram`|String|True|Human readible RAM size (e.g. 8192Mb, 24Gi)|
|`winvm_rootdisk_size`|String|True|Human readible disk size (e.g. 100Gi)|
|`winvm_rootdisk_source_pvc`|Dictionary|True|This dictionary should contain two key/value pairs, `winvm_rootdisk_source_pvc.name` is the name of the PVC containing the root disk we are cloning and `winvm_rootdisk_source_pvc.namespace` is the namespace containing the PVC|
|`winvm_rootdisk_storageclass`|String|True|The `StorageClass` to use for the VM root disk|

## Examples

> [!TIP]
> For better readability, it is recommended to store `winvm_` variables in a vars file. A complete example vars file is located in the root of the repository under [`vars/example-vm.yaml`](https://github.com/nasx/ocp-virt-windows/blob/main/vars/example-vm.yaml).

```yaml
- name: Deploy Windows VM
  vars:
    winvm_admin_password: ChangeMe!
    winvm_cpu_cores: 1
    winvm_cpu_sockets: 6
    winvm_cpu_threads: 1
    winvm_dns_domain: lab.uc2.io
    winvm_dns_server: 172.16.80.20
    winvm_gateway: 172.16.10.254
    winvm_interface_name: Ethernet Instance 0
    winvm_ip: 172.16.10.252
    winvm_ip_cidr: 24
    winvm_name: ServerTest03
    winvm_namespace: windows-virtual-machines
    winvm_network_name: default/vlan-100
    winvm_ram: 32Gi
    winvm_rootdisk_size: 60Gi
    winvm_rootdisk_source_pvc:
      name: server-2022-template-volume
      namespace: windows-virtual-machines
    winvm_rootdisk_storageclass: odf-storagecluster-ceph-rbd-virtualization
  ansible.builtin.include_role:
    name: kubevirt-vm-wait-for-ip
```