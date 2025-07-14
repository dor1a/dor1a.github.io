---
title: Ubuntu - nmap 사용 해보기
date: 2023-09-10 18:29:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, nmap]
description: Linux에서 강력한 Network mapper를 사용 해보았다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `nmap`은 **N**etwork **MAP**per의 약자로 network port, host, os info 등 정보를 scan 할 수 있다.
* Scan이 된다는 의미는 보안 적으로 봤을 때, 공격을 의미할 수 있으며 아무런 생각 없이 날린 packet이 서비스에 장애를 일으킬 수도 있다.
* 대다수 Linux 배포판에서 간편하게 설치 된다.  

## 설치
---

설치는 대다수 Linux 배포판 repository에 존재하기에 한 줄로 설치가 가능하다.

```shell
# install nmap
dor1@dor1-lxc-provisioning ~ ❯ sudo apt install nmap
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libblas3 liblinear4 liblua5.3-0 lua-lpeg nmap-common
Suggested packages:
  liblinear-tools liblinear-dev ncat ndiff zenmap
The following NEW packages will be installed:
  libblas3 liblinear4 liblua5.3-0 lua-lpeg nmap nmap-common
0 upgraded, 6 newly installed, 0 to remove and 2 not upgraded.
Need to get 6113 kB of archives.
After this operation, 26.8 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://mirror.kakao.com/ubuntu jammy/main amd64 libblas3 amd64 3.10.0-2ubuntu1 [228 kB]
Get:2 http://mirror.kakao.com/ubuntu jammy/universe amd64 liblinear4 amd64 2.3.0+dfsg-5 [41.4 kB]
Get:3 http://mirror.kakao.com/ubuntu jammy/main amd64 liblua5.3-0 amd64 5.3.6-1build1 [140 kB]
Get:4 http://mirror.kakao.com/ubuntu jammy/universe amd64 lua-lpeg amd64 1.0.2-1 [31.4 kB]
Get:5 http://mirror.kakao.com/ubuntu jammy-updates/universe amd64 nmap-common all 7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1 [3940 kB]
Get:6 http://mirror.kakao.com/ubuntu jammy-updates/universe amd64 nmap amd64 7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1 [1731 kB]
Fetched 6113 kB in 0s (23.7 MB/s)
Selecting previously unselected package libblas3:amd64.
(Reading database ... 59557 files and directories currently installed.)
Preparing to unpack .../0-libblas3_3.10.0-2ubuntu1_amd64.deb ...
Unpacking libblas3:amd64 (3.10.0-2ubuntu1) ...
Selecting previously unselected package liblinear4:amd64.
Preparing to unpack .../1-liblinear4_2.3.0+dfsg-5_amd64.deb ...
Unpacking liblinear4:amd64 (2.3.0+dfsg-5) ...
Selecting previously unselected package liblua5.3-0:amd64.
Preparing to unpack .../2-liblua5.3-0_5.3.6-1build1_amd64.deb ...
Unpacking liblua5.3-0:amd64 (5.3.6-1build1) ...
Selecting previously unselected package lua-lpeg:amd64.
Preparing to unpack .../3-lua-lpeg_1.0.2-1_amd64.deb ...
Unpacking lua-lpeg:amd64 (1.0.2-1) ...
Selecting previously unselected package nmap-common.
Preparing to unpack .../4-nmap-common_7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1_all.deb ...
Unpacking nmap-common (7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1) ...
Selecting previously unselected package nmap.
Preparing to unpack .../5-nmap_7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1_amd64.deb ...
Unpacking nmap (7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1) ...
Setting up lua-lpeg:amd64 (1.0.2-1) ...
Setting up libblas3:amd64 (3.10.0-2ubuntu1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/blas/libblas.so.3 to provide /usr/lib/x86_64-linux-gnu/libblas.so.3 (libblas.so.3-x86_64-linux-gnu) in auto mode
Setting up nmap-common (7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1) ...
Setting up liblua5.3-0:amd64 (5.3.6-1build1) ...
Setting up liblinear4:amd64 (2.3.0+dfsg-5) ...
Setting up nmap (7.91+dfsg1+really7.80+dfsg1-2ubuntu0.1) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.3) ...
```

## 사용법
---

찾아보면 여러가지 사용 법이 있다.  
이 중 핵심적으로 필요한 부분에 대해서는 지속적으로 기록 할 예정이다.

### 1. TDP Scan

```shell
# TCP [nmap -sT -p <port> <ip>]
dor1@dor1-lxc-provisioning ~ ❯ sudo nmap -sT -p 27015 211.201.59.187
Starting Nmap 7.80 ( https://nmap.org ) at 2023-09-26 22:43 KST
Nmap scan report for 211.201.59.187
Host is up (0.00082s latency).

PORT      STATE SERVICE
27015/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

### 2. UDP Scan

```shell
# UDP [nmap -sU -p <port> <ip>]
dor1@dor1-lxc-provisioning ~ ❯ sudo nmap -sU -p 51820 211.201.59.187
Starting Nmap 7.80 ( https://nmap.org ) at 2023-09-10 18:22 KST
Nmap scan report for 211.201.59.187
Host is up (0.00078s latency).

PORT      STATE         SERVICE
51820/udp open|filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds
```