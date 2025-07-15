---
title: Ubuntu - DRBD(Distributed Replicated Block Device) 구축
date: 2023-09-12 17:42:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, drbd]
description: Ubuntu에서 Disk HA(High Availability)의 구성을 해주는 DRBD를 구축해보는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* DRBD(Distributed Replicated Block Device)는 사전적 의미와 동일하게 Block device(보통 Disk를 의미)에 대해 Distributed Replicated(분산 복제)를 해준다.
* 보통 `haproxy`와 같은 패키지에 대해 설정값 또는 특정 설정들에 대해 mirroring 할 때 필요하다.

> ref.
> * <https://ubuntu.com/server/docs/ubuntu-ha-drbd>

## 설치
---

Dependency에 기본적으로 `postfix`(메일 서버)가 있다.  
`postfix`가 필요가 없으면 `--no-install-recommends` 옵션을 설정해서 써준다.  
mirroring이 기본이기에 양쪽 node에 다 설치를 해준다.

```shell
dor1@haproxy1:~$ sudo apt install drbd-utils --no-install-recommends
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Suggested packages:
  heartbeat
Recommended packages:
  bsd-mailx | mailx
The following NEW packages will be installed:
  drbd-utils
0 upgraded, 1 newly installed, 0 to remove and 57 not upgraded.
Need to get 734 kB of archives.
After this operation, 2,010 kB of additional disk space will be used.
Get:1 http://mirror.kakao.com/ubuntu jammy/main amd64 drbd-utils amd64 9.15.0-1build2 [734 kB]
Fetched 734 kB in 0s (3,906 kB/s)
Selecting previously unselected package drbd-utils.
(Reading database ... 80240 files and directories currently installed.)
Preparing to unpack .../drbd-utils_9.15.0-1build2_amd64.deb ...
Unpacking drbd-utils (9.15.0-1build2) ...
Setting up drbd-utils (9.15.0-1build2) ...
Processing triggers for man-db (2.10.2-1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

## 세팅 및 테스트
---

### 1.  DRBD config 세팅

`/etc/drbd.conf`에 기본적인 세팅을 확인해보면 `/etc/drbd.d`의 디렉토리 안에 res를 만든 뒤 설정하게 되어있다.  
양쪽의 node에 다 세팅이 필요하다.

```shell
# config 세팅
dor1@haproxy1:~$ sudo vi /etc/drbd.d/haproxy.res
resource r0 {
  protocol C;
  startup {
    wfc-timeout  15;
    degr-wfc-timeout 60;
  }
  net {
    cram-hmac-alg sha1;
    # 랜덤 값으로 가능
    shared-secret "sync_disk";
  }
  # 수정
  on haproxy1 {
    device /dev/drbd0;
    disk /dev/sdb;
    address 192.168.0.201:7788;
    meta-disk internal;
  }
  on haproxy2 {
    device /dev/drbd0;
    disk /dev/sdb;
    address 192.168.0.202:7788;
    meta-disk internal;
  }
}
```

다른 node에도 바라볼 수 있도록 config 값을 넘긴다.

```shell
# scp 로 전달
dor1@haproxy1:~$ sudo scp /etc/drbd.d/haproxy.res dor1@haproxy2:~
dor1@haproxy2 password:
haproxy.res                                                                                                            100%  456     1.4MB/s   00:00

# 2번 node에서 config 이전
dor1@haproxy2:~$ sudo chown root:root haproxy.res
dor1@haproxy2:~$ sudo mv haproxy.res /etc/drbd.d/
```

### 2. `drbdadm`을 통하여 md(Software RAID) 생성

`drbd`의 생성에는 2대의 node에서 다 진행한다.  
단, 마지막에 우선순위 작업은 1번 node에서 진행한다.

```shell
# md 생성 [[2 node 다 진행]
dor1@haproxy1:~$ sudo drbdadm create-md r0
initializing activity log
initializing bitmap (1024 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.

# 서비스 시작 [2 node 다 진행]
dor1@haproxy1:~$ sudo systemctl start drbd.service
dor1@haproxy1:~$ sudo systemctl status drbd
● drbd.service - DRBD -- please disable. Unless you are NOT using a cluster manager.
     Loaded: loaded (/lib/systemd/system/drbd.service; disabled; vendor preset: enabled)
     Active: active (exited) since Tue 2023-09-12 18:09:39 KST; 11s ago
    Process: 3281 ExecStart=/lib/drbd/drbd start (code=exited, status=0/SUCCESS)
   Main PID: 3281 (code=exited, status=0/SUCCESS)
        CPU: 61ms

Sep 12 18:09:24 haproxy1 drbd[3289]:     adjust disk: r0
Sep 12 18:09:24 haproxy1 drbd[3289]:      adjust net: r0
Sep 12 18:09:24 haproxy1 drbd[3289]: ]
Sep 12 18:09:24 haproxy1 drbd[3325]: WARN: stdin/stdout is not a TTY; using /dev/console
Sep 12 18:09:24 haproxy1 drbd[3327]: degr-wfc-timeout has to be shorter than wfc-timeout
Sep 12 18:09:24 haproxy1 drbd[3327]: degr-wfc-timeout implicitly set to wfc-timeout (15s)
Sep 12 18:09:24 haproxy1 drbd[3327]: outdated-wfc-timeout has to be shorter than degr-wfc-timeout
Sep 12 18:09:24 haproxy1 drbd[3327]: outdated-wfc-timeout implicitly set to degr-wfc-timeout (15s)
Sep 12 18:09:39 haproxy1 drbd[3281]:    ...done.
Sep 12 18:09:39 haproxy1 systemd[1]: Finished DRBD -- please disable. Unless you are NOT using a cluster manager..

# drbd 확인 [2 node 다 진행]
dor1@haproxy1:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0   62M  1 loop /snap/core20/1587
loop1     7:1    0 79.9M  1 loop /snap/lxd/22923
loop2     7:2    0   47M  1 loop /snap/snapd/16292
sda       8:0    0   64G  0 disk
├─sda1    8:1    0    1G  0 part /boot/efi
└─sda2    8:2    0   63G  0 part /
sdb       8:16   0   32G  0 disk
└─drbd0 147:0    0   32G  1 disk
sr0      11:0    1  1.4G  0 rom

# 1번 node로 우선순위 작업(1번 node에서만 필수!)
dor1@haproxy1:~$ sudo drbdadm -- --overwrite-data-of-peer primary all
```

이 후 2번 node에서 확인 해본다.

```shell
# drbd sync 확인
dor1@haproxy2:~$ watch -n1 cat /proc/drbd
Every 1.0s: cat /proc/drbd
haproxy2: Tue Sep 12 18:30:46 2023

version: 8.4.11 (api:1/proto:86-101)
srcversion: 3605238F092EFFCDC77483B
 0: cs:SyncTarget ro:Secondary/Primary ds:Inconsistent/UpToDate C r-----
    ns:0 nr:1003520 dw:1003520 dr:0 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:32549852
        [>....................] sync'ed:  3.1% (31784/32764)M
        finish: 0:14:01 speed: 38,656 (38,596) want: 50,280 K/sec
```

connected 상태가 되었다면 1번 node에서 partition을 만든 뒤 mount 해준다.

```shell
# ext4로 생성
dor1@haproxy1:~$ sudo mkfs.ext4 /dev/drbd0
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 8388343 4k blocks and 2097152 inodes
Filesystem UUID: 4df186e6-4474-47bd-b5fc-d4f8c603360c
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information:

done

# mount 및 확인
dor1@haproxy1:~$ sudo mount /dev/drbd /config
dor1@haproxy1:~$ lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0    62M  1 loop /snap/core20/1587
loop1     7:1    0  79.9M  1 loop /snap/lxd/22923
loop3     7:3    0  40.8M  1 loop /snap/snapd/20092
loop4     7:4    0 111.9M  1 loop /snap/lxd/24322
sda       8:0    0    64G  0 disk
├─sda1    8:1    0     1G  0 part /boot/efi
└─sda2    8:2    0    63G  0 part /
sdb       8:16   0    32G  0 disk
└─drbd0 147:0    0    32G  0 disk /config
sr0      11:0    1   1.4G  0 rom
```

### 3. 테스트

파일을 생성해본 뒤 2번째 node에서 확인해본다.  
DRBD 특성 상 primary를 내리고 secondary를 올리는 방법이다 보니, 실시간 sync는 기대하기는 어렵다.  
1번 node가 죽었을 경우 2번에서는 데이터는 받아올 수 있다.

```shell
# 1번 node에서 파일 생성
dor1@haproxy1:~$ cd /config
dor1@haproxy1:/config$ sudo fallocate -l 10G test
dor1@haproxy1:/config$ ll
total 10485772
drwxr-xr-x  2 root root        4096 Sep 13 08:57 ./
drwxr-xr-x 20 root root        4096 Sep 13 08:53 ../
-rw-r--r--  1 root root 10737418240 Sep 13 08:57 test

# 1번 node에서 생성 후 secondary node에서 r0에서 바라볼 수 있게 세팅, unmount 필수
dor1@haproxy1:~$ sudo umount /config
dor1@haproxy1:~$ sudo drbdadm secondary r0

# secondary(2번 node)를 primary로 변경 후 mount 하여 파일 확인
dor1@haproxy2:~$ sudo drbdadm primary r0
dor1@haproxy2:~$ sudo mount /dev/drbd /config
dor1@haproxy2:~$ cd /config
dor1@haproxy2:/config$ ll
total 10485772
drwxr-xr-x  2 root root        4096 Sep 13 08:57 ./
drwxr-xr-x 20 root root        4096 Sep 13 08:59 ../
-rw-r--r--  1 root root 10737418240 Sep 13 08:57 test
```