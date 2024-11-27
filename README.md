# Provisioning Windows VMs on OpenShift Virtualization using Ansible

This repository provides a set of Ansible roles designed to facilitate the provisioning and management of Windows virtual machines on OpenShift Virtualization. The roles help automate common tasks such as provisioning Windows VMs, managing VM power states, waiting for network readiness, enabling Remote Desktop connections, and configuring additional storage disks.

Roles included:

* `kubevirt-vm-power-state`: Manages the power state of virtual machines by setting the runStrategy and using API calls. It allows you to power on, gracefully shut down, or forcefully turn off VMs. The role handles different strategies like `Always`, `RerunOnFailure`, `Manual`, and `Halted` to control VM behavior.

* `kubevirt-vm-wait-for-ip`: Waits for a `VirtualMachineInstance` to report an IP address on a specific network interface. This is useful during the provisioning process to ensure that the VM is network-ready before proceeding with tasks that require direct access.

* `windows-enable-rdp`: Enables Remote Desktop Protocol (RDP) connections on Windows virtual machines. It modifies registry settings, configures the Remote Desktop service to start automatically, and adjusts firewall rules to allow incoming RDP connections on port 3389.

* `windows-setup-disks`: Configures additional disks added during VM provisioning. The role initializes disks, creates partitions, formats them with NTFS, and assigns drive letters. It relies on consistent disk naming to map DataVolumes to specific disks within the guest OS.

* `winvm`: Deploy a Windows VM customized with sysprep (`unattend.xml`). Supports a static networking configuration, setting the computer name (hostname), configuring WinRM, etc. 

Each role comes with detailed notes, required variables, and example tasks to help you integrate them into your Ansible workflows effectively.

## Preparing Your Environment

The modules used in these roles have specific Python dependencies and collection requirements. To build an Ansible environment supporting these dependencies/requirements, find a good working directory in your workstation and clone this repository using the following command:

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

## Role Documentation

Documentation for the various roles are included in the README.md in each role's respective directory.

* [kubevirt-vm-power-state](https://github.com/nasx/ocp-virt-windows/blob/main/roles/kubevirt-vm-power-state/README.md)
* [kubevirt-vm-wait-for-ip](https://github.com/nasx/ocp-virt-windows/blob/main/roles/kubevirt-vm-wait-for-ip/README.md)
* [windows-enable-rdp](https://github.com/nasx/ocp-virt-windows/blob/main/roles/windows-enable-rdp/README.md)
* [windows-setup-disks](https://github.com/nasx/ocp-virt-windows/blob/main/roles/windows-setup-disks/README.md)
* [winvm](https://github.com/nasx/ocp-virt-windows/blob/main/roles/winvm/README.md)
