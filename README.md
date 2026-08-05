# Ansible Collection - ibrahimmd.raspberrypi


Ansible collection for Raspberry Pi homelab setup, configuration and USB/NVMe disk migration

## Roles

| Role | Description | Documentation |
|---|---|---|
| `storage` | SD card to USB/NVMe migration | [README](roles/storage/README.md) |
| `bootstrap` | Base Pi configuration | [README](roles/bootstrap/README.md) (TODO) |

## Requirements

- Ansible 2.20+
- Raspberry Pi OS Trixie (Debian 13)

## Installation

```bash
ansible-galaxy collection install git+https://github.com/ibrahimmd/ansible-collection-raspberrypi.git
```
