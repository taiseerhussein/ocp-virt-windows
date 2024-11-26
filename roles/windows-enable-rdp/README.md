# Role: windows-enable-rdp

Description: Enable Remote Desktop connections.

## Notes

This is a very basic/static role. When run, the following registry settings are applied. `UserAuthentication` is set to `1` for the path `HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp` and `fDenyTSConnections` is set to `0` for the path `HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server`. The service `TermService` (Remote Desktop Service) is enabled and set to automatically start on reboot, and a firewall rule is created to allow incomming connections on `3389/tcp`.

## Variables

None.

## Examples

```yaml
- name: Enable Remote Desktop Connections
  ansible.builtin.include_role:
    name: windows-enable-rdp
```
