# ETHIO-Telekom-Wiki

Related Product: VPS Service

## Issue - Domain resolution and outbound connectivity is not working
### Indication
```
$ ping www.google.com                               
ping: unknown host www.google.com
```

### Solution
[https://askubuntu.com/questions/1012641/dns-set-to-systemds-127-0-0-53-how-to-change-permanently/1083843#1083843]
```
$ cat /etc/systemd/resolved.conf
<...>
[Resolve]
DNS=8.8.8.8 8.8.4.4
<...>
```

## Issue - Assigned disc space is not mounted and the root partition is very small (3.9G)
### Indication
There is only one mapped filesystem which shows 3.9G of available space
```
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  3.9G  3.3G  600M  84% /
/dev/sda2                          976M  150M  759M  17% /boot
/dev/sda4                           79G  9.1G   66G  13% /opt
tmpfs                              798M     0  798M   0% /run/user/1001
```
Lsbld shows more available space. Only 19G of are mounted but 100G available. the 19G for the lvm drive are only used with 3.9G
```
$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 99.2M  1 loop /snap/core/11167
loop1                       7:1    0 99.4M  1 loop /snap/core/11187
sda                         8:0    0  100G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
├─sda3                      8:3    0   19G  0 part
│ └─ubuntu--vg-ubuntu--lv 253:0    0  3.9G  0 lvm  /

```
### Solution
Run parted to fix issues with the partition table
```
sudo parted -l
--> Let it fix the found issue
```

Increase the size of the lvm partition (from 3.9G to 19G)
https://askubuntu.com/questions/1106795/ubuntu-server-18-04-lvm-out-of-space-with-improper-default-partitioning
```
$ lvm
lvm> lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
lvm> exit

$ resize2fs /dev/ubuntu-vg/ubuntu-lv
```

Create a new partition from the free disk space and mount it

```
$ sudo fdisk /dev/sda
n (Create New Partition)
default (partition number)
default (starrting sector)
default (size - all remaining space)

w (Write on Disk
Ctrl+C to exit
```
Check the assigned name of the new partition. In this case sda4
```
$ sudo fdisk -l
$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 99.2M  1 loop /snap/core/11167
loop1                       7:1    0 99.4M  1 loop /snap/core/11187
sda                         8:0    0  100G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
├─sda3                      8:3    0   19G  0 part
│ └─ubuntu--vg-ubuntu--lv 253:0    0   19G  0 lvm  /
└─sda4                      8:4    0   80G  0 part 
```


Make a file system on the new partition
```
mkfs -t ext4 /dev/sda4
```

Auto Mount the filesystem
https://www.linuxbabe.com/desktop-linux/how-to-automount-file-systems-on-linux

```
$sudo blkid
Find the arrording UUID
```

Create a Mountpoint
```
sudo mkdir /opt
```

Edit /etc/fstab File using the identified UUID
```
sudo nano /etc/fstab
```
Add a line with the following content (change UUID)
```
UUID=eb67c479-962f-4bcc-b3fe-cefaf908f01e  /opt  ext4  defaults  0  2
```

Run Automount
```
sudo mount -a
```

