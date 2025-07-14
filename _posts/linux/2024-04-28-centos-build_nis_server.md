---
title: CentOS - NIS Server 구축
date: 2024-04-28 20:19:00 +0900
categories: [Linux, CentOS]
tags: [linux, centos, nis]
description: CentOS에서 계정 관리하는 NIS(Network Information Service)의 server를 구축하는 방법이다.
---

>CentOS 7.9
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* NIS(Network Information Service)은 과거 Sun Microsystems에서 Directory service protocol이며, 굉장히 오래 된 기술이다.
* 요즘은 LDAP과 같은 기술이 있기 때문에 많이 쓰지는 않지만, 아직도 Red hat 계열에서는 쓰는 곳이 있다.
* 목적이 Directory service인 만큼 여러대의 서버에 대한 계정 관리를 위해 나왔다고 보면 된다.
* NIS에서는 기본적으로 **hostname.domain**의 rule이 있기 때문에 hostname을 맞추고 진행하는 것이 좋다.
* Client 설정은 다음과 같은 link로 가면 된다.  
  [Linux(CentOS) - NIS Client 설정](/posts/centos-setup_nis_client)

![NIS Server와 Client 구성도](/assets/img/post/linux/2024-04-28-centos-build_nis_server/1.png)
_NIS Server와 Client 구성도_

## 설치 및 세팅
---

### 1. NIS Server 패키지 설치

NIS Server 패키지는 `ypserv, rpcbind`두 가지 패키지로 이뤄져있다.  
`ypserv`의 yp는 Yellow Pages를 의미한다.  
두 패키지는 다음과 같이 사용된다.

|  패키지   | 내용                                                                                                                                                        |
| :-------: |
| `ypserv`  | NIS Server의 Daemon(service) 패키지이다.                                                                                                                    |
| `rpcbind` | **Remote Procedure Call (RPC) 서비스를 지원하는 패키지입니다.**<br>**RPC는 네트워크를 통해 다른 컴퓨터에서 프로그램을 실행할 수 있게 해주는 protocol이다.** |

```shell
[root@server ~]$ yum install ypserv rpcbind -y
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.kakao.com
 * extras: mirror.kakao.com
 * updates: mirror.kakao.com
base                                                                                                                                                                                      | 3.6 kB  00:00:00
extras                                                                                                                                                                                    | 2.9 kB  00:00:00
updates                                                                                                                                                                                   | 2.9 kB  00:00:00
(1/4): base/7/x86_64/group_gz                                                                                                                                                             | 153 kB  00:00:00
(2/4): extras/7/x86_64/primary_db                                                                                                                                                         | 253 kB  00:00:00
(3/4): updates/7/x86_64/primary_db                                                                                                                                                        |  26 MB  00:00:00
(4/4): base/7/x86_64/primary_db                                                                                                                                                           | 6.1 MB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package rpcbind.x86_64 0:0.2.0-49.el7 will be installed
--> Processing Dependency: libtirpc >= 0.2.4-0.7 for package: rpcbind-0.2.0-49.el7.x86_64
--> Processing Dependency: libtirpc.so.1()(64bit) for package: rpcbind-0.2.0-49.el7.x86_64
---> Package ypserv.x86_64 0:2.31-12.el7 will be installed
--> Processing Dependency: tokyocabinet for package: ypserv-2.31-12.el7.x86_64
--> Processing Dependency: libtokyocabinet.so.9()(64bit) for package: ypserv-2.31-12.el7.x86_64
--> Running transaction check
---> Package libtirpc.x86_64 0:0.2.4-0.16.el7 will be installed
---> Package tokyocabinet.x86_64 0:1.4.48-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================================================================================================================
 Package                                              Arch                                           Version                                                  Repository                                    Size
=================================================================================================================================================================================================================
Installing:
 rpcbind                                              x86_64                                         0.2.0-49.el7                                             base                                          60 k
 ypserv                                               x86_64                                         2.31-12.el7                                              base                                         156 k
Installing for dependencies:
 libtirpc                                             x86_64                                         0.2.4-0.16.el7                                           base                                          89 k
 tokyocabinet                                         x86_64                                         1.4.48-3.el7                                             base                                         459 k

Transaction Summary
=================================================================================================================================================================================================================
Install  2 Packages (+2 Dependent packages)

Total download size: 764 k
Installed size: 1.9 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/base/packages/rpcbind-0.2.0-49.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for rpcbind-0.2.0-49.el7.x86_64.rpm is not installed
(1/4): rpcbind-0.2.0-49.el7.x86_64.rpm                                                                                                                                                    |  60 kB  00:00:00
(2/4): libtirpc-0.2.4-0.16.el7.x86_64.rpm                                                                                                                                                 |  89 kB  00:00:00
(3/4): tokyocabinet-1.4.48-3.el7.x86_64.rpm                                                                                                                                               | 459 kB  00:00:00
(4/4): ypserv-2.31-12.el7.x86_64.rpm                                                                                                                                                      | 156 kB  00:00:00
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                            5.9 MB/s | 764 kB  00:00:00
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-9.2009.0.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : tokyocabinet-1.4.48-3.el7.x86_64                                                                                                                                                              1/4
  Installing : libtirpc-0.2.4-0.16.el7.x86_64                                                                                                                                                                2/4
  Installing : rpcbind-0.2.0-49.el7.x86_64                                                                                                                                                                   3/4
  Installing : ypserv-2.31-12.el7.x86_64                                                                                                                                                                     4/4
  Verifying  : libtirpc-0.2.4-0.16.el7.x86_64                                                                                                                                                                1/4
  Verifying  : ypserv-2.31-12.el7.x86_64                                                                                                                                                                     2/4
  Verifying  : tokyocabinet-1.4.48-3.el7.x86_64                                                                                                                                                              3/4
  Verifying  : rpcbind-0.2.0-49.el7.x86_64                                                                                                                                                                   4/4

Installed:
  rpcbind.x86_64 0:0.2.0-49.el7                                                                            ypserv.x86_64 0:2.31-12.el7

Dependency Installed:
  libtirpc.x86_64 0:0.2.4-0.16.el7                                                                       tokyocabinet.x86_64 0:1.4.48-3.el7

Complete!
```

### 2. NIS domain 설정

설치가 완료 되었으면 Global NIS domain을 설정한다.

```shell
[root@server ~]$ ypdomainname test.com

# Domain 확인
[root@server ~]$ ypdomainname
test.com
```

Network 상에서도 알아볼 수 있도록 설정해준다.

```shell
[root@server ~]$ vi /etc/sysconfig/network
# Created by anaconda
NISDOMAIN=test.com
```

### 3. 보안 설정

보안을 위해 접속 대상의 IP 대역을 허용한다.  
허용하지 않은 IP 대역은 접속이 불가능하다.

```shell
# 신규 파일 생성
[root@server ~]$ vi /var/yp/securenets
# add IP addresses you allow to access to NIS server
255.0.0.0       127.0.0.0
255.255.255.0   192.168.0.0
```

### 4. /etc/hosts 등록

개요에 적혀있듯이 NIS는 hostname 기반이 기본이다.  
물론 변경은 가능하지만, 기본적인 설치에서는 hosts에 등록하여 domain으로 통신되게 끔 해준다.  
Server 뿐만이 아닌 client까지도 등록 해준다.  
당연하지만 client 세팅할 때는 server를 등록해 줘야 한다.

```shell
[root@server ~]$ vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# add server and clients' IP address for NIS database
192.168.0.55   server.test.com server
192.168.0.59   client.test.com client
```

### 5. Service 관리

NIS Server에 사용되는 service는 다음과 같다.

| 서비스      | 내용                                                       |
| ----------- | ---------------------------------------------------------- |
| `rpcbind`   | **RPC service**                                            |
| `ypserv`    | **NIS Server service**                                     |
| `ypxfrd`    | **Server와 client간 NIS MAP 전송하는 service**             |
| `yppasswdd` | **Clinet에서 비밀번호를 설정 할 수 있도록 해주는 service** |

다음과 같이 service를 start 및 enable 해준다.

```shell
[root@server ~]$ systemctl start rpcbind ypserv ypxfrd yppasswdd
[root@server ~]$ systemctl enable rpcbind ypserv ypxfrd yppasswdd
Created symlink from /etc/systemd/system/multi-user.target.wants/ypserv.service to /usr/lib/systemd/system/ypserv.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/ypxfrd.service to /usr/lib/systemd/system/ypxfrd.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/yppasswdd.service to /usr/lib/systemd/system/yppasswdd.service.
```

### 6. NIS Server DB update

`ypinit`을 통하여 간단히 진행한다.  
만약 hostname이 안 맞는다면 `next host to add` 항목에서 다르게 나올 것이며, 이 곳에서 추가를 할 수 있으나 추가한다면 세팅이 더 필요로 한다.

```shell
[root@server ~]$ /usr/lib64/yp/ypinit -m

At this point, we have to construct a list of the hosts which will run NIS
servers.  server.test.com is in the list of NIS server hosts.  Please continue to add
the names for the other hosts, one per line.  When you are done with the
list, type a <control D>.
        next host to add:  server.test.com
        next host to add:  # Ctrl + D key
The current list of NIS servers looks like this:

server.test.com

Is this correct?  [y/n: y]  y
We need a few minutes to build the databases...
Building /var/yp/test.com/ypservers...
Running /var/yp/Makefile...
gmake[1]: Entering directory `/var/yp/test.com'
Updating passwd.byname...
Updating passwd.byuid...
Updating group.byname...
Updating group.bygid...
Updating hosts.byname...
Updating hosts.byaddr...
Updating rpc.byname...
Updating rpc.bynumber...
Updating services.byname...
Updating services.byservicename...
Updating netid.byname...
Updating protocols.bynumber...
Updating protocols.byname...
Updating mail.aliases...
gmake[1]: Leaving directory `/var/yp/test.com'

server.test.com has been set up as a NIS master server.

Now you can run ypinit -s server.test.com on all slave server.
```

### 7. Test 계정 생성 후 NIS update

Client에서 로그인 테스트를 하기 위하여 test라는 계정을 생성한 뒤 NIS의 정보를 갱신한다.  
NIS는 Linux PAM 계정을 사용한다.  
즉, `useradd`와 같은 계정으로 생성하면 된다.  
단, 생성 하였다면 꼭 `make`를 하여 build 해야 정보가 갱신 된다.

```shell
# test 계정 생성
[root@server ~]$ adduser test

# test 계정 비밀번호 생성
[root@server ~]$ passwd test
Changing password for user test.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

# make build를 통한 NIS update
[root@server ~]$ make -C /var/yp/
make: Entering directory `/var/yp'
gmake[1]: Entering directory `/var/yp/test.com'
Updating passwd.byname...
Updating passwd.byuid...
Updating netid.byname...
gmake[1]: Leaving directory `/var/yp/test.com'
make: Leaving directory `/var/yp'
```

### 8. firewalld에서 port 개방

사용되는 기본 port는 다음과 같다.


| 포트                       | 서비스      |
| -------------------------- | ----------- |
| **976/tcp**<br>**972/udp** | `ypserv`    |
| **971/tcp**<br>**969/udp** | `ypxfrd`    |
| **974/udp**                | `yppasswdd` |


```shell
[root@server ~]$ firewall-cmd --add-service=rpc-bind --permanent
success
[root@server ~]$ firewall-cmd --add-port=976/tcp --permanent
success
[root@server ~]$ firewall-cmd --add-port=972/udp --permanent
success
[root@server ~]$ firewall-cmd --add-port=971/tcp --permanent
success
[root@server ~]$ firewall-cmd --add-port=969/udp --permanent
success
[root@server ~]$ firewall-cmd --add-port=974/udp --permanent
success
[root@server ~]$ firewall-cmd --reload
success

# 확인
[root@server ~]$ firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client rpc-bind ssh
  ports: 976/tcp 972/udp 971/tcp 969/udp 974/udp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

## Options
---

### 1. NIS의 기본 portmap을 변경하여 처리하는 방법

#### 1-1. NIS Portmap 변경

기본적으로 NIS의 port는 `rpcinfo -p | grep -i ypserv`의 명령어를 통해 찾아볼 수 있다.  
보통은 **823/udp,** **827/tcp**를 사용한다.  
이것을 범위내로 통일 하기 위해서는 다음과 같이 작업 해주면 된다.

```shell
# ypserv, ypxfrd 변경
[root@server ~]$ vi /etc/sysconfig/network
# Created by anaconda
NISDOMAIN=test.com

# 추가
YPSERV_ARGS="-p 944"
YPXFRD_ARGS="-p 945"

# yppasswdd 변경
[root@server ~]$ vi /etc/sysconfig/yppasswdd
# The passwd and shadow files are located under the specified
# directory path. rpc.yppasswdd will use these files, not /etc/passwd
# and /etc/shadow.
#ETCDIR=/etc

# This option tells rpc.yppasswdd to use a different source file
# instead of /etc/passwd
# You can't mix usage of this with ETCDIR
#PASSWDFILE=/etc/passwd

# This option tells rpc.yppasswdd to use a different source file
# instead of /etc/passwd.
# You can't mix usage of this with ETCDIR
#SHADOWFILE=/etc/shadow

# Additional arguments passed to yppasswd
## 추가
YPPASSWDD_ARGS="--port 946"

# 적용
[root@server ~]$ systemctl restart rpcbind ypserv ypxfrd yppasswdd
```

#### 1-2. `firewalld`에서 port 개방

위와 같이 변경이 되었다면 필요한 port는 다음과 같다.

| 포트                       | 서비스      |
| -------------------------- | ----------- |
| **944/tcp**<br>**944/udp** | `ypserv`    |
| **945/tcp**<br>**945/udp** | `ypxfrd`    |
| **946/udp**                | `yppasswdd` |

```shell
[root@server ~]$ firewall-cmd --add-service=rpc-bind --permanent
success
[root@server ~]$ firewall-cmd --add-port=944/tcp --permanent
success
[root@server ~]$ firewall-cmd --add-port=944/udp --permanent
success
[root@server ~]$ firewall-cmd --add-port=945/tcp --permanent
success
[root@server ~]$ firewall-cmd --add-port=945/udp --permanent
success
[root@server ~]$ firewall-cmd --add-port=946/udp --permanent
success
[root@server ~]$ firewall-cmd --reload
success

# 확인
[root@server ~]$ firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client rpc-bind ssh
  ports: 944/tcp 944/udp 945/tcp 945/udp 946/udp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```