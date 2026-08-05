# ibrahimmd.raspberrypi.storage

Migrates a Raspberry Pi from SD card to USB/NVMe storage. The role partitions and formats the target device, sets up LVM, clones the SD card filesystem, updates boot configuration, and configures the EEPROM to boot from the new device.

The role is designed to be idempotent — it checks if the Pi is booted from SD card before running and skips all tasks if already booted from USB/NVMe.

## Requirements

- Raspberry Pi OS Bookworm (Debian 12) or Trixie (Debian 13)
- Pi must be booted from SD card
- Target USB or NVMe disk must be unpartitioned
- `community.general` collection
- `ansible.posix` collection

## Dependencies

- `ibrahimmd.homelab.facts` — ensures `/etc/ansible/facts.d` exists

## Assumptions

- Target disk must have no existing partitions before first run
- Boot partition must have `role: boot` defined in `storage_partitions`
- Root partition must have `role: root` defined in `storage_partitions`
- `/home` must be defined as an LVM logical volume with `role: home` in `storage_lvm.lvs`
- LVM partition must have `role: lvm` defined in `storage_partitions`
- Swap partition must have `role: swap` defined in `storage_partitions` if swap is needed
- Role writes local facts to `/etc/ansible/facts.d/storage.fact` on the target device after a successful clone — on subsequent runs the role is skipped automatically if facts are set
- Role will not run if the Pi is not booted from SD card — this is checked at runtime

## Role Variables

### Required

| Variable | Description |
|---|---|
| `storage_disk` | Disk configuration — see below |
| `storage_partitions` | List of partitions to create — see below |
| `storage_lvm` | LVM configuration — see below |

### `storage_disk`

```yaml
storage_disk:
  type: usb           # tran value from lsblk - usb or nvme
  device: /dev/sda    # block device path
  label: gpt          # partition table label
  partition_unit: s   # partition unit - s for sectors
```


### `storage_partitions`

```yaml
storage_partitions:
  - number: 1
    name: usb-bootfs
    role: boot          # boot, root, swap, lvm
    start: 2048s
    end: 1050623s
    fs: vfat
    flags:
      - msftdata
  - number: 2
    name: usb-root
    role: root
    start: 1050624s
    end: 84936703s
    fs: ext4
  - number: 3
    name: swap
    role: swap
    start: 84936704s
    end: 93325311s
    fs: swap
    flags:
      - swap
  - number: 4
    name: lvm
    role: lvm
    start: 93325312s
    end: 100%
    flags:
      - lvm
```

### `storage_lvm`

```yaml
storage_lvm:
  vg:
    name: vg0
    pvs:
      - /dev/sda4
  lvs:
    - name: home
      size: 20G
      fs: ext4
      mount: /home
      role: home        # role: home is cloned from  sdcard /home
    - name: data
      size: 10G
      fs: ext4
      mount: /data
```


### Optional

| Variable | Default | Description |
|---|---|---|
| `storage_eeprom_boot_order` | `0xf41` | EEPROM boot order — `0xf41` tries USB before SD card |
| `storage_preserve_lvm` | `false` | Preserve existing LVM partitions — useful for OS reinstall - not implemented yet |
| `storage_rsync_opts` | `["--force", "--quiet", "-AWHXx"]` | rsync options used during clone |

### `storage_disk`

```yaml
storage_disk:
  device: /dev/sda    # block device path
  label: gpt          # partition table label
```

### `storage_partitions`

```yaml
storage_partitions:
  - number: 1
    name: usb-bootfs
    role: boot          # boot, root, swap, lvm
    start: 2048s
    end: 1050623s
    fs: vfat
    flags:
      - msftdata
  - number: 2
    name: usb-root
    role: root
    start: 1050624s
    end: 84936703s
    fs: ext4
  - number: 3
    name: swap
    role: swap
    start: 84936704s
    end: 93325311s
    fs: swap
    flags:
      - swap
  - number: 4
    name: lvm
    role: lvm
    start: 93325312s
    end: 100%
    flags:
      - lvm
```

### `storage_lvm`

```yaml
storage_lvm:
  vg:
    name: vg0
    pvs:
      - /dev/sda4
  lvs:
  - name: home
    size: 20G
    fs: ext4
    role: home      # home is only supported within lvm
    mount: /home
  - name: rancher
    size: 10G
    fs: ext4
    mount: /var/lib/rancher
 ``

## License

MIT - see [LICENSE](LICENSE) for details.

## Author

ibrahim — [GitHub](https://github.com/ibrahimmd)
