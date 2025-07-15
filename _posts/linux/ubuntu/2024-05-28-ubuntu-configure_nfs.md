---
title: Ubuntu - NFS(Network File System) 구성
date: 2024-05-28 09:52:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, nfs]
description: Ubuntu에서 mdadm의 패키지를 통해 Software RAID를 구성하는 방법이다.
---

>Ubuntu 24.04 LTS
{: .prompt-oinfo}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* NFS(Network File System)의 Server & Client 설치 뿐만이 아닌 필요한 명령어나 추가 구성에 대해 알아본다.
* Redhat 계열이나 Debian 계열이나 동일하게 패키지가 존재하여 설치나 구성이 비교적 간단하다.
* NFS는 RPC(Remote Procedure Call)를 통하여 port 관리가 이뤄진다.
* NFS는 기본적으로 TCP를 사용하지만, version 3에서는 UDP도 구성이 가능하다.

## NFS Server 설치
---

Server의 설치는 패키지가 있어 한 줄로 간단히 설치가 가능하다.

```shell
# 설치
admin@ubuntu-2404-test:~$ sudo apt install nfs-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'nfs-kernel-server' instead of 'nfs-server'
The following additional packages will be installed:
  keyutils libnfsidmap1 nfs-common rpcbind
Suggested packages:
  watchdog
The following NEW packages will be installed:
  keyutils libnfsidmap1 nfs-common nfs-kernel-server rpcbind
0 upgraded, 5 newly installed, 0 to remove and 8 not upgraded.
Need to get 569 kB of archives.
After this operation, 2,022 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://mirror.kakao.com/ubuntu noble/main amd64 libnfsidmap1 amd64 1:2.6.4-3ubuntu5 [48.2 kB]
Get:2 http://mirror.kakao.com/ubuntu noble/main amd64 rpcbind amd64 1.2.6-7ubuntu2 [46.5 kB]
Get:3 http://mirror.kakao.com/ubuntu noble/main amd64 keyutils amd64 1.6.3-3build1 [56.8 kB]
Get:4 http://mirror.kakao.com/ubuntu noble/main amd64 nfs-common amd64 1:2.6.4-3ubuntu5 [248 kB]
Get:5 http://mirror.kakao.com/ubuntu noble/main amd64 nfs-kernel-server amd64 1:2.6.4-3ubuntu5 [169 kB]
Fetched 569 kB in 0s (2,193 kB/s)
Selecting previously unselected package libnfsidmap1:amd64.
(Reading database ... 85808 files and directories currently installed.)
Preparing to unpack .../libnfsidmap1_1%3a2.6.4-3ubuntu5_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.4-3ubuntu5) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../rpcbind_1.2.6-7ubuntu2_amd64.deb ...
Unpacking rpcbind (1.2.6-7ubuntu2) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.3-3build1_amd64.deb ...
Unpacking keyutils (1.6.3-3build1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.4-3ubuntu5_amd64.deb ...
Unpacking nfs-common (1:2.6.4-3ubuntu5) ...
Selecting previously unselected package nfs-kernel-server.
Preparing to unpack .../nfs-kernel-server_1%3a2.6.4-3ubuntu5_amd64.deb ...
Unpacking nfs-kernel-server (1:2.6.4-3ubuntu5) ...
Setting up libnfsidmap1:amd64 (1:2.6.4-3ubuntu5) ...
Setting up rpcbind (1.2.6-7ubuntu2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /usr/lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /usr/lib/systemd/system/rpcbind.socket.
Setting up keyutils (1.6.3-3build1) ...
Setting up nfs-common (1:2.6.4-3ubuntu5) ...

Creating config file /etc/idmapd.conf with new version

Creating config file /etc/nfs.conf with new version
info: Selecting UID from range 100 to 999 ...

info: Adding system user `statd' (UID 111) ...
info: Adding new user `statd' (UID 111) with group `nogroup' ...
info: Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
auth-rpcgss-module.service is a disabled or a static unit, not starting it.
nfs-idmapd.service is a disabled or a static unit, not starting it.
nfs-utils.service is a disabled or a static unit, not starting it.
proc-fs-nfsd.mount is a disabled or a static unit, not starting it.
rpc-gssd.service is a disabled or a static unit, not starting it.
rpc-statd-notify.service is a disabled or a static unit, not starting it.
rpc-statd.service is a disabled or a static unit, not starting it.
rpc-svcgssd.service is a disabled or a static unit, not starting it.
Setting up nfs-kernel-server (1:2.6.4-3ubuntu5) ...
Created symlink /etc/systemd/system/nfs-mountd.service.requires/fsidd.service → /usr/lib/systemd/system/fsidd.service.
Created symlink /etc/systemd/system/nfs-server.service.requires/fsidd.service → /usr/lib/systemd/system/fsidd.service.
Created symlink /etc/systemd/system/nfs-client.target.wants/nfs-blkmap.service → /usr/lib/systemd/system/nfs-blkmap.service.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
nfs-mountd.service is a disabled or a static unit, not starting it.
nfsdcld.service is a disabled or a static unit, not starting it.

Creating config file /etc/exports with new version

Creating config file /etc/default/nfs-kernel-server with new version
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

# 서비스 확인

admin@ubuntu-2404-test:/$ sudo systemctl status nfs-server.service
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: enabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Tue 2024-05-28 01:18:38 UTC; 40min ago
   Main PID: 2082 (code=exited, status=0/SUCCESS)
        CPU: 15ms

May 28 01:18:38 ubuntu-2404-test systemd[1]: Starting nfs-server.service - NFS server and services...
May 28 01:18:38 ubuntu-2404-test exportfs[2080]: exportfs: can't open /etc/exports for reading
May 28 01:18:38 ubuntu-2404-test systemd[1]: Finished nfs-server.service - NFS server and services.
```

정상 설치가 되었으면 NFS 서버에서 사용되는 version 및 사용하는 rpc port에 대해서 확인해본다.

```shell
# Ubuntu 24.04에서 기본적으로 지원되는 version
admin@ubuntu-2404-test:~$ sudo cat /proc/fs/nfsd/versions
+3 +4 +4.1 +4.2

# rpc port
admin@ubuntu-2404-test:~$ sudo rpcinfo -p
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  37185  status
    100024    1   tcp  44397  status
    100005    1   udp  45089  mountd
    100005    1   tcp  57739  mountd
    100005    2   udp  42514  mountd
    100005    2   tcp  48631  mountd
    100005    3   udp  57191  mountd
    100005    3   tcp  52697  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100021    1   udp  56069  nlockmgr
    100021    3   udp  56069  nlockmgr
    100021    4   udp  56069  nlockmgr
    100021    1   tcp  46015  nlockmgr
    100021    3   tcp  46015  nlockmgr
    100021    4   tcp  46015  nlockmgr
```

NFS Server에 사용할 테스트 디렉토리도 만들어준다.

```shell
admin@ubuntu-2404-test:~$ sudo mkdir /nfs-share
```

`/etc/exports`는 NFS에서 사용하는 mount point의 정보를 담고 있다.  
이 곳에 테스트로 생성한 디렉토리의 정보를 담아주고, option을 추가 해준다.  
Option에 대해서는 다음의 링크를 참조하면 된다.  
<https://manpages.debian.org/unstable/nfs-kernel-server/exports.5.en.html>

```shell
admin@ubuntu-2404-test:/$ sudo vi /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
## 추가
/nfs-share *(rw,no_subtree_check)
```

`/etc/exports`에 등록되었으면 `exportfs`의 명령어를 통해 구성을 완료한다.

```shell
# /etc/exports의 내용을 reload 한 뒤 설정
admin@ubuntu-2404-test:/$ sudo exportfs -ar

# 확인
admin@ubuntu-2404-test:/$ sudo exportfs -v
/nfs-share      <world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

## NFS Client 설치
---

Client 또한 쉽게 설치가 가능하다.

```shell
admin@ubuntu-24:~$ sudo apt install nfs-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libevent-core-2.1-7t64 rpcbind
Suggested packages:
  open-iscsi watchdog
The following NEW packages will be installed:
  keyutils libevent-core-2.1-7t64 nfs-common rpcbind
0 upgraded, 4 newly installed, 0 to remove and 5 not upgraded.
Need to get 443 kB of archives.
After this operation, 1,497 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://archive.ubuntu.com/ubuntu noble/main amd64 libevent-core-2.1-7t64 amd64 2.1.12-stable-9ubuntu2 [91.3 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/main amd64 rpcbind amd64 1.2.6-7ubuntu2 [46.5 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble/main amd64 keyutils amd64 1.6.3-3build1 [56.8 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble/main amd64 nfs-common amd64 1:2.6.4-3ubuntu5 [248 kB]
Fetched 443 kB in 3s (168 kB/s)
Selecting previously unselected package libevent-core-2.1-7t64:amd64.
(Reading database ... 177336 files and directories currently installed.)
Preparing to unpack .../libevent-core-2.1-7t64_2.1.12-stable-9ubuntu2_amd64.deb ...
Unpacking libevent-core-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../rpcbind_1.2.6-7ubuntu2_amd64.deb ...
Unpacking rpcbind (1.2.6-7ubuntu2) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.3-3build1_amd64.deb ...
Unpacking keyutils (1.6.3-3build1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a2.6.4-3ubuntu5_amd64.deb ...
Unpacking nfs-common (1:2.6.4-3ubuntu5) ...
Setting up rpcbind (1.2.6-7ubuntu2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /usr/lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /usr/lib/systemd/system/rpcbind.socket.
Setting up keyutils (1.6.3-3build1) ...
Setting up libevent-core-2.1-7t64:amd64 (2.1.12-stable-9ubuntu2) ...
Setting up nfs-common (1:2.6.4-3ubuntu5) ...

Creating config file /etc/idmapd.conf with new version

Creating config file /etc/nfs.conf with new version
info: Selecting UID from range 100 to 999 ...

info: Adding system user `statd' (UID 124) ...
info: Adding new user `statd' (UID 124) with group `nogroup' ...
info: Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /usr/lib/systemd/system/nfs-client.target.
auth-rpcgss-module.service is a disabled or a static unit, not starting it.
nfs-idmapd.service is a disabled or a static unit, not starting it.
nfs-utils.service is a disabled or a static unit, not starting it.
proc-fs-nfsd.mount is a disabled or a static unit, not starting it.
rpc-gssd.service is a disabled or a static unit, not starting it.
rpc-statd-notify.service is a disabled or a static unit, not starting it.
rpc-statd.service is a disabled or a static unit, not starting it.
rpc-svcgssd.service is a disabled or a static unit, not starting it.
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.1) ...
```

Client에서 mount 할 디렉토리를 생성하여 준 뒤 `mount` 명령어를 통하여 간단히 mount 한다.

```shell
# Client에서 사용 할 디렉토리 생성
admin@ubuntu-24:~$ sudo mkdir /data

# Mount 후 확인
admin@ubuntu-24:~$ sudo mount 10.50.1.210:/nfs-share /data
admin@ubuntu-24:~$ mount | grep 10.50.1.210
10.50.1.210:/nfs-share on /data type nfs4 (rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.50.1.211,local_lock=none,addr=10.50.1.210)
admin@ubuntu-24:~$ df -h
Filesystem                                        Size  Used Avail Use% Mounted on
tmpfs                                             1.6G  1.8M  1.6G   1% /run
rpool/ROOT/ubuntu_49vzqf                          115G  3.1G  112G   3% /
rpool/ROOT/ubuntu_49vzqf/srv                      112G  128K  112G   1% /srv
rpool/ROOT/ubuntu_49vzqf/usr/local                112G  128K  112G   1% /usr/local
rpool/ROOT/ubuntu_49vzqf/var/games                112G  128K  112G   1% /var/games
rpool/ROOT/ubuntu_49vzqf/var/lib                  114G  1.3G  112G   2% /var/lib
rpool/ROOT/ubuntu_49vzqf/var/lib/AccountsService  112G  128K  112G   1% /var/lib/AccountsService
rpool/ROOT/ubuntu_49vzqf/var/lib/NetworkManager   112G  128K  112G   1% /var/lib/NetworkManager
rpool/ROOT/ubuntu_49vzqf/var/lib/apt              112G   88M  112G   1% /var/lib/apt
rpool/ROOT/ubuntu_49vzqf/var/lib/dpkg             112G   35M  112G   1% /var/lib/dpkg
rpool/ROOT/ubuntu_49vzqf/var/log                  112G   21M  112G   1% /var/log
rpool/ROOT/ubuntu_49vzqf/var/mail                 112G  128K  112G   1% /var/mail
rpool/ROOT/ubuntu_49vzqf/var/snap                 112G  1.8M  112G   1% /var/snap
rpool/ROOT/ubuntu_49vzqf/var/spool                112G  128K  112G   1% /var/spool
rpool/ROOT/ubuntu_49vzqf/var/www                  112G  128K  112G   1% /var/www
tmpfs                                             7.8G     0  7.8G   0% /dev/shm
tmpfs                                             5.0M  8.0K  5.0M   1% /run/lock
efivarfs                                          256K  111K  141K  45% /sys/firmware/efi/efivars
rpool/USERDATA/home_a9w2yl                        112G  7.4M  112G   1% /home
rpool/USERDATA/root_a9w2yl                        112G  256K  112G   1% /root
bpool/BOOT/ubuntu_49vzqf                          1.8G   96M  1.7G   6% /boot
/dev/sda1                                         1.1G  6.2M  1.1G   1% /boot/efi
tmpfs                                             1.6G  124K  1.6G   1% /run/user/1000
10.50.1.210:/nfs-share                             31G  7.2G   23G  24% /data
```

기본적으로 mount 할 시 가장 높은 버전이 우선 순위이다.  
또한 `df`의 명령어를 통하여 해당 디렉토리에 대한 크기도 확인이 가능하다.  
만약 이러한 과정을 persistence 적으로 사용할 경우 `/etc/fstab`에 등록하여 사용하면 된다.

```shell
admin@ubuntu-24:~$ sudo vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/disk/by-uuid/b905bab4-618c-467e-bbfd-808d705adff8 none swap sw 0 0
# Use `zfs list` for current zfs mount info
# bpool none defaults 0 0
# Use `zfs list` for current zfs mount info
# rpool none defaults 0 0
# Use `zfs list` for current zfs mount info
# rpool / defaults 0 0
# Use `zfs list` for current zfs mount info
# rpool none defaults 0 0
# Use `zfs list` for current zfs mount info
# rpool /root defaults 0 0
# Use `zfs list` for current zfs mount info
# rpool /home defaults 0 0
# Use `zfs list` for current zfs mount info
# bpool /boot defaults 0 0
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/8C99-169C /boot/efi vfat defaults 0 1

# 추가
10.50.1.210:/nfs-share /data nfs noatime,nodiratime,nofail 0 0
```

등록 후 명령어를 통해 처리한다.

```shell
# Ubuntu 24.04에서 zfs를 지원하다 보니 zfs를 사용시에는 systemd를 통해 deamon-reload 과정이 필요 함
admin@ubuntu-24:~$ sudo systemctl daemon-reload
admin@ubuntu-24:~$ sudo mount -a
admin@ubuntu-24:~$ mount | grep 10.50.1.210
10.50.1.210:/nfs-share on /data type nfs4 (rw,noatime,nodiratime,vers=4.2,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.50.1.211,local_lock=none,addr=10.50.1.210)
```

## 추가 명령어
---

NFS의 사용에 도움이 되는 명령어이다.

### 1. showmount

Server에서 client가 어디서 mount를 하였는지 확인할 수 있는 명령어이다.

```shell
admin@ubuntu-2404-test:$ sudo showmount -a
All mount points on ubuntu-2404-test:
10.50.1.211:/nfs-share
```