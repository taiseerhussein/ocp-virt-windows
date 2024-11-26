# Role: kubevirt-vm-wait-for-ip

Description: Wait for a `VirtualMachineInstance` to report an IP address for a specific interface.

## Notes

This role would typically be used during the provisioning process. Once a VM is deployed we can use it to wait/verify network connectivity before continuing with additional tasks that require direct network access to the Guest. 

## Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|winvm_interface_name|String|True|Name of the interface as defined in the guest OS|
|winvm_ip|String|True|IP to wait for
|winvm_name|String|True|Name of the virtual machine|
|winvm_namespace|String|True|Namespace containing the virtual machine|

## Examples

```yaml
- name: Wait for VM Network Connectivity
  vars:
    winvm_interface_name: Ethernet Instance 0
    winvm_ip: 172.16.10.100
    winvm_name: mssql02
    winvm_namespace: windows-workloads
  ansible.builtin.include_role:
    name: kubevirt-vm-wait-for-ip
```
