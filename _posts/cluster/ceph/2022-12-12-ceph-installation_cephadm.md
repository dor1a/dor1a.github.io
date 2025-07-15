---
title: Ceph - cephadm으로 설치
date: 2022-12-12 14:11:00 +0900
categories: [Cluster, Ceph]
tags: [cluster, ceph, cephadm]
description: Ceph을 cephadm으로 설치하는 방법이다.
---

>Ceph v17.2.5 (quincy) / Ubuntu 20.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요 및 요구 사항
---

* Ceph은 기본적으로 2개의 node(Master 1 + Storage 1) 이상으로 구성하며 디스크는 4개 이상으로 진행해야 Warning 메세지가 발생하지 않는다.
* 네트워크는 최소 1GbE 이상을 요구한다.
* `cephfs`는 NAS 용도 외에는 크게 사용할 일이 없기 때문에, 25GbE 이상은 필요할까? 라는 의문이 든다.
* `salt`, `puppet`, `ansible`, `rook` 등 많은 자동화 도구로 deployment를 할 수 있지만, 최신 버전에서는 `cephadm`을 권장하고 있다.
* Ceph은 `osd(Object Storage Daemon)`, `mon(Monitors)`, `mgr(Managers)`, `mds(Metadata Server)`로 구성 되어있다.
* k8s에서 Ceph은 사용법이 3가지 정도가 있다.
  1. Block Device(RBD) 1개의 pod에서 사용(RWO)
  2. Shared Filesystem(CephFS)는 Multiple pods에서 사용(RWX)
  3. Object(RGW)는 inside and outside cluster에서 사용

> ref.
> - <https://docs.ceph.com/en/quincy/install/>

## cephadm으로 설치하기
---

`cephadm`으로 설치하게 된다면 `docker`가 기본으로 설치 되어있어야 한다.
`docker`는 Master 뿐만이 아닌 ceph 구축할 모든 node에 반드시 설치 되어있어야 한다.

**Testbed**
* m1 (master)
* s1 (two disk per one node for `osd`)
* s2 (two disk per one node for `osd`)

### 1. cephadm 설치

Official documents에서는 `curl-based`와 `distribution-specific` 두 가지 방법을 제공해준다.  
`distribution-specific`은 Linux의 배포판에 따라 패키지로 설치가 진행 됨을 의미하며, Repository가 업데이트 될 때마다 새로운 버전이 올라가기 때문에 버전 고정이 되지 않을 수도 있다.  
보통 Cluster 구성의 버전은 지정을 한 뒤 설치하기 때문에 `curl-based`가 좀 더 권장되는 방법이다.  
또한 Ceph Cluster는 기본적으로 root 권한을 요구하기 때문에 root로 진행 해 주는 것이 좋다.  
Storage Cluster 특성 상 한번 구축하면 재구성이나 수정할 이유가 거의 없다.

```shell
# root로 진행 (server)
ubuntu@m1:~$ sudo -i

# cephadm binary
root@m1:~$ curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
root@m1:~$ chmod +x cephadm

root@m1:~$ ./cephadm add-repo --release quincy
Installing repo GPG key from https://download.ceph.com/keys/release.gpg...
Installing repo file at /etc/apt/sources.list.d/ceph.list...
Updating package list...
Completed adding repo.

root@m1:~$ ./cephadm install
Installing packages ['cephadm']...

# cephadm 설치 확인
root@m1:~$ which cephadm
/usr/sbin/cephadm

# cephadm이 제대로 설치 되었다면 curl로 받은 건 필요 없으므로 삭제
root@m1:~$ rm ./cephadm

# /usr/sbin/cephadm의 binary로 ceph 설치를 완료 함
root@m1:~$ cephadm add-repo --release quincy
Installing repo GPG key from https://download.ceph.com/keys/release.gpg...
Installing repo file at /etc/apt/sources.list.d/ceph.list...
Updating package list...
Completed adding repo.

root@m1:~$ cephadm install ceph-common
Installing packages ['ceph-common']...
```

### 2. bootstrap (Web Dashboard) 설치

`cephadm`을 통하여 bootstrap을 설치하게 된다면 `mon(Monitors)` daemon도 같이 설치 된다.

```shell
# server
root@m1:~$ cephadm bootstrap --mon-ip 10.3.0.118
Creating directory /etc/ceph for ceph.conf
Verifying podman|docker is present...
Verifying lvm2 is present...
Verifying time synchronization is in place...
Unit systemd-timesyncd.service is enabled and running
Repeating the final host check...
docker (/usr/bin/docker) is present
systemctl is present
lvcreate is present
Unit systemd-timesyncd.service is enabled and running
Host looks OK
Cluster fsid: 6486cbb6-79e3-11ed-9d26-ebae253e4923
Verifying IP 10.3.0.118 port 3300 ...
Verifying IP 10.3.0.118 port 6789 ...
Mon IP `10.3.0.118` is in CIDR network `10.3.0.0/22`
Mon IP `10.3.0.118` is in CIDR network `10.3.0.0/22`
Mon IP `10.3.0.118` is in CIDR network `10.3.0.1/32`
Mon IP `10.3.0.118` is in CIDR network `10.3.0.1/32`
Ceph version: ceph version 17.2.5 (98318ae89f1a893a6ded3a640405cdbb33e08757) quincy (stable)the public_network
Extracting ceph user uid/gid from container image...
Creating initial keys...
Creating initial monmap...
Creating mon...
Waiting for mon to start...
Waiting for mon...
mon is available
Assimilating anything we can from ceph.conf...
Generating new minimal ceph.conf...
Restarting the monitor...
Setting mon public_network to 10.3.0.1/32,10.3.0.0/22
Wrote config to /etc/ceph/ceph.conf
Wrote keyring to /etc/ceph/ceph.client.admin.keyring
Creating mgr...
Verifying port 9283 ...
Waiting for mgr to start...
Waiting for mgr...
mgr not available, waiting (1/15)...
mgr not available, waiting (2/15)...
mgr not available, waiting (3/15)...
mgr is available
Enabling cephadm module...
Waiting for the mgr to restart...
Waiting for mgr epoch 5...
mgr epoch 5 is available
Setting orchestrator backend to cephadm...
Generating ssh key...
Wrote public SSH key to /etc/ceph/ceph.pub
Adding key to root@localhost authorized_keys...
Adding host m1...
Deploying mon service with default placement...
Deploying mgr service with default placement...
Deploying crash service with default placement...
Deploying prometheus service with default placement...
Deploying grafana service with default placement...
Deploying node-exporter service with default placement...
Deploying alertmanager service with default placement...
Enabling the dashboard module...
Waiting for the mgr to restart...
Waiting for mgr epoch 9...
mgr epoch 9 is available
Generating a dashboard self-signed certificate...
Creating initial admin user...
Fetching dashboard port number...
Ceph Dashboard is now available at:

             URL: https://m1:8443/
            User: admin
        Password: qurtnkshq6

Enabling client.admin keyring and conf on hosts with "admin" label
Saving cluster configuration to /var/lib/ceph/6486cbb6-79e3-11ed-9d26-ebae253e4923/config directory
Enabling autotune for osd_memory_target
You can access the Ceph CLI as following in case of multi-cluster or non-default config:

        sudo /usr/sbin/cephadm shell --fsid 6486cbb6-79e3-11ed-9d26-ebae253e4923 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

Or, if you are only running a single cluster on this host:

        sudo /usr/sbin/cephadm shell

Please consider enabling telemetry to help improve Ceph:

        ceph telemetry on

For more information see:

        https://docs.ceph.com/docs/master/mgr/telemetry/

Bootstrap complete.
```

설치가 완료 되면 Console에 나온 것과 같이 `https://<ip>:8443`에 웹으로 접근하면 Web dashboard가 나오게 된다.  
quincy 버전의 dashboard는 다음과 같다.

![Web dashboard](/assets/img/post/cluster/ceph/2022-12-12-ceph-installation_cephadm/1.png)
_Web dashboard_

### 3. Hosts 추가

Hosts 추가 때에도 Storage node의 root 권한에 대해서 필요하기 때문에, root 계정의 [Ubuntu - SSH key를 통한 무인증 로그인](/posts/ubuntu-ssh_key_based_authentication/)의 작업을 필수로 해줘야 한다.  
Ubuntu는 기본적으로 root 계정의 비밀번호가 따로 존재하지 않기 때문에 만들어줘야 한다.  
무인증 로그인 작업이 완료 되었다 하면, Hosts을 추가해준다.  
작업 전 다시 한번 Storage node에 `docker`가 설치 되어있는지 확인한다.

```shell
# ceph key값을 Hosts의 /root/.ssh/authorized_keys에 추가 함 (server)
root@m1:~$ ssh-copy-id -f -i /etc/ceph/ceph.pub root@s1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/etc/ceph/ceph.pub"

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@s1'"
and check to make sure that only the key(s) you wanted were added.

root@m1:~$ ssh-copy-id -f -i /etc/ceph/ceph.pub root@s2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/etc/ceph/ceph.pub"

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@s2'"
and check to make sure that only the key(s) you wanted were added.

# Adding hosts
root@m1:~$ ceph orch host add s1 10.3.0.119
Added host 's1' with addr '10.3.0.119'
root@m1:~$ ceph orch host add s2 10.3.0.123
Added host 's2' with addr '10.3.0.123'

# 확인
root@m1:~$ ceph orch host ls
HOST  ADDR        LABELS  STATUS
m1    10.3.0.118  _admin
s1    10.3.0.119
s2    10.3.0.123
3 hosts in cluster
```

Host 추가를 하면 `mon` daemon은 자연스럽게 모든 node에 deploy이 된다.

### 4. Storage (osd) 추가

`cephadm`은 host를 추가하게 되면 알아서 disk list를 불러온다.

```shell
# Disk 확인 (server)
root@m1:~$ ceph orch device ls
HOST  PATH      TYPE  DEVICE ID   SIZE  AVAILABLE  REFRESHED  REJECT REASONS
s1    /dev/sdb  ssd               274G  Yes        3m ago
s1    /dev/sdc  ssd               274G  Yes        3m ago
s2    /dev/sdb  ssd               274G  Yes        3m ago
s2    /dev/sdc  ssd               274G  Yes        3m ago

# orach의 기능 --all-available-devices로 한번에 등록(디스크가 많다면 시간이 다소 걸림)
root@m1:~$ ceph orch apply osd --all-available-devices
Scheduled osd.all-available-devices update...

# osd.all-available-devices의 RUNNING status를 통하여 확인
root@m1:~$ ceph orch ls
NAME                       PORTS        RUNNING  REFRESHED  AGE  PLACEMENT
alertmanager               ?:9093,9094      1/1  5m ago     34m  count:1
crash                                       3/3  5m ago     34m  *
grafana                    ?:3000           1/1  5m ago     34m  count:1
mgr                                         2/2  5m ago     34m  count:2
mon                                         3/5  5m ago     34m  count:5
node-exporter              ?:9100           3/3  5m ago     34m  *
osd.all-available-devices                     4  2s ago     34s  *
prometheus                 ?:9095           1/1  5m ago     34m  count:1
```

### 5. CephFS 구성

[ceph osd pool](https://docs.ceph.com/en/latest/rados/operations/placement-groups/) 명령어를 통하여 FS(File System)을 구성하게 된다면 **PG**(A placement group)가 auto-scale로 구성 된다.  
PG는 Ceph 구성에 핵심인데 구체적으로 알아보려면 다음의 문서를 참조하면 된다.

- <https://docs.ceph.com/en/latest/rados/operations/placement-groups/#viewing-pg-scaling-recommendations>
- <https://old.ceph.com/pgcalc/>

```shell
# pool을 먼저 만든 뒤 fs를 만듬 (server)
root@m1:~$ ceph osd pool create cephfs_metadata
pool 'cephfs_metadata' created

root@m1:~$ ceph osd pool create cephfs_data
pool 'cephfs_data' created

root@m1:~$ ceph fs new cephfs cephfs_metadata cephfs_data
new fs with metadata pool 2 and data pool 3

# 구성 확인
root@m1:~$ ceph osd status
ID  HOST   USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE
 0  s2    7840k   255G      0        0       0        0   exists,up
 1  s1    8096k   255G      0        0       0        0   exists,up
 2  s2    8804k   255G      0        0       0        0   exists,up
 3  s1    8612k   255G      0        0       0        0   exists,up

root@m1:~$ ceph fs status
cephfs - 0 clients
======
      POOL         TYPE     USED  AVAIL
cephfs_metadata  metadata     0    324G
cephfs_data        data       0    324G

root@m1:~$ ceph osd pool stats
pool .mgr id 1
  2/6 objects degraded (33.333%)

pool cephfs_data id 2
  nothing is going on

pool cephfs_metadata id 3
  nothing is going on

root@m1:~$ ceph osd pool autoscale-status
POOL               SIZE  TARGET SIZE  RATE  RAW CAPACITY   RATIO  TARGET RATIO  EFFECTIVE RATIO  BIAS  PG_NUM  NEW PG_NUM  AUTOSCALE  BULK
.mgr             448.5k                3.0         1023G  0.0000                                  1.0       1              on         False
cephfs_metadata      0                 3.0         1023G  0.0000                                  4.0      16              on         False
cephfs_data          0                 3.0         1023G  0.0000                                  1.0      32              on         False
```

CephFS 구성한다 해도 mds가 없다면 다음과 같은 에러가 발생하게 된다.

![mds Error](/assets/img/post/cluster/ceph/2022-12-12-ceph-installation_cephadm/2.png)
_mds Error_

### 6. mds 구성

mds는 말 그대로 Metadata만 저장해주는 서버다.  
mds는 필요 시 여러 node에 올려 줄 수도 있으며 mds가 없으면 CephFS는 정상 작동이 불가능 하다.  
mds는 orch로 쉽게 구성이 가능하다.

```shell
# 2 node에 mds를 구성하게 되면 active/standby로 동작 함 (server)
root@m1:~$ ceph orch apply mds mds.cephfs --placement="2 s1 s2"
Scheduled mds.mds.cephfs update...

# 구성 확인
root@m1:~$ ceph orch ls
NAME                       PORTS        RUNNING  REFRESHED  AGE  PLACEMENT
alertmanager               ?:9093,9094      1/1  10m ago    60m  count:1
crash                                       3/3  10m ago    60m  *
grafana                    ?:3000           1/1  10m ago    60m  count:1
mds.mds.cephfs                              2/2  41s ago    48s  s1;s2;count:2
mgr                                         2/2  10m ago    60m  count:2
mon                                         3/5  10m ago    60m  count:5
node-exporter              ?:9100           3/3  10m ago    60m  *
osd.all-available-devices                     4  41s ago    26m  *
prometheus                 ?:9095           1/1  10m ago    60m  count:1

root@m1:~$ ceph fs status
cephfs - 0 clients
======
RANK  STATE           MDS              ACTIVITY     DNS    INOS   DIRS   CAPS
 0    active  mds.cephfs.s2.xvdpfr  Reqs:    0 /s    10     13     12      0
      POOL         TYPE     USED  AVAIL
cephfs_metadata  metadata   2306  486G
  cephfs_data      data       0   324G
    STANDBY MDS
mds.cephfs.s1.zleqsv
MDS version: ceph version 17.2.5 (98318ae89f1a893a6ded3a640405cdbb33e08757) quincy (stable)
```

## Kernel driver로 mount 하기
---

>해당 부분은 legacy 방법을 적어 놓았으며 최신 버전에서는 mount 방법이 다르기 때문에 아래의 문서에서 확인이 가능하다.  
[Ceph - CephFS Client 권한 부여](/posts/ceph-cephfs_client_authorization)
{: .prompt-info}

Ceph 10.x (Jewel) 또는 4.x kernel 이상이면 지원된다.  
Client에서는 NFS와 비슷하게 `ceph-common`의 패키지가 필요하다.  
방법은 `mount` 명령어를 통하여 하는 법과 `/etc/fstab`에 등록하는 법 두 개 다 가능하다.  
mount에 secret을 적는 옵션이 있는데 이것은 base64로 이뤄진 key이며, master에서 확인이 가능하다.  
<https://docs.ceph.com/en/quincy/cephfs/mount-using-kernel-driver/>

```shell
# key 확인 (server)
root@m1:~$ cat /etc/ceph/ceph.client.admin.keyring
[client.admin]
        key = AQBuxZZjEWE0LhAAQVs4C3NgEBpzq1yGWj54sg==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
```

### 1. Client에 ceph-common설치

```shell
# client
deepadmin@gbia-tmp-cpu1:/$ sudo apt install ceph-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ibverbs-providers libbabeltrace1 libboost-context1.74.0 libboost-filesystem1.74.0 libboost-iostreams1.74.0 libboost-program-options1.74.0
  libboost-thread1.74.0 libcephfs2 libdaxctl1 libgoogle-perftools4 libibverbs1 liblua5.3-0 libndctl6 libnl-route-3-200 liboath0 libpmem1 libpmemobj1
  librabbitmq4 librados2 libradosstriper1 librbd1 librdmacm1 libsnappy1v5 libtcmalloc-minimal4 python3-ceph-argparse python3-ceph-common python3-cephfs
  python3-prettytable python3-rados python3-rbd python3-wcwidth
Suggested packages:
  ceph ceph-mds
The following NEW packages will be installed:
  ceph-common ibverbs-providers libbabeltrace1 libboost-context1.74.0 libboost-filesystem1.74.0 libboost-iostreams1.74.0 libboost-program-options1.74.0
  libboost-thread1.74.0 libcephfs2 libdaxctl1 libgoogle-perftools4 libibverbs1 liblua5.3-0 libndctl6 libnl-route-3-200 liboath0 libpmem1 libpmemobj1
  librabbitmq4 librados2 libradosstriper1 librbd1 librdmacm1 libsnappy1v5 libtcmalloc-minimal4 python3-ceph-argparse python3-ceph-common python3-cephfs
  python3-prettytable python3-rados python3-rbd python3-wcwidth
0 upgraded, 32 newly installed, 0 to remove and 64 not upgraded.
Need to get 23.4 MB/35.4 MB of archives.
After this operation, 141 MB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://kr.archive.ubuntu.com/ubuntu jammy-updates/main amd64 ceph-common amd64 17.2.7-0ubuntu0.22.04.1 [23.0 MB]
Get:2 http://kr.archive.ubuntu.com/ubuntu jammy/main amd64 ibverbs-providers amd64 39.0-1 [341 kB]
Fetched 8,379 kB in 2s (3,532 kB/s)
Extracting templates from packages: 100%
Selecting previously unselected package libboost-iostreams1.74.0:amd64.
(Reading database ... 116000 files and directories currently installed.)
Preparing to unpack .../00-libboost-iostreams1.74.0_1.74.0-14ubuntu3_amd64.deb ...
Unpacking libboost-iostreams1.74.0:amd64 (1.74.0-14ubuntu3) ...
Selecting previously unselected package libboost-thread1.74.0:amd64.
Preparing to unpack .../01-libboost-thread1.74.0_1.74.0-14ubuntu3_amd64.deb ...
Unpacking libboost-thread1.74.0:amd64 (1.74.0-14ubuntu3) ...
Selecting previously unselected package libnl-route-3-200:amd64.
Preparing to unpack .../02-libnl-route-3-200_3.5.0-0.1_amd64.deb ...
Unpacking libnl-route-3-200:amd64 (3.5.0-0.1) ...
Selecting previously unselected package libibverbs1:amd64.
Preparing to unpack .../03-libibverbs1_39.0-1_amd64.deb ...
Unpacking libibverbs1:amd64 (39.0-1) ...
Selecting previously unselected package librdmacm1:amd64.
Preparing to unpack .../04-librdmacm1_39.0-1_amd64.deb ...
Unpacking librdmacm1:amd64 (39.0-1) ...
Selecting previously unselected package librados2.
Preparing to unpack .../05-librados2_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking librados2 (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package libdaxctl1:amd64.
Preparing to unpack .../06-libdaxctl1_72.1-1_amd64.deb ...
Unpacking libdaxctl1:amd64 (72.1-1) ...
Selecting previously unselected package libndctl6:amd64.
Preparing to unpack .../07-libndctl6_72.1-1_amd64.deb ...
Unpacking libndctl6:amd64 (72.1-1) ...
Selecting previously unselected package libpmem1:amd64.
Preparing to unpack .../08-libpmem1_1.11.1-3build1_amd64.deb ...
Unpacking libpmem1:amd64 (1.11.1-3build1) ...
Selecting previously unselected package libpmemobj1:amd64.
Preparing to unpack .../09-libpmemobj1_1.11.1-3build1_amd64.deb ...
Unpacking libpmemobj1:amd64 (1.11.1-3build1) ...
Selecting previously unselected package librbd1.
Preparing to unpack .../10-librbd1_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking librbd1 (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package python3-ceph-argparse.
Preparing to unpack .../11-python3-ceph-argparse_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking python3-ceph-argparse (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package python3-ceph-common.
Preparing to unpack .../12-python3-ceph-common_17.2.7-0ubuntu0.22.04.1_all.deb ...
Unpacking python3-ceph-common (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package libcephfs2.
Preparing to unpack .../13-libcephfs2_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking libcephfs2 (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package python3-rados.
Preparing to unpack .../14-python3-rados_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking python3-rados (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package python3-cephfs.
Preparing to unpack .../15-python3-cephfs_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking python3-cephfs (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package python3-wcwidth.
Preparing to unpack .../16-python3-wcwidth_0.2.5+dfsg1-1_all.deb ...
Unpacking python3-wcwidth (0.2.5+dfsg1-1) ...
Selecting previously unselected package python3-prettytable.
Preparing to unpack .../17-python3-prettytable_2.5.0-2_all.deb ...
Unpacking python3-prettytable (2.5.0-2) ...
Selecting previously unselected package python3-rbd.
Preparing to unpack .../18-python3-rbd_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking python3-rbd (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package libbabeltrace1:amd64.
Preparing to unpack .../19-libbabeltrace1_1.5.8-2build1_amd64.deb ...
Unpacking libbabeltrace1:amd64 (1.5.8-2build1) ...
Selecting previously unselected package libboost-context1.74.0:amd64.
Preparing to unpack .../20-libboost-context1.74.0_1.74.0-14ubuntu3_amd64.deb ...
Unpacking libboost-context1.74.0:amd64 (1.74.0-14ubuntu3) ...
Selecting previously unselected package libboost-filesystem1.74.0:amd64.
Preparing to unpack .../21-libboost-filesystem1.74.0_1.74.0-14ubuntu3_amd64.deb ...
Unpacking libboost-filesystem1.74.0:amd64 (1.74.0-14ubuntu3) ...
Selecting previously unselected package libboost-program-options1.74.0:amd64.
Preparing to unpack .../22-libboost-program-options1.74.0_1.74.0-14ubuntu3_amd64.deb ...
Unpacking libboost-program-options1.74.0:amd64 (1.74.0-14ubuntu3) ...
Selecting previously unselected package libtcmalloc-minimal4:amd64.
Preparing to unpack .../23-libtcmalloc-minimal4_2.9.1-0ubuntu3_amd64.deb ...
Unpacking libtcmalloc-minimal4:amd64 (2.9.1-0ubuntu3) ...
Selecting previously unselected package libgoogle-perftools4:amd64.
Preparing to unpack .../24-libgoogle-perftools4_2.9.1-0ubuntu3_amd64.deb ...
Unpacking libgoogle-perftools4:amd64 (2.9.1-0ubuntu3) ...
Selecting previously unselected package liblua5.3-0:amd64.
Preparing to unpack .../25-liblua5.3-0_5.3.6-1build1_amd64.deb ...
Unpacking liblua5.3-0:amd64 (5.3.6-1build1) ...
Selecting previously unselected package liboath0:amd64.
Preparing to unpack .../26-liboath0_2.6.7-3build1_amd64.deb ...
Unpacking liboath0:amd64 (2.6.7-3build1) ...
Selecting previously unselected package librabbitmq4:amd64.
Preparing to unpack .../27-librabbitmq4_0.10.0-1ubuntu2_amd64.deb ...
Unpacking librabbitmq4:amd64 (0.10.0-1ubuntu2) ...
Selecting previously unselected package libradosstriper1.
Preparing to unpack .../28-libradosstriper1_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking libradosstriper1 (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package libsnappy1v5:amd64.
Preparing to unpack .../29-libsnappy1v5_1.1.8-1build3_amd64.deb ...
Unpacking libsnappy1v5:amd64 (1.1.8-1build3) ...
Selecting previously unselected package ceph-common.
Preparing to unpack .../30-ceph-common_17.2.7-0ubuntu0.22.04.1_amd64.deb ...
Unpacking ceph-common (17.2.7-0ubuntu0.22.04.1) ...
Selecting previously unselected package ibverbs-providers:amd64.
Preparing to unpack .../31-ibverbs-providers_39.0-1_amd64.deb ...
Unpacking ibverbs-providers:amd64 (39.0-1) ...
Setting up librabbitmq4:amd64 (0.10.0-1ubuntu2) ...
Setting up liboath0:amd64 (2.6.7-3build1) ...
Setting up libboost-iostreams1.74.0:amd64 (1.74.0-14ubuntu3) ...
Setting up libboost-program-options1.74.0:amd64 (1.74.0-14ubuntu3) ...
Setting up libtcmalloc-minimal4:amd64 (2.9.1-0ubuntu3) ...
Setting up python3-ceph-argparse (17.2.7-0ubuntu0.22.04.1) ...
Setting up libboost-filesystem1.74.0:amd64 (1.74.0-14ubuntu3) ...
Setting up libsnappy1v5:amd64 (1.1.8-1build3) ...
Setting up libnl-route-3-200:amd64 (3.5.0-0.1) ...
Setting up python3-wcwidth (0.2.5+dfsg1-1) ...
Setting up python3-ceph-common (17.2.7-0ubuntu0.22.04.1) ...
Setting up libboost-context1.74.0:amd64 (1.74.0-14ubuntu3) ...
Setting up libdaxctl1:amd64 (72.1-1) ...
Setting up libbabeltrace1:amd64 (1.5.8-2build1) ...
Setting up liblua5.3-0:amd64 (5.3.6-1build1) ...
Setting up libndctl6:amd64 (72.1-1) ...
Setting up python3-prettytable (2.5.0-2) ...
Setting up libpmem1:amd64 (1.11.1-3build1) ...
Setting up libgoogle-perftools4:amd64 (2.9.1-0ubuntu3) ...
Setting up libboost-thread1.74.0:amd64 (1.74.0-14ubuntu3) ...
Setting up libibverbs1:amd64 (39.0-1) ...
Setting up ibverbs-providers:amd64 (39.0-1) ...
Setting up libpmemobj1:amd64 (1.11.1-3build1) ...
Setting up librdmacm1:amd64 (39.0-1) ...
Setting up librados2 (17.2.7-0ubuntu0.22.04.1) ...
Setting up libcephfs2 (17.2.7-0ubuntu0.22.04.1) ...
Setting up libradosstriper1 (17.2.7-0ubuntu0.22.04.1) ...
Setting up librbd1 (17.2.7-0ubuntu0.22.04.1) ...
Setting up python3-rados (17.2.7-0ubuntu0.22.04.1) ...
Setting up python3-rbd (17.2.7-0ubuntu0.22.04.1) ...
Setting up python3-cephfs (17.2.7-0ubuntu0.22.04.1) ...
Setting up ceph-common (17.2.7-0ubuntu0.22.04.1) ...
Adding group ceph....done
Adding system user ceph....done
Setting system user ceph properties....done
chown: cannot access '/var/log/ceph/*.log*': No such file or directory
Created symlink /etc/systemd/system/multi-user.target.wants/ceph.target → /lib/systemd/system/ceph.target.
Created symlink /etc/systemd/system/multi-user.target.wants/rbdmap.service → /lib/systemd/system/rbdmap.service.
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.6) ...
```

### 2. 명령어를 통한 mount

```shell
# client
root@s1:~$ mount -t ceph 10.3.0.118:/ /mnt -o name=admin,secret=AQBuxZZjEWE0LhAAQVs4C3NgEBpzq1yGWj54sg==

# 확인
root@s1:~/.ssh$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           796M  2.0M  794M   1% /run
/dev/sda2       125G  8.7G  110G   8% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1       511M  5.3M  506M   2% /boot/efi
/dev/loop0       62M   62M     0 100% /snap/core20/1611
/dev/loop1       68M   68M     0 100% /snap/lxd/22753
/dev/loop2       47M   47M     0 100% /snap/snapd/16292
tmpfs           796M     0  796M   0% /run/user/1000
tmpfs           796M     0  796M   0% /run/user/0
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/45d075d7dd954c023fd1214c5cd49ec50d0de6a6bf751e1298cdebdaeb8628cc/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/11216523157c8e7603e405a916347ee37aeed8bd158194bb41fa79a04b65eaea/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/484c35268b8f279e0ddb7e2bebbb1c4d8b9b4f7d49f647a66846c9dcb9f6b007/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/c36b04439a7715f0874bb72c3af81a3448da09a69c9755ff9a3b7003af31e72c/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/e999d5743d652856a2dd09ebf2886242ab63d0c4d4e3e5498371e10eb45e23d3/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/608a64dc4971e81c2bf77b315773808d54a6ace9cc1979a3be7e2a9ecf793959/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/5924aeb89202645710c3cc3ff8ccf23e53d130d300874eab532177d412c7a339/merged
10.3.0.118:/    325G     0  325G   0% /mnt
```

### 3. /etc/fstab에 등록하여 mount

명령어를 통한 mount는 reboot이 되면 unmount가 되기 때문에, persistence를 위해 `/etc/fstab`에 등록하는 방법을 사용하는 것이 더 좋다.

```shell
# /etc/fstab (client)
root@s1:~$ vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/6abe1204-d753-4703-9937-2491c74580e0 / ext4 defaults 0 1
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/0057-2FA9 /boot/efi vfat defaults 0 1
/swap.img       none    swap    sw      0       0

# 추가 부분
10.3.0.118:/     /mnt    ceph    name=admin,secret=AQBuxZZjEWE0LhAAQVs4C3NgEBpzq1yGWj54sg==   0       0

root@s1:~$ mount -a

# 확인
root@s1:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           796M  2.0M  794M   1% /run
/dev/sda2       125G  8.7G  110G   8% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1       511M  5.3M  506M   2% /boot/efi
/dev/loop0       62M   62M     0 100% /snap/core20/1611
/dev/loop1       68M   68M     0 100% /snap/lxd/22753
/dev/loop2       47M   47M     0 100% /snap/snapd/16292
tmpfs           796M     0  796M   0% /run/user/1000
tmpfs           796M     0  796M   0% /run/user/0
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/45d075d7dd954c023fd1214c5cd49ec50d0de6a6bf751e1298cdebdaeb8628cc/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/11216523157c8e7603e405a916347ee37aeed8bd158194bb41fa79a04b65eaea/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/484c35268b8f279e0ddb7e2bebbb1c4d8b9b4f7d49f647a66846c9dcb9f6b007/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/c36b04439a7715f0874bb72c3af81a3448da09a69c9755ff9a3b7003af31e72c/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/e999d5743d652856a2dd09ebf2886242ab63d0c4d4e3e5498371e10eb45e23d3/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/608a64dc4971e81c2bf77b315773808d54a6ace9cc1979a3be7e2a9ecf793959/merged
overlay         125G  8.7G  110G   8% /var/lib/docker/overlay2/5924aeb89202645710c3cc3ff8ccf23e53d130d300874eab532177d412c7a339/merged
10.3.0.118:/    325G     0  325G   0% /mnt
```