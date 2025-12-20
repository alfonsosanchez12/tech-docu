## Install Incus
`sudo dnf install incus`

## 'incus-admin' group

**Check**
```
id
cat /etc/group
```

**Add local user to incus-admin group**
`sudo usermod -aG incus-admin $USER`

## Enable incus socket
`sudo systemctl enable --now incus.socket`

## If re-using an lvm storage pool

**Check**
```
> lsblk
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0 223.6G  0 disk
└─ssd--data-sda1       252:1    0 223.6G  0 lvm
.
.
.

> sudo vgs
  VG           #PV #LV #SN Attr   VSize    VFree
  fedora         1   1   0 wz--n-  952.28g 937.28g
  slow-storage   2   1   0 wz--n- <763.85g      0
  ssd-data       1   1   0 wz--n- <223.57g      0
  
> sudo lvs
  LV     VG           Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   fedora       -wi-ao----   15.00g
  hdslow slow-storage -wi-a----- <763.85g
  sda1   ssd-data     -wi-a----- <223.57g
```

**Removing lvm storage to dedicate to storage pool**
```
sudo lvremove /dev/mapper/ssd--data-sda1
sudo vgremove ssd-data
```

## Configure incus
```
> incus admin init

Would you like to use clustering? (yes/no) [default=no]:
Do you want to configure a new storage pool? (yes/no) [default=yes]:
Name of the new storage pool [default=default]:
Name of the storage backend to use (lvm, lvmcluster, dir, btrfs) [default=btrfs]: lvm
Create a new LVM pool? (yes/no) [default=yes]:
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: yes
Path to the existing block device: /dev/sda
Would you like to create a new local network bridge? (yes/no) [default=yes]: yes
What should the new bridge be called? [default=incusbr0]:
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:
Would you like the server to be available over the network? (yes/no) [default=no]:
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]:
Would you like a YAML "init" preseed to be printed? (yes/no) [default=no]:
```
## Set firewall
**Check**
```
sudo firewall-cmd --list-all
sudo firewall-cmd --get-active-zones
```

**Set incus bridge to trusted zone**
```
sudo firewall-cmd --zone=trusted --change-interface=incusbr0 --permanent
sudo firewall-cmd --zone=trusted --change-interface=incusbr0
```

## Set subordinate user and group id
**Check**
```
cat /etc/subuid
cat /etc/subgid
```

**Set subordinate user and group id**
```
> sudo vi /etc/subuid
-- Add:
root:1000000:1000000000
root:1000:1

> sudo vi /etc/subgid
-- Add:
root:1000000:1000000000
root:1000:1
```

*Recommended: Reboot Server after config!*

## If using Docker in the same Machine
[How to configure your firewall - incus documentation](https://linuxcontainers.org/incus/docs/main/howto/network_bridge_firewalld/)

*As root*
**check**
`sysctl -n net.ipv4.conf.all.forwarding`

**set**
```
echo "net.ipv4.conf.all.forwarding=1" > /etc/sysctl.d/99-forwarding.conf
systemctl restart systemd-sysctl
```
## Test

**check**
```
> incus list
> incus config show
> incus storage show default
> incus network show incusbr0
```

**test**
```
> incus launch images:debian/12 first
> incus exec first -- bash

> incus launch images:debian/12 debian-vm --vm
> incus exec debian-vm -- bash
```
