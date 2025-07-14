---
title: Ubuntu - keepalived 구성
date: 2023-01-31 21:32:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, keepalived]
description: Ubuntu에서 HA(High Availability)에 가장 많이 사용하는 패키지인 keepalived를 구성하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `keepalived`는 주로 `haproxy`와 많이 사용이 되며 server 간 HA(High Availability) 구성을 한다.
* `keepalived`와 `haproxy`는 따로 사용이 가능하며, 보통 `keepalived`는 vIP 목적으로 사용된다.

![keepalived](/assets/img/post/linux/2023-01-31-ubuntu-configure_keepalived/1.png)
_keepalived_

## 설치
---

설치는 보통 `haproxy`와 함께 설치되며 간단하다.
`haproxy`항목은 [Linux - haproxy 구성](/posts/ubuntu-configure_haproxy/)에서 확인이 가능하다.

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

# 설정 및 구성
---

기본적으로 패키지로 설치를 한다 해도 설정 값인 `/etc/keepalived/keepalived.conf`는 비어있다.  
구성이 끝나면 vIP 확인 후 접속 테스트 해보면 된다.

```shell
# conf 파일을 새로 생성 함
dor1@is-ha1:~$ sudo vi /etc/keepalived/keepalived.conf
global_defs {
  notification_email {
  }
  router_id LVS_DEVEL
  vrrp_skip_check_adv_addr
  vrrp_garp_interval 0
  vrrp_gna_interval 0
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  weight 2
}

vrrp_instance haproxy-vip {
  state BACKUP
  priority 100
  # NIC interface 확인
  interface enp6s18
  virtual_router_id 60
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  # 구성 서버의 IP (만약 is-ha1이 아닌 is-ha2와 같이 다른 서버라면 is-ha2 서버의 IP를 입력해 줌)
  unicast_src_ip 172.16.1.201
  # 구성 서버가 장애난 경우 다음 서버의 IP로 forwarding
  unicast_peer {
    172.16.1.202
  }
  # vIP 입력
  virtual_ipaddress {
    10.1.20.100/16
  }
  track_script {
    chk_haproxy
  }
}

# 서비스 시작
dor1@is-ha1:~$ sudo systemctl start keepalived.service

# 서비스 확인
dor1@is-ha1:~$ sudo systemctl status keepalived.service
● keepalived.service - Keepalive Daemon (LVS and VRRP)
     Loaded: loaded (/lib/systemd/system/keepalived.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-01-30 14:40:20 KST; 20h ago
   Main PID: 286585 (keepalived)
      Tasks: 2 (limit: 4535)
     Memory: 2.3M
        CPU: 3min 7.914s
     CGroup: /system.slice/keepalived.service
             ├─286585 /usr/sbin/keepalived --dont-fork
             └─286586 /usr/sbin/keepalived --dont-fork

Jan 30 14:40:20 is-ha1 systemd[1]: keepalived.service: Got notification message from PID 286586, but reception only permitted for main PID 286585
Jan 30 14:40:20 is-ha1 Keepalived_vrrp[286586]: WARNING - default user 'keepalived_script' for script execution does not exist - please create.
Jan 30 14:40:20 is-ha1 Keepalived_vrrp[286586]: WARNING - script `killall` resolved by path search to `/usr/bin/killall`. Please specify full path.
Jan 30 14:40:20 is-ha1 systemd[1]: Started Keepalive Daemon (LVS and VRRP).
Jan 30 14:40:20 is-ha1 Keepalived[286585]: Startup complete
Jan 30 14:40:20 is-ha1 Keepalived_vrrp[286586]: SECURITY VIOLATION - scripts are being executed but script_security not enabled.
Jan 30 14:40:20 is-ha1 Keepalived_vrrp[286586]: (haproxy-vip) Entering BACKUP STATE (init)
Jan 30 14:40:20 is-ha1 Keepalived_vrrp[286586]: VRRP_Script(chk_haproxy) succeeded
Jan 30 14:40:20 is-ha1 Keepalived_vrrp[286586]: (haproxy-vip) Changing effective priority from 100 to 102
Jan 30 14:41:10 is-ha1 Keepalived_vrrp[286586]: (haproxy-vip) Entering MASTER STATE

# 서비스 부팅 시 시작
dor1@is-ha1:~$ sudo systemctl enable keepalived.service
Synchronizing state of keepalived.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable keepalived

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
    # vIP 
    inet 10.1.20.100/16 scope global enp6s18
       valid_lft forever preferred_lft forever
    inet6 fe80::b01a:45ff:fe7b:ca3f/64 scope link
       valid_lft forever preferred_lft forever
```