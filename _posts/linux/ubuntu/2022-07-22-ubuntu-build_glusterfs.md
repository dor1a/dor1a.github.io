---
title: Ubuntu - glusterfs 구축
date: 2022-07-22 10:43:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, glusterfs]
description: Ubuntu에서 glusterfs를 구축 해본다.
---

>glusterfs 10 / Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 분산 스토리지 기반이지만, 다루기가 쉬우며 관리에도 용이하다.
* 단점으로는 속도가 많이 느리다.
* 대부분의 File system을 지원한다.

## 기본 설치
---

```shell
# apt update
admin@test:~$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Hit:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease

# repository 추가
admin@test:~$ sudo add-apt-repository ppa:gluster/glusterfs-10
PPA publishes dbgsym, you may need to include 'main/debug' component
Repository: 'deb https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu/ jammy main'
Description:
GlusterFS 10
More info: https://launchpad.net/~gluster/+archive/ubuntu/glusterfs-10
Adding repository.
Press [ENTER] to continue or Ctrl-c to cancel.
Adding deb entry to /etc/apt/sources.list.d/gluster-ubuntu-glusterfs-10-jammy.list
Adding disabled deb-src entry to /etc/apt/sources.list.d/gluster-ubuntu-glusterfs-10-jammy.list
Adding key to /etc/apt/trusted.gpg.d/gluster-ubuntu-glusterfs-10.gpg with fingerprint F7C73FCC930AC9F83B387A5613E01B7B3FE869A9
Hit:1 http://security.ubuntu.com/ubuntu jammy-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Get:4 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy InRelease [24.3 kB]
Get:5 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 Packages [2,280 B]
Get:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [127 kB]
Get:7 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main Translation-en [1,344 B]
Fetched 283 kB in 2s (120 kB/s)
Reading package lists... Done

# 설치
admin@test:~$ sudo apt install glusterfs-server -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  attr glusterfs-client glusterfs-common keyutils libgfapi0 libgfchangelog0 libgfrpc0 libgfxdr0 libglusterd0 libglusterfs0 libgoogle-perftools-dev
  libgoogle-perftools4 liblzma-dev libnfsidmap1 libtcmalloc-minimal4 libunwind-dev liburing2 nfs-common python3-prettytable python3-wcwidth rpcbind
Suggested packages:
  liblzma-doc watchdog
The following NEW packages will be installed:
  attr glusterfs-client glusterfs-common glusterfs-server keyutils libgfapi0 libgfchangelog0 libgfrpc0 libgfxdr0 libglusterd0 libglusterfs0
  libgoogle-perftools-dev libgoogle-perftools4 liblzma-dev libnfsidmap1 libtcmalloc-minimal4 libunwind-dev liburing2 nfs-common python3-prettytable
  python3-wcwidth rpcbind
0 upgraded, 22 newly installed, 0 to remove and 0 not upgraded.
Need to get 6,855 kB of archives.
After this operation, 30.9 MB of additional disk space will be used.
0% [Working]
Get:1 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 libnfsidmap1 amd64 1:2.6.1-1ubuntu1.2 [42.9 kB]
Get:2 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 libgfxdr0 amd64 10.5-ubuntu1~jammy1 [30.5 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy/main amd64 rpcbind amd64 1.2.6-2build1 [46.6 kB]
Get:4 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 libglusterfs0 amd64 10.5-ubuntu1~jammy1 [299 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy/main amd64 keyutils amd64 1.6.1-2ubuntu3 [50.4 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 nfs-common amd64 1:2.6.1-1ubuntu1.2 [241 kB]
Get:7 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 libgfrpc0 amd64 10.5-ubuntu1~jammy1 [56.8 kB]
Get:8 http://archive.ubuntu.com/ubuntu jammy/main amd64 attr amd64 1:2.5.1-1build1 [22.6 kB]
Get:9 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 libgfapi0 amd64 10.5-ubuntu1~jammy1 [87.3 kB]
Get:10 http://archive.ubuntu.com/ubuntu jammy/main amd64 libtcmalloc-minimal4 amd64 2.9.1-0ubuntu3 [98.2 kB]
Get:11 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 libgfchangelog0 amd64 10.5-ubuntu1~jammy1 [37.1 kB]
Get:12 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 libglusterd0 amd64 10.5-ubuntu1~jammy1 [14.2 kB]
Get:13 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 glusterfs-common amd64 10.5-ubuntu1~jammy1 [2,785 kB]
Get:14 http://archive.ubuntu.com/ubuntu jammy/main amd64 liburing2 amd64 2.1-2build1 [10.3 kB]
Get:15 http://archive.ubuntu.com/ubuntu jammy/main amd64 python3-wcwidth all 0.2.5+dfsg1-1 [21.9 kB]
Get:16 http://archive.ubuntu.com/ubuntu jammy/main amd64 python3-prettytable all 2.5.0-2 [31.3 kB]
Get:17 http://archive.ubuntu.com/ubuntu jammy/main amd64 libgoogle-perftools4 amd64 2.9.1-0ubuntu3 [212 kB]
Get:18 http://archive.ubuntu.com/ubuntu jammy/main amd64 liblzma-dev amd64 5.2.5-2ubuntu1 [159 kB]
Get:19 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 libunwind-dev amd64 1.3.2-2build2.1 [1,883 kB]
Get:20 http://archive.ubuntu.com/ubuntu jammy/main amd64 libgoogle-perftools-dev amd64 2.9.1-0ubuntu3 [470 kB]
Get:21 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 glusterfs-client amd64 10.5-ubuntu1~jammy1 [36.0 kB]
Get:22 https://ppa.launchpadcontent.net/gluster/glusterfs-10/ubuntu jammy/main amd64 glusterfs-server amd64 10.5-ubuntu1~jammy1 [221 kB]
Fetched 6,855 kB in 16s (439 kB/s)
Selecting previously unselected package libnfsidmap1:amd64.
(Reading database ... 81129 files and directories currently installed.)
Preparing to unpack .../00-libnfsidmap1_1%3a2.6.1-1ubuntu1.2_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.1-1ubuntu1.2) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../01-rpcbind_1.2.6-2build1_amd64.deb ...
Unpacking rpcbind (1.2.6-2build1) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../02-keyutils_1.6.1-2ubuntu3_amd64.deb ...
Unpacking keyutils (1.6.1-2ubuntu3) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../03-nfs-common_1%3a2.6.1-1ubuntu1.2_amd64.deb ...
Unpacking nfs-common (1:2.6.1-1ubuntu1.2) ...
Selecting previously unselected package attr.
Preparing to unpack .../04-attr_1%3a2.5.1-1build1_amd64.deb ...
Unpacking attr (1:2.5.1-1build1) ...
Selecting previously unselected package libtcmalloc-minimal4:amd64.
Preparing to unpack .../05-libtcmalloc-minimal4_2.9.1-0ubuntu3_amd64.deb ...
Unpacking libtcmalloc-minimal4:amd64 (2.9.1-0ubuntu3) ...
Selecting previously unselected package libgfxdr0:amd64.
Preparing to unpack .../06-libgfxdr0_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking libgfxdr0:amd64 (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package libglusterfs0:amd64.
Preparing to unpack .../07-libglusterfs0_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking libglusterfs0:amd64 (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package libgfrpc0:amd64.
Preparing to unpack .../08-libgfrpc0_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking libgfrpc0:amd64 (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package libgfapi0:amd64.
Preparing to unpack .../09-libgfapi0_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking libgfapi0:amd64 (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package libgfchangelog0:amd64.
Preparing to unpack .../10-libgfchangelog0_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking libgfchangelog0:amd64 (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package libglusterd0:amd64.
Preparing to unpack .../11-libglusterd0_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking libglusterd0:amd64 (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package liburing2:amd64.
Preparing to unpack .../12-liburing2_2.1-2build1_amd64.deb ...
Unpacking liburing2:amd64 (2.1-2build1) ...
Selecting previously unselected package python3-wcwidth.
Preparing to unpack .../13-python3-wcwidth_0.2.5+dfsg1-1_all.deb ...
Unpacking python3-wcwidth (0.2.5+dfsg1-1) ...
Selecting previously unselected package python3-prettytable.
Preparing to unpack .../14-python3-prettytable_2.5.0-2_all.deb ...
Unpacking python3-prettytable (2.5.0-2) ...
Selecting previously unselected package libgoogle-perftools4:amd64.
Preparing to unpack .../15-libgoogle-perftools4_2.9.1-0ubuntu3_amd64.deb ...
Unpacking libgoogle-perftools4:amd64 (2.9.1-0ubuntu3) ...
Selecting previously unselected package liblzma-dev:amd64.
Preparing to unpack .../16-liblzma-dev_5.2.5-2ubuntu1_amd64.deb ...
Unpacking liblzma-dev:amd64 (5.2.5-2ubuntu1) ...
Selecting previously unselected package libunwind-dev:amd64.
Preparing to unpack .../17-libunwind-dev_1.3.2-2build2.1_amd64.deb ...
Unpacking libunwind-dev:amd64 (1.3.2-2build2.1) ...
Selecting previously unselected package libgoogle-perftools-dev:amd64.
Preparing to unpack .../18-libgoogle-perftools-dev_2.9.1-0ubuntu3_amd64.deb ...
Unpacking libgoogle-perftools-dev:amd64 (2.9.1-0ubuntu3) ...
Selecting previously unselected package glusterfs-common.
Preparing to unpack .../19-glusterfs-common_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking glusterfs-common (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package glusterfs-client.
Preparing to unpack .../20-glusterfs-client_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking glusterfs-client (10.5-ubuntu1~jammy1) ...
Selecting previously unselected package glusterfs-server.
Preparing to unpack .../21-glusterfs-server_10.5-ubuntu1~jammy1_amd64.deb ...
Unpacking glusterfs-server (10.5-ubuntu1~jammy1) ...
Setting up libnfsidmap1:amd64 (1:2.6.1-1ubuntu1.2) ...
Setting up libglusterd0:amd64 (10.5-ubuntu1~jammy1) ...
Setting up attr (1:2.5.1-1build1) ...
Setting up libtcmalloc-minimal4:amd64 (2.9.1-0ubuntu3) ...
Setting up rpcbind (1.2.6-2build1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /lib/systemd/system/rpcbind.socket.
Setting up libglusterfs0:amd64 (10.5-ubuntu1~jammy1) ...
Setting up python3-wcwidth (0.2.5+dfsg1-1) ...
Setting up liblzma-dev:amd64 (5.2.5-2ubuntu1) ...
Setting up keyutils (1.6.1-2ubuntu3) ...
Setting up liburing2:amd64 (2.1-2build1) ...
Setting up python3-prettytable (2.5.0-2) ...
Setting up libgoogle-perftools4:amd64 (2.9.1-0ubuntu3) ...
Setting up libgfxdr0:amd64 (10.5-ubuntu1~jammy1) ...
Setting up libunwind-dev:amd64 (1.3.2-2build2.1) ...
Setting up libgoogle-perftools-dev:amd64 (2.9.1-0ubuntu3) ...
Setting up nfs-common (1:2.6.1-1ubuntu1.2) ...

Creating config file /etc/idmapd.conf with new version

Creating config file /etc/nfs.conf with new version
Adding system user `statd' (UID 115) ...
Adding new user `statd' (UID 115) with group `nogroup' ...
Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
auth-rpcgss-module.service is a disabled or a static unit, not starting it.
nfs-idmapd.service is a disabled or a static unit, not starting it.
nfs-utils.service is a disabled or a static unit, not starting it.
proc-fs-nfsd.mount is a disabled or a static unit, not starting it.
rpc-gssd.service is a disabled or a static unit, not starting it.
rpc-statd-notify.service is a disabled or a static unit, not starting it.
rpc-statd.service is a disabled or a static unit, not starting it.
rpc-svcgssd.service is a disabled or a static unit, not starting it.
rpc_pipefs.target is a disabled or a static unit, not starting it.
var-lib-nfs-rpc_pipefs.mount is a disabled or a static unit, not starting it.
Setting up libgfrpc0:amd64 (10.5-ubuntu1~jammy1) ...
Setting up libgfchangelog0:amd64 (10.5-ubuntu1~jammy1) ...
Setting up libgfapi0:amd64 (10.5-ubuntu1~jammy1) ...
Setting up glusterfs-common (10.5-ubuntu1~jammy1) ...
Adding group `gluster' (GID 119) ...
Done.
Setting up glusterfs-client (10.5-ubuntu1~jammy1) ...
Setting up glusterfs-server (10.5-ubuntu1~jammy1) ...
gluster-ta-volume.service is a disabled or a static unit, not starting it.
glusterd.service is a disabled or a static unit, not starting it.
glustereventsd.service is a disabled or a static unit, not starting it.
glusterfssharedstorage.service is a disabled or a static unit, not starting it.
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.10) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

설치가 완료 되었으면 서비스를 켜주면 된다.

```shell
# 서비스 관리
admin@test:~$ sudo systemctl start glusterd.service

admin@test:~$ sudo systemctl status glusterd.service
● glusterd.service - GlusterFS, a clustered file-system server
     Loaded: loaded (/lib/systemd/system/glusterd.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-07-14 05:37:56 UTC; 6s ago
       Docs: man:glusterd(8)
    Process: 16108 ExecStart=/usr/sbin/glusterd -p /var/run/glusterd.pid --log-level $LOG_LEVEL $GLUSTERD_OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 16109 (glusterd)
      Tasks: 24 (limit: 9363)
     Memory: 15.5M
        CPU: 1.396s
     CGroup: /system.slice/glusterd.service
             └─16109 /usr/sbin/glusterd -p /var/run/glusterd.pid --log-level INFO

Jul 14 05:37:54 test systemd[1]: Starting GlusterFS, a clustered file-system server...
Jul 14 05:37:56 test systemd[1]: Started GlusterFS, a clustered file-system server.

admin@test:~$ sudo systemctl enable glusterd.service
Created symlink /etc/systemd/system/multi-user.target.wants/glusterd.service → /lib/systemd/system/glusterd.service.
```

## Standalone server
---

한 개의 서버에서 생성이 가능하다.  
보통은 BMT 또는 k3s 같은 곳에서 사용 한다.

### 1. 구성

```shell
# GlusterFS에서 최상위 디렉토리는 지원 안하므로 하위 디렉토리 생성한다.
admin@test:~$ sudo mkdir -p /data/brick/gv0

# volume 생성 시 hostname을 이용해 준다.
admin@test:~$ sudo gluster volume create gv0 m:/data/brick/gv0
volume create: gv0: success: please start the volume to access data

# 구성 확인
admin@test:~$ sudo gluster volume info
Volume Name: gv0
Type: Distribute
Volume ID: dde81d1e-dde3-4317-aa89-722344ed0e16
Status: Started
Snapshot Count: 0
Number of Bricks: 1
Transport-type: tcp
Bricks:
Brick1: m:/data/brick/gv0
Options Reconfigured:
storage.fips-mode-rchecksum: on
transport.address-family: inet
nfs.disable: on

# gv0 volume 시작
admin@test:~$ sudo gluster volume start gv0
volume start: gv0: success
```

### 2. 삭제

```shell
# gv0 volume stop
admin@test:~$$ sudo gluster volume stop gv0
Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
volume stop: gv0: success

# gv0 volume 삭제
admin@test:~$ sudo gluster volume delete gv0
Deleting volume will erase all information about the volume. Do you want to continue? (y/n) y
volume delete: gv0: success

# 디렉토리 삭제
admin@test:~$ sudo rm -rf /data/brick/gv0
```

## Standalone client
---

### 1. 설치

Client의 설치는 위에 기본설치 과정과 같다.

### 2. Mount

```shell
admin@n1:~# sudo mount -t glusterfs 10.50.1.242:/gv0 /mnt
```

`/etc/fstab`에 등록하여 부팅시 자동으로 mount 시킬수 있다.

```shell
admin@n1:~# sudo vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/vda2 during curtin installation
/dev/disk/by-uuid/c0d1cf89-0be0-48a9-9c5d-b35d9a96c160 / ext4 defaults 0 1
# /boot/efi was on /dev/vda1 during curtin installation
/dev/disk/by-uuid/FA83-ADDF /boot/efi vfat defaults 0 1
#/swap.img      none    swap    sw      0       0

# Mpount glusterfs server
10.50.1.242:/gv0 /mnt glusterfs defaults,_netdev 0 0

admin@n1:~# sudo mount -a
```

### 3. 다중 파일 테스트

간단히 `for문`을 이용하여 syslog를 copy 하는 방법이 있다.

```shell
for i in `seq -w 1 100`; do cp -rp /var/log/syslog /mnt/copy-test-$i; done
```

## Log 확인
---

기본적으로 GlusterFS의 log는 `/var/log/glusterfs` 이하에 쌓이게 된다.  
Brick의 log는 `/var/log/glusterfs/brick` 에서 확인이 가능하다