---
title: Linux(Ubuntu) - mdadm을 통해 Software RAID 구성
date: 2024-03-12 09:45:00 +0900
categories: [Linux]
tags: [linux, ubuntu, mdadm, raid]
description: Ubuntu에서 mdadm의 패키지를 통해 Software RAID를 구성하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `mdadm`은 Linux에서 사용할 수 있는 Software RAID 패키지다.
* Linux kernel module인 md(modular device)를 이용해서 RAID를 구성하기 때문에 작업 시 신중함이 필요하다.
* 사용법은 대부분의 Linux에서도 동일하다.

## 설치
---

Ubuntu 사용한다면 기본적으로 설치가 되어있다.

```shell
admin@idc-dx-h100:~$ sudo apt install mdadm
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
mdadm is already the newest version (4.2-0ubuntu2).
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
```

## 구성 및 제거 방법
---

### RAID 구성

다음 예제는 RAID5를 구성하는 방법이다.
RAID5의 구조 상 disk는 3장이 필수로 필요하다.

```shell
# Disk 확인(RAID 대상은 sdb, sdc, sdd)
admin@test:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sda      8:0    0  50G  0 disk
├─sda1   8:1    0   1G  0 part /boot/efi
└─sda2   8:2    0  49G  0 part /
sdb      8:16   0  10G  0 disk
sdc      8:32   0  10G  0 disk
sdd      8:48   0  10G  0 disk
sr0     11:0    1   3G  0 rom

# RAID5로 구성
admin@test:~$ sudo mdadm --create -v /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 10476544K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

# md0 확인
admin@test:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE  MOUNTPOINTS
sda      8:0    0  50G  0 disk
├─sda1   8:1    0   1G  0 part  /boot/efi
└─sda2   8:2    0  49G  0 part  /
sdb      8:16   0  10G  0 disk
└─md0    9:0    0  20G  0 raid5
sdc      8:32   0  10G  0 disk
└─md0    9:0    0  20G  0 raid5
sdd      8:48   0  10G  0 disk
└─md0    9:0    0  20G  0 raid5
sr0     11:0    1   3G  0 rom

# RAID5 상태 확인
admin@test:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid5 sdd[3] sdc[1] sdb[0]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [=================>...]  recovery = 85.0% (8909588/10476544) finish=0.1min speed=171977K/sec

unused devices: <none>

# 파일시스템 생성
admin@test:~$ sudo mkfs.ext4 /dev/md0
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 5238272 4k blocks and 1310720 inodes
Filesystem UUID: b1fc5d16-b8e2-4c75-870c-c521763a920c
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

# /mnt에 mount 및 확인
admin@test:~$ sudo mount /dev/md0 /mnt
admin@test:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           793M  1.3M  791M   1% /run
efivarfs        256K   46K  206K  18% /sys/firmware/efi/efivars
/dev/sda2        49G  6.8G   41G  15% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/sda1      1022M  6.2M 1016M   1% /boot/efi
/dev/md0         20G   24K   19G   1% /mnt
tmpfs           793M   12K  792M   1% /run/user/1000
```

### RAID 제거

구성 할때와 반대로 진행한다.

```shell
# unmount
admin@test:~$ sudo umount /mnt

# RAID 해제
admin@test:~$ sudo mdadm --stop /dev/md0
mdadm: stopped /dev/md0

# RAID의 메타데이터 제거
admin@test:~$ sudo mdadm --zero-superblock /dev/sdb
admin@test:~$ sudo mdadm --zero-superblock /dev/sdc
admin@test:~$ sudo mdadm --zero-superblock /dev/sdd

# RAID 구성 확인(없으면 정상)
admin@test:~$ cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
unused devices: <none>

# Disk 확인
admin@test:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sda      8:0    0  50G  0 disk
├─sda1   8:1    0   1G  0 part /boot/efi
└─sda2   8:2    0  49G  0 part /
sdb      8:16   0  10G  0 disk
sdc      8:32   0  10G  0 disk
sdd      8:48   0  10G  0 disk
sr0     11:0    1   3G  0 rom
```

## Troubleshooting
---

### 1. /dev/md127 변경되는 현상

오래전부터 있었던 현상이다.  
근본적으로는 `md`이하에 hostname으로 ARRAY가 짜져서 그런 것으로 보인다.

```shell
admin@idc-dx-h100:~$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
loop0         7:0    0     4K  1 loop  /snap/bare/5
loop1         7:1    0 160.5M  1 loop  /snap/chromium/2768
loop2         7:2    0  63.4M  1 loop  /snap/core20/1974
loop3         7:3    0  63.9M  1 loop  /snap/core20/2182
loop4         7:4    0  74.2M  1 loop  /snap/core22/1122
loop5         7:5    0  66.5M  1 loop  /snap/cups/1024
loop6         7:6    0   497M  1 loop  /snap/gnome-42-2204/141
loop7         7:7    0  91.7M  1 loop  /snap/gtk-common-themes/1535
loop8         7:8    0    87M  1 loop  /snap/lxd/27037
loop9         7:9    0    87M  1 loop  /snap/lxd/27428
loop10        7:10   0  39.1M  1 loop  /snap/snapd/21184
loop11        7:11   0  40.4M  1 loop  /snap/snapd/20671
sda           8:0    0 931.5G  0 disk
sdb           8:16   0 931.5G  0 disk
nvme0n1     259:0    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme1n1     259:1    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme2n1     259:2    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme3n1     259:3    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme9n1     259:4    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme7n1     259:5    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme4n1     259:6    0   1.7T  0 disk
├─nvme4n1p1 259:10   0     1G  0 part
└─nvme4n1p2 259:11   0   1.7T  0 part
  └─md0       9:0    0   1.7T  0 raid1
    └─md0p1 259:14   0   1.7T  0 part  /
nvme6n1     259:7    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data
nvme5n1     259:8    0   1.7T  0 disk
├─nvme5n1p1 259:12   0     1G  0 part  /boot/efi
└─nvme5n1p2 259:13   0   1.7T  0 part
  └─md0       9:0    0   1.7T  0 raid1
    └─md0p1 259:14   0   1.7T  0 part  /
nvme8n1     259:9    0   3.5T  0 disk
└─md127       9:127  0  24.5T  0 raid5 /data

admin@idc-dx-h100:~$ sudo mdadm --detail --scan
ARRAY /dev/md/idc-ai-h100-gpu1:1 metadata=1.2 name=idc-ai-h100-gpu1:1 UUID=cdee4e56:092b4529:f2c9eff8:2ea9c5d6
ARRAY /dev/md0 metadata=1.2 name=ubuntu-server:0 UUID=41e9d2f4:8ca4283a:fa4e58fd:00931269
```

해당 현상은 kernel에 update를 해줘서 처리가 가능하다.

```shell
# /etc/mdadm/mdadm.conf에 해당 부분 추가
admin@idc-dx-h100:~$ sudo vi /etc/mdadm/mdadm.conf
ARRAY /dev/md0 metadata=1.2 name=ubuntu-server:0 UUID=41e9d2f4:8ca4283a:fa4e58fd:00931269
# 추가 후 /dev/md/idc-ai-h100-gpu1:1 -> md1로 수정
ARRAY /dev/md1 metadata=1.2 name=idc-ai-h100-gpu1:1 UUID=cdee4e56:092b4529:f2c9eff8:2ea9c5d6
MAILADDR root

# ramfs에 등록
admin@idc-dx-h100:~$ sudo update-initramfs -u
update-initramfs: Generating /boot/initrd.img-5.15.0-97-generic
W: Possible missing firmware /lib/firmware/ast_dp501_fw.bin for module ast
```

이후 rebooting하면 정상적으로 확인 된다.