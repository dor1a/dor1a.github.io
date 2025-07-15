---
title: Ubuntu - haproxy 구성
date: 2023-01-31 09:30:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, haproxy]
description: Ubuntu에서 L7으로 사용하는 Proxy server인 haproxy의 구성하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `haproxy`는 Proxy의 일종이며, Load balance의 역할로 사용된다.
* Proxy의 쓰임새는 Client의 access root는 보통 내부에서 우회(Outbound 접근) 할 때 많이 사용하지만, Reverse Proxy는 역으로 외부에서 내부(Inbound) 접근 시에 사용된다.
* `haproxy`는 보통 여러 node에 대상으로 접근 시 LB(Load Balancing)를 위해 사용되며 performance 또한 다른 proxy server 대비 우수하다.

![haproxy](/assets/img/post/linux/ubuntu/2023-01-31-ubuntu-configure_haproxy/1.png)
_haproxy_

## 설치
---

설치는 간단하다.  
보통 `haproxy`는 LB에 목적을 두고 있어 보통 `keepalived`와 같이 사용된다.(물론 목적 상 따로 사용 가능)  
`keepalived`는 [Linux - keepalived 구성](/posts/ubuntu-configure_keepalived/) 항목을 참조하면 된다.

```shell
# psmisc는 process 관리를 위해 설치 함
dor1@is-ha1:~$ sudo apt install haproxy keepalived psmisc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
psmisc is already the newest version (23.4-2build3).
Suggested packages:
  vim-haproxy haproxy-doc
The following NEW packages will be installed:
  haproxy keepalived
0 upgraded, 2 newly installed, 0 to remove and 87 not upgraded.
Need to get 2,092 kB of archives.
After this operation, 4,998 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://mirror.kakao.com/ubuntu jammy-security/main amd64 haproxy amd64 2.4.18-0ubuntu1.1 [1,639 kB]
Get:2 http://mirror.kakao.com/ubuntu jammy/main amd64 keepalived amd64 1:2.2.4-0.2build1 [453 kB]
Fetched 2,092 kB in 0s (6,303 kB/s)
Selecting previously unselected package haproxy.
(Reading database ... 115965 files and directories currently installed.)
Preparing to unpack .../haproxy_2.4.18-0ubuntu1.1_amd64.deb ...
Unpacking haproxy (2.4.18-0ubuntu1.1) ...
Selecting previously unselected package keepalived.
Preparing to unpack .../keepalived_1%3a2.2.4-0.2build1_amd64.deb ...
Unpacking keepalived (1:2.2.4-0.2build1) ...
Setting up keepalived (1:2.2.4-0.2build1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /lib/systemd/system/keepalived.service.
Setting up haproxy (2.4.18-0ubuntu1.1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/haproxy.service → /lib/systemd/system/haproxy.service.
Processing triggers for dbus (1.12.20-2ubuntu4.1) ...
Processing triggers for rsyslog (8.2112.0-2ubuntu2.2) ...
Processing triggers for man-db (2.10.2-1) ...
Scanning processes...
Scanning candidates...
Scanning processor microcode...
Scanning linux images...

Running kernel seems to be up-to-date.

Failed to check for processor microcode upgrades.

Restarting services...
Service restarts being deferred:
 systemctl restart cron.service
 systemctl restart ssh.service
 systemctl restart systemd-journald.service
 systemctl restart systemd-logind.service
 /etc/needrestart/restart.d/systemd-manager
 systemctl restart systemd-networkd.service
 systemctl restart systemd-resolved.service
 systemctl restart user@1000.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

## TCP Load balancing 설정 해보기
---

Testbed는 HA 1대로 k8s의 마스터를 proxy하는 구조다.  
만약 2대 node 이상을 세팅하려면 `/etc/haproxy/haproxy.cfg`세팅 파일을 하나의 storage에 넣어두고 mount하여 세팅해도 좋을 법하다.
* is-ha1 (haproxy)
* is-mn1 (k8s master)

```shell
# 기본 설정값
dor1@is-ha1:~$ cat /etc/haproxy/haproxy.cfg
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http
```

위와 같은 기본 설정 값을 변경하여 is-m1의 nginx(NodePort 32222)를 forwarding 한다.

```shell
# k8s의 nginx 상태 확인
dor1@is-mn1:~$ kubectl get svc -A
NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.233.0.1    <none>        443/TCP                  12d
default       my-nginx     NodePort    10.233.0.38   <none>        8080:32222/TCP           5d23h
kube-system   coredns      ClusterIP   10.233.0.3    <none>        53/UDP,53/TCP,9153/TCP   12d

# IP 확인
dor1@is-ha1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether b2:1a:45:7b:ca:3f brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.201/24 brd 172.16.1.255 scope global enp6s18
       valid_lft forever preferred_lft forever
    # keepalived를 통하여 만들어진 vIP
    inet 10.1.20.100/16 scope global enp6s18
       valid_lft forever preferred_lft forever
    inet6 fe80::b01a:45ff:fe7b:ca3f/64 scope link
       valid_lft forever preferred_lft forever

# 변경 설정 값
dor1@is-ha1:~$ cat /etc/haproxy/haproxy.cfg
global
  daemon
  maxconn 50000
  log /dev/log local0
  chroot /var/lib/haproxy
  stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
  stats timeout 30s
  user haproxy
  group haproxy
  pidfile /var/run/haproxy.pid

defaults
  log global
  mode http

  option httplog
  option dontlognull
  option http-server-close

  timeout http-request 10s
  timeout client 20s
  timeout connect 4s
  timeout server 30s
  timeout http-keep-alive 10s

  errorfile 400 /etc/haproxy/errors/400.http
  errorfile 403 /etc/haproxy/errors/403.http
  errorfile 408 /etc/haproxy/errors/408.http
  errorfile 500 /etc/haproxy/errors/500.http
  errorfile 502 /etc/haproxy/errors/502.http
  errorfile 503 /etc/haproxy/errors/503.http
  errorfile 504 /etc/haproxy/errors/504.http

#---------------------- listen --------------------#

listen stats
  bind :9090
  stats enable
  stats realm Haproxy\ Statistics
  stats uri /

listen testApp1
  bind :32222
  balance roundrobin
  mode tcp
  server is-mn1 172.16.1.101:32222 check

# 서비스 재시작
dor1@is-ha1:~$ sudo systemctl restart haproxy.service

# 서비스 확인
dor1@is-ha1:~$ sudo systemctl status haproxy.service
● haproxy.service - HAProxy Load Balancer
     Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-01-31 10:37:58 KST; 1h 2min ago
       Docs: man:haproxy(1)
             file:/usr/share/doc/haproxy/configuration.txt.gz
    Process: 323664 ExecStartPre=/usr/sbin/haproxy -Ws -f $CONFIG -c -q $EXTRAOPTS (code=exited, status=0/SUCCESS)
   Main PID: 323666 (haproxy)
      Tasks: 5 (limit: 4535)
     Memory: 16.5M
        CPU: 1.964s
     CGroup: /system.slice/haproxy.service
             ├─323666 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
             └─323669 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

Jan 31 10:37:57 is-ha1 systemd[1]: Starting HAProxy Load Balancer...
Jan 31 10:37:58 is-ha1 haproxy[323666]: [WARNING]  (323666) : parsing [/etc/haproxy/haproxy.cfg:16] : 'option httplog' not usable with proxy 'testApp1' (needs 'mode http'). Falling back to 'option tcplog'.
Jan 31 10:37:58 is-ha1 haproxy[323666]: [NOTICE]   (323666) : New worker #1 (323669) forked
Jan 31 10:37:58 is-ha1 systemd[1]: Started HAProxy Load Balancer.
```

중요한 건 `listen`의 항목인데, `traefik`, `nginx prxoy`등을 만져봤다면 비슷하다는 것을 볼 수 있다.  
`listen`의 항목은 `frontend`와 `backend`를 합친 항목으로 생각하면 된다.  
구체적인 설정 방법은 아래의 링크를 통해 확인이 가능하다  
<https://www.haproxy.com/documentation/hapee/latest/configuration/config-sections/listen/>

![vIP(10.1.20.100)로 구성](/assets/img/post/linux/ubuntu/2023-01-31-ubuntu-configure_haproxy/2.png)
_vIP(10.1.20.100)로 구성_

![is-mn1(172.16.1.101)으로도 확인 가능](/assets/img/post/linux/ubuntu/2023-01-31-ubuntu-configure_haproxy/3.png)
_is-mn1(172.16.1.101)으로도 확인 가능_

결과적으로 `10.1.20.100`의 `vIP`를 통하여 `is-mn1(172.16.1.101)`의 `32222`포트가 proxy 되는 것을 확인 할 수 있다.  
또한 꼭 `keepalived`를 통한 `vIP`가 아니더라도 `is-mn1(172.16.1.101)`를 통해 `32222`포트를 확인 할 수 있다.