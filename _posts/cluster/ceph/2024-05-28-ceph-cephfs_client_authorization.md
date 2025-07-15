---
title: Ceph - CephFS Client 권한 부여
date: 2024-05-28 17:43:00 +0900
categories: [Cluster, Ceph]
tags: [cluster, ceph, cephadm]
description: Ceph의 CephFS mount 시 client에서 권한을 받는 방법이다.
---

>v18.2.2(reef) / Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요 및 요구 사항
---

* Ceph에서 mount 시 보통 `admin.keyring`을 통해 mount을 한다.  
  `admin.kering`은 `mds`/`mgr`/`mon`/`osd` 전부 `allow *`의 상태로 허용 하기 때문에 보안적인 위험성이 있다.  
  물론 내부에서만 사용한다면 큰 문제는 없지만, Cloud를 사용하거나 외부에서 mount 할 일이 있으면 보안에 대해 신경 쓰는 편이 좋다.
* cephfs에 사용할 사용자 권한을 부여 하여서 keyring을 만들어 client에 사용하면 이 문제를 해결할 수 있다.

## 사용자 kerying 만들고 server에 적용
---

`ceph`명령어를 통하여 client의 kerying을 만들어 준 뒤`tee` 명령어로 해당 파일을 파일화 시킨 후 해당 파일을 `/etc/ceph/ceph.keyring`에 넣어서 server에 등록을 시켜준다.  
등록 시 권한도 추가적으로 등록이 가능하다.

- <https://docs.ceph.com/en/latest/cephfs/mount-prerequisites/>
- <https://docs.ceph.com/en/latest/cephfs/client-auth/>

```shell
# ceph or cephadm shell (server)
root@gbia-ceph-storage1:~$ ceph fs authorize cephfs client.deepadmin / rw | tee ./ceph.client.deepadmin.keyring
[client.deepadmin]
        key = AQCjJwVm4qsGAhAAVnD9fhjrZgs+LQxbP91frA==
        
root@gbia-ceph-storage1:~$ vi /etc/ceph/ceph.keyring
[client.admin]
        key = AQBCBgVmp/CoJRAATgFjdA8YYqr1zJGs/+9GKQ==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
# 해당 부분에 key 추가 한 뒤 권한을 추가로 입력 함
[client.deepadmin]
        key = AQCjJwVm4qsGAhAAVnD9fhjrZgs+LQxbP91frA==
        caps mds = "allow rw fsname=cephfs"
        caps mon = "allow r fsname=cephfs"
        caps osd = "allow rw tag cephfs data=cephfs"
```

## Client에서 사용자 keyring으로 CephFS mount
---

keyring의 부분 중 권한 부분은 server에서만 있으면 되고, mount에서는 key값만 있으면 되어서 해당 부분을 client에서 추가하면 된다.

```shell
# /etc/ceph/ceph.kerying (client)
deepadmin@gbia-tmp-cpu1:~$ sudo vi /etc/ceph/ceph.keyring
[client.deepadmin]
        key = AQCjJwVm4qsGAhAAVnD9fhjrZgs+LQxbP91frA==
```

CephFS를 mount 할 때 여러가지 방식이 있긴 하지만 official page에서 제공 해주는 방식을 사용하는 방법이 좋다.

- <https://docs.ceph.com/en/latest/cephfs/mount-prerequisites/>
- <https://docs.ceph.com/en/latest/cephfs/mount-using-kernel-driver/>

```shell
# /etc/ceph/ceph.conf (server)
deepadmin@gbia-ceph-storage1:~$ cat /etc/ceph/ceph.conf
# minimal ceph.conf for b0372332-ecc7-11ee-86d1-b75ac1030ed7
[global]
        fsid = b0372332-ecc7-11ee-86d1-b75ac1030ed7
        mon_host = [v2:10.100.100.1:3300/0,v1:10.100.100.1:6789/0] [v2:10.100.100.2:3300/0,v1:10.100.100.2:6789/0] [v2:10.100.100.3:3300/0,v1:10.100.100.3:6789/0]
```

우선 위의 내용은 server의 `/etc/ceph/ceph.conf`의 내용이다.  
이 내용을 client에 `/etc/ceph/ceph.conf`에 넣어주면 된다.  
이렇게 하면 fstab에 등록 시 kernel driver에서 conf를 참조하기 때문에 굳이 IP를 입력하지 않아도 되는 장점이 있다.  
이후 fstab에 등록하여 mount를 해주면 된다.

```shell
# /etc/fstab (client)
deepadmin@gbia-tmp-cpu1:/$ vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/md0p1 during curtin installation
/dev/disk/by-id/md-uuid-c5dac3bd:1334d2e1:ae5e3b5a:01f36b0c-part1 / ext4 defaults 0 1
# /boot/efi was on /dev/sdb1 during curtin installation
/dev/disk/by-uuid/8D2D-77E5 /boot/efi vfat defaults 0 1
/swap.img       none    swap    sw      0       0

# 해당 부분 추가
# ceph mount
deepadmin@.cephfs=/ /data ceph noatime,_netdev,nofail 0 0

# mount
deepadmin@gbia-tmp-cpu1:/$ sudo mount -a

# 확인
deepadmin@gbia-tmp-cpu1:/$ df -h
Filesystem                                               Size  Used Avail Use% Mounted on
tmpfs                                                    101G  2.4M  101G   1% /run
/dev/md0p1                                               879G   13G  821G   2% /
tmpfs                                                    504G     0  504G   0% /dev/shm
tmpfs                                                    5.0M     0  5.0M   0% /run/lock
/dev/sdb1                                               1022M  6.1M 1016M   1% /boot/efi
tmpfs                                                    101G  4.0K  101G   1% /run/user/1000
10.100.100.1:6789,10.100.100.2:6789,10.100.100.3:6789:/   26T     0   26T   0% /data
```