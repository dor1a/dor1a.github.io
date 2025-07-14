---
title: CentOS - NIS Client 설정
date: 2024-04-28 22:15:00 +0900
categories: [Linux, CentOS]
tags: [linux, centos, nis]
description: CentOS에서 NIS(Network Information Service)의 client를 설정하는 방법이다.
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
* Server 설정은 다음과 같은 link로 가면 된다.  
  [Linux(CentOS) - NIS Server 구축](/posts/centos-build_nis_server)

![NIS Server와 Client 구성도](/assets/img/post/linux/2024-04-28-centos-setup_nis_client/1.png)
_NIS Server와 Client 구성도_

## 설치 및 세팅
---

### 1. NIS Client 패키지 설치

NIS Client 패키지는 `ypbind, rpcbind, yp-tools` 세가지 패키지로 이뤄져있다.  
세가지 패키지는 다음과 같이 사용된다.

| 패키지     | 내용                                                                                                                                                        |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ypbind`   | **NIS Client의 Daemon(service) 패키지이다.**                                                                                                                |
| `rpcbind`  | **Remote Procedure Call (RPC) 서비스를 지원하는 패키지입니다.**<br>**RPC는 네트워크를 통해 다른 컴퓨터에서 프로그램을 실행할 수 있게 해주는 protocol이다.** |
| `yp-tools` | **NIS 서비스를 사용하기 위해 필요한 client 유틸리티 모음이다.**                                                                                             |

```shell
[root@client ~]$ yum install ypbind rpcbind yp-tools -y
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
(3/4): base/7/x86_64/primary_db                                                                                                                                                           | 6.1 MB  00:00:00
(4/4): updates/7/x86_64/primary_db                                                                                                                                                        |  26 MB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package rpcbind.x86_64 0:0.2.0-49.el7 will be installed
--> Processing Dependency: libtirpc >= 0.2.4-0.7 for package: rpcbind-0.2.0-49.el7.x86_64
--> Processing Dependency: libtirpc.so.1()(64bit) for package: rpcbind-0.2.0-49.el7.x86_64
---> Package yp-tools.x86_64 0:2.14-5.el7 will be installed
---> Package ypbind.x86_64 3:1.37.1-9.el7 will be installed
--> Running transaction check
---> Package libtirpc.x86_64 0:0.2.4-0.16.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================================================================================================================
 Package                                           Arch                                            Version                                                   Repository                                     Size
=================================================================================================================================================================================================================
Installing:
 rpcbind                                           x86_64                                          0.2.0-49.el7                                              base                                           60 k
 yp-tools                                          x86_64                                          2.14-5.el7                                                base                                           79 k
 ypbind                                            x86_64                                          3:1.37.1-9.el7                                            base                                           62 k
Installing for dependencies:
 libtirpc                                          x86_64                                          0.2.4-0.16.el7                                            base                                           89 k

Transaction Summary
=================================================================================================================================================================================================================
Install  3 Packages (+1 Dependent package)

Total download size: 291 k
Installed size: 583 k
Downloading packages:
warning: /var/cache/yum/x86_64/7/base/packages/rpcbind-0.2.0-49.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for rpcbind-0.2.0-49.el7.x86_64.rpm is not installed
(1/4): rpcbind-0.2.0-49.el7.x86_64.rpm                                                                                                                                                    |  60 kB  00:00:00
(2/4): yp-tools-2.14-5.el7.x86_64.rpm                                                                                                                                                     |  79 kB  00:00:00
(3/4): libtirpc-0.2.4-0.16.el7.x86_64.rpm                                                                                                                                                 |  89 kB  00:00:00
(4/4): ypbind-1.37.1-9.el7.x86_64.rpm                                                                                                                                                     |  62 kB  00:00:00
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                            1.5 MB/s | 291 kB  00:00:00
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
  Installing : libtirpc-0.2.4-0.16.el7.x86_64                                                                                                                                                                1/4
  Installing : rpcbind-0.2.0-49.el7.x86_64                                                                                                                                                                   2/4
  Installing : yp-tools-2.14-5.el7.x86_64                                                                                                                                                                    3/4
  Installing : 3:ypbind-1.37.1-9.el7.x86_64                                                                                                                                                                  4/4
  Verifying  : 3:ypbind-1.37.1-9.el7.x86_64                                                                                                                                                                  1/4
  Verifying  : libtirpc-0.2.4-0.16.el7.x86_64                                                                                                                                                                2/4
  Verifying  : yp-tools-2.14-5.el7.x86_64                                                                                                                                                                    3/4
  Verifying  : rpcbind-0.2.0-49.el7.x86_64                                                                                                                                                                   4/4

Installed:
  rpcbind.x86_64 0:0.2.0-49.el7                                         yp-tools.x86_64 0:2.14-5.el7                                         ypbind.x86_64 3:1.37.1-9.el7

Dependency Installed:
  libtirpc.x86_64 0:0.2.4-0.16.el7

Complete!
```

### 2. NIS domain 설정

설치가 완료 되었으면 Global NIS domain을 설정한다.

```shell
[root@client ~]$ ypdomainname test.com

# Domain 확인
[root@client ~]$ ypdomainname
test.com
```

Network 상에서도 알아볼 수 있도록 설정해준다.

```shell
[root@client ~]$ vi /etc/sysconfig/network
# Created by anaconda
NISDOMAIN=test.com
```

### 3. `/etc/hosts` 등록

개요에 적혀있듯이 NIS는 hostname 기반이 기본이다.  
Client 세팅할 때는 server를 등록해 줘야 한다.

```shell
[root@client ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# add server IP address for NIS database
192.168.0.55   server.test.com server
```

### 4. `authconfig`를 통한 계정 정보 설정

Red hat 계열 OS에서는`authconfig`라는 것을 제공 해준다.  
이 기능으로 간편하게 계정 설정을 할 수 있다.

```shell
[root@client ~]$ authconfig --enablenis --nisdomain=test.com --nisserver=server.test.com --enablemkhomedir --update
```

### 5. Service start & enable

service를 start & enable 해준다.

```shell
[root@client ~]$ systemctl start rpcbind ypbind
[root@client ~]$ systemctl enable rpcbind ypbind
```

### 6. Test 계정 로그인

`ypcat`의 명령어를 통하여 server에 들어가있는 계정 정보를 확인 해본다.

```shell
[root@client ~]$ ypcat passwd
test:$6$EX6J/FxX$J/Dhox0fTV0b4Qh8w18GuLvFSS5SdklpwqLkPQwHQNhLRo4c38a3MeHZ8kl4/5aYZyntuGVc7ciND3SIkjoyt.:1000:1000::/home/test:/bin/bas

[root@client ~]$ ypcat group
test:!:1000:
```

계정이 확인이 되면 로그인하여 확인해본다.

```shell
[root@client ~]$ su test
[test@client root]$ cd /home/test
[test@client ~]$ ls -alh
total 12K
drwx------. 2 test test  62 Apr 27 18:41 .
drwxr-xr-x. 3 root root  18 Apr 27 18:41 ..
-rw-------. 1 test test  18 Apr 27 18:41 .bash_logout
-rw-------. 1 test test 193 Apr 27 18:41 .bash_profile
-rw-------. 1 test test 231 Apr 27 18:41 .bashrc
```

정상적으로 확인이 된다면, `passwd` or `yppasswd`의 명령어를 통하여 비밀번호를 변경 해본다.

```shell
# passwd를 통한 변경
[test@client ~]$ passwd
Changing password for user test.
Changing password for test.
(current) UNIX password:
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

# yppasswd를 통한 변경
[test@client ~]$ yppasswd
Changing NIS account information for test on server.test.com.
Please enter old password:
Changing NIS password for test on server.test.com.
Please enter new password:
Please retype new password:

The NIS password has been changed on server.test.com.
```