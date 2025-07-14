---
title: Linux - iptables 알아보기
date: 2022-07-22 11:22:00 +0900
categories: [Linux]
tags: [linux, iptables]
description: Linux의 Host 네트워크 중 가장 기본인 iptables를 알아보았다.
---

>대부분의 Linux 배포판
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `iptables`를 이용한 Port forwarding 설정이 가능하다.
* `iptables`를 통하여 SDN(Software Defined Network) 구성 및 Routing 가능하다.

## 개념
---

보통 `/sbin/iptables`의 binary를 통하여 작업 진행 한다.

### 1. 테이블(Table)의 종류

| 종류     | 내용                                                                                   |
| -------- | -------------------------------------------------------------------------------------- |
| `filter` | 기본 방화벽 관련 작업이 가능 함                                                        |
| `nat`    | 새로운 연결이나 NAT 이하에 네트워크 연결이 필요(Port Forwarding)할 때 사용             |
| `mangle` | TTL, TOS(Type of Service, 패킷 우선 순위) 변경과 같은 특수 rules를 구성할 때 사용      |
| `raw`    | Connection Tracking 기능에 대해 자세한 설정이나 exception과 같은 기능이 필요할 때 사용 |

### 2. 체인(Chain)의 종류

| 종류          | 내용                                                                                                                  |
| ------------- | --------------------------------------------------------------------------------------------------------------------- |
| `INPUT`       | 패킷이 네트워크로부터 오거나 서버로 갈 때 사용, host로 향하는 모든 패킷을 의미                                        |
| `OUTPUT`      | 패킷이 서버로부터 시작되거나 네트워크로 나갈 때 사용, host에서 발생하는 모든 패킷을 의미                              |
| `FORWARD`     | 패킷이 서버에 의해 전달 되거나 네트워크 간 routing이 필요 할 시 사용, routing 되고 local 전달이 아닌 모든 패킷을 의미 |
| `PREROUTING`  | Routing 전 사용                                                                                                       |
| `POSTROUTING` | Routing 후 사용                                                                                                       |

### 3. iptables 구성 확인 및 삭제

Chain의 번호는 옵션에 `--line-numbers` 를 붙여주면 된다.

```shell
# iptables 모든 체인(Chain) 확인
root@dgx-1p:~$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

# iptables 모든 NAT 체인(Chain) 확인
root@dgx-1p:~$ sudo iptables -nL -t nat

Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination

# 특정 체인(Chain) 삭제
iptables -D <chain_name> <number>

# 특정 NAT 체인(Chain) 삭제
iptables -t nat -D <chain_name> <number>

# !!사용시 주의!! iptables 체인(Chain) 모두 삭제
iptables -F
```

### 4. iptables 저장

`iptables`는 특성상 rebooting하면 설정들이 날라가게 된다.  
`netfilter-persistent`의 패키지를 통하여 저장해주면 rebooting 후에도 설정이 남아있다.  
`NAT` 같은 경우에는 `iptables-persistent` 패키지를 통하여 저장이 가능하다.

```shell
root@dgx-1p:~$ apt install netfilter-persistent
Reading package lists... Done
Building dependency tree
Reading state information... Done
Suggested packages:
  iptables-persistent
The following NEW packages will be installed:
  netfilter-persistent
0 upgraded, 1 newly installed, 0 to remove and 15 not upgraded.
Need to get 7,268 B of archives.
After this operation, 38.9 kB of additional disk space will be used.
Get:1 http://mirror.kakao.com/ubuntu focal-updates/universe amd64 netfilter-persistent all 1.0.14ubuntu1 [7,268 B]
Fetched 7,268 B in 0s (72.9 kB/s)
Selecting previously unselected package netfilter-persistent.
(Reading database ... 118601 files and directories currently installed.)
Preparing to unpack .../netfilter-persistent_1.0.14ubuntu1_all.deb ...
Unpacking netfilter-persistent (1.0.14ubuntu1) ...
Setting up netfilter-persistent (1.0.14ubuntu1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/netfilter-persistent.service → /lib/systemd/system/netfilter-persistent.service.
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for systemd (245.4-4ubuntu3.16) ...

root@dgx-1p:~$ netfilter-persistent save
```

## 예제
---

```shell
# INPUT 패킷 중 TCP이며 80번 포트에 대해 추가(Append)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 192.168.11.1의 INPUT 패킷 중 TCP이며 80 포트에 대해 추가(Append)
iptables -A INPUT -s 192.168.11.1 -p tcp --dport 80 -j ACCEPT

# INPUT 패킷 중 TCP이며 80번 포트에 대해 5번째 line에 추가(Insert)
iptables -I INPUT 5 -p tcp --dport 50001 -j ACCEPT

# INPUT 패킷 중 2번째 line을 삭제(Delete)
iptables -D INPUT 2
```

### 1. VM 사용 예제

```shell
# 192.168.122.31 VM의 ssh(22) 포트를 host 서버에 PREROUTING
root@dgx-1p:~$ iptables -t nat -A PREROUTING -d 192.168.11.220/24 -i enp1s0f0 -p tcp --dport 65000 -j DNAT --to-destination 192.168.122.31:22

# 이후 192.168.122.31 VM의 ssh(22) 포트를 host 서버에 FORWARD
root@dgx-1p:~$ iptables -I FORWARD -p tcp -d 192.168.122.31/24 --dport 22 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
```