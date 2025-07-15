---
title: Debian - DNS Client 설정
date: 2024-11-04 14:36:00 +0900
categories: [Linux, Debian]
tags: [linux, debian, dns, openmediavault]
description: Debian에서 DNS Client 설정을 해보는 방법이다.
---

>Debian GNU/Linux 11 (bullseye)
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* DNS Server가 구축이 되어있다는 가정하에, client 설정하는 방법에 대해서 설명한다.

## 1. /etc/resolv.conf
---

`/etc/resolv.conf`는 일반적으로 가장 많이 사용하는 설정이다.  
매우 간단하게 설정이 가능하며, 서비스에 대해 재시작도 안해도(kernel 단에서 지속 확인) 된다.

```shell
dor1@Atsui ~ ❯ cat /etc/resolv.conf
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

## nameserver에 DNS Server를 적어주면 되며, 특정 host를 찾으려면 search(ex. tera.echidna-cloud.net)에 적어주면 됨
nameserver 127.0.0.53
options edns0 trust-ad
search localhost
```

## 2. OMV(OpenMediaVault)와 같이 특정 OS에서 resolv.conf가 작동하지 않을 경우
---

OMV에서는 `salt`라는 automation tool로 서비스들이 배포되기 때문에 `/etc/resolv.conf`가 정상적으로 작동하지 않는다.  
웹에서 네트워크 설정을 할 경우에도 따로 들어가지는 않게 되기 때문에 다른 방법을 사용해야한다.  
만약에 강제로 `/etc/resolv.conf`에 입력하여 사용한다해도 변경될 수 있으며, `avahi-daemon`에서 잡고있는 multicast 된 DNS를 우선적으로 가져오기 때문에 최상위로 올리는 방법을 고려해야한다.  
Linux에서 `systemd`의 `resolved` 서비스가 DNS 서비스 중 가장 최우선 적으로 바라보기에 여기에 먼저 설정을 해주면된다.

예를들어 `tailscale`를 사용하기 위해 magicDNS(100.100.100.100)을 써야할 경우 다음과 같이 설정 해주면 된다.

```shell
# resolved.conf가 바라보는 conf.d 디렉토리
dor1@Atsui ~ ❯ cd /etc/systemd/resolved.conf.d
dor1@Atsui ~ ❯ ll
total 8
drwxr-xr-x 1 root root  88 Sep  5 17:47 .
drwxr-xr-x 1 root root 300 Sep  3 08:28 ..
-rw-r--r-- 1 root root  59 Sep  5 15:32 99-openmediavault-mdns.conf

# 설정 파일 작성
dor1@Atsui ~ ❯ sudo vi 10-tailscale.conf
[Resolve]
DNS=100.100.100.100
Domains=echidna-cloud.ts.net
```

작성 후 아래와 같이 서비스를 재시작한다.

```shell
# 서비스 재시작
dor1@Atsui ~ ❯ sudo systemctl restart systemd-resolved.service

# 서비스 확인
dor1@Atsui ~ ❯ sudo systemctl status systemd-resolved.service
● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-11-01 18:15:25 KST; 2 days ago
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
   Main PID: 2040 (systemd-resolve)
     Status: "Processing requests..."
      Tasks: 1 (limit: 154310)
     Memory: 11.8M
        CPU: 2min 6.608s
     CGroup: /system.slice/systemd-resolved.service
             └─2040 /lib/systemd/systemd-resolved

Nov 01 18:15:25 Atsui systemd-resolved[2040]: Positive Trust Anchors:
Nov 01 18:15:25 Atsui systemd-resolved[2040]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc683457104237c7f8ec8d
Nov 01 18:15:25 Atsui systemd-resolved[2040]: Negative trust anchors: 10.in-addr.arpa 16.172.in-addr.arpa 17.172.in-addr.arpa 18.172.in-addr.arpa 19.172>
Nov 01 18:15:25 Atsui systemd-resolved[2040]: Using system hostname 'Atsui'.
Nov 01 18:15:25 Atsui systemd[1]: Started Network Name Resolution.
Nov 01 18:15:35 Atsui systemd-resolved[2040]: Using degraded feature set UDP instead of UDP+EDNS0 for DNS server 10.0.0.1.
Nov 01 18:27:20 Atsui systemd-resolved[2040]: Grace period over, resuming full feature set (UDP+EDNS0) for DNS server 10.0.0.1.
Nov 01 18:27:21 Atsui systemd-resolved[2040]: Using degraded feature set UDP instead of UDP+EDNS0 for DNS server 10.0.0.1.
Nov 01 18:37:22 Atsui systemd-resolved[2040]: Grace period over, resuming full feature set (UDP+EDNS0) for DNS server 10.0.0.1.
Nov 01 18:37:22 Atsui systemd-resolved[2040]: Using degraded feature set UDP instead of UDP+EDNS0 for DNS server 10.0.0.1.

# DNS 확인
dor1@Atsui ~ ❯ resolvectl
Global
         Protocols: +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub
Current DNS Server: 100.100.100.100
       DNS Servers: 100.100.100.100
        DNS Domain: echidna-cloud.ts.net

Link 2 (enp6s0)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 3 (eno1)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 4 (enp2s0)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 5 (enp2s0d1)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 7 (bond0)
    Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 10.0.0.1
       DNS Servers: 10.0.0.1

Link 8 (bond1)
Current Scopes: LLMNR/IPv4 LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 9 (docker0)
Current Scopes: LLMNR/IPv4 LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 10 (tailscale0)
Current Scopes: LLMNR/IPv4 LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 12 (vethc51e4a4)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 13 (docker-backend)
Current Scopes: LLMNR/IPv4 LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 15 (vethbee64c6)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 17 (vethf09e704)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 18 (docker-service)
Current Scopes: LLMNR/IPv4 LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 20 (veth8f1a2f9)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 22 (vethd45af52)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 24 (vethb4c26d8)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 26 (vethbb8af54)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 28 (vethe18c9eb)
Current Scopes: LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 29 (enxea27b1fe0a8d)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
```

Global 항목이 최우선 적으로 적용 되기 때문에 정상적으로 DNS query를 가져오게 된다.

```shell
# ping test
dor1@Atsui ~ ❯ ping tera
PING tera.echidna-cloud.ts.net (100.64.1.1) 56(84) bytes of data.
64 bytes from tera.echidna-cloud.ts.net (100.64.1.1): icmp_seq=1 ttl=64 time=148 ms
64 bytes from tera.echidna-cloud.ts.net (100.64.1.1): icmp_seq=2 ttl=64 time=151 ms
64 bytes from tera.echidna-cloud.ts.net (100.64.1.1): icmp_seq=3 ttl=64 time=148 ms

--- tera.echidna-cloud.ts.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 148.284/149.190/150.998/1.278 ms
```