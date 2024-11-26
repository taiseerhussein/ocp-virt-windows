# Role: kubevirt-vm-power-state

Description: Power on, gracefully shutdown or force a VM off using `runStrategy` and/or API calls.

## Notes

Power state is mostly affected by setting the `runStrategy` on a `VirtualMachine` resource. `runStrategy` can be set to one of the following:

|Strategy|Description|
|:---|:---|
|Always|The VM is always running. KubeVirt will attempt to keep the VM running indefinitely. If the VM crashes or the node fails, KubeVirt will automatically restart it on the same or another node.|
|RerunOnFailure|The VM starts immediately and restarts only if it fails (exits with a non-zero exit code). If the VM is stopped gracefully (exits with a zero exit code), it will not restart.|
|Manual|The VM does not start automatically. You must explicitly start or stop the VM using `virtctl` or the API.|
|Halted|The VM is explicitly halted and will not run until the strategy is changed.|

The `kubevirt-vm-power-state` role will mostly use the run strategy to control power state. For example, if `run_strategy` is set to `Always` or `RerunOnFailure` and a VM is powered off the VM will start. If `run_strategy` is set to `Manual`, an API call will be used. The same logic can be used to power off a VM set to `Manual`. The API is also used when powering off a VM and `force` is set to `true` (this effectivly sets `gracePeriod` to `0` in the API request body).

## Variables

|Variable|Type|Required|Description|
|:---|:---|:---|:---|
|vm_power_state|Dictionary|True|All role specific vars defined in this dictionary (see below)|
|vm_power_state.force|Boolean|False|When set to `true`, the VM will be force powered off (i.e. terminationGracePeriodSeconds will be set to 0).
|vm_power_state.name|String|True|Name of the virtual machine.|
|vm_power_state.namespace|String|True|Namespace containing the virtual machine.|
|vm_power_state.running|Boolean|True|Should the VM be powered on or off.|
|vm_power_state.run_strategy|Enum|True|The run strategy to use to control power state. When `running` is `true`, `run_strategy` must be one of `Always`, `RerunOnFailure`, or `Manual`. When `running` is set to  `false`, `run_strategy` must be one of `RerunOnFailure`, `Manual`, or `Halted`|

## Examples

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
