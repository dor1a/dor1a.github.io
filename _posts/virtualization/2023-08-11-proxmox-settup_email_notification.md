---
title: Proxmox - E-Mail notification 설정
date: 2023-08-11 09:44:00 +0900
categories: [virualization]
tags: [virualization, proxmox, notification]
description: Proxmox의 Hypervisor에서 E-Mail Notification을 설정하는 방법이다.
---

>Proxmox Virtual Environment 7.4-3
{: .prompt-info}

>Hypervisor
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Proxmox에서 장애 또는 Backup에 대한 alert을 받기 위해서 진행하게 되었다.
* 해당 글에서는 `Microsoft Office365`를 대상으로 설정하였다.

## E-Mail notification 설정방법
---

Proxmox는 Linux이기 때문에 `postfix`의 패키지를 통해서 E-Mail을 전달 한다.  
기본적으로 SSL(TLS)가 있는 SMTP 서버 같은 경우엔 Linux의 `sasl` module을 이용이 필요하다.

```shell
# libsasl2-module 설치
root@is-pve1:~$ apt install libsasl2-modules
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Suggested packages:
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql
The following NEW packages will be installed:
  libsasl2-modules
0 upgraded, 1 newly installed, 0 to remove and 75 not upgraded.
Need to get 104 kB of archives.
After this operation, 274 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 libsasl2-modules amd64 2.1.27+dfsg-2.1+deb11u1 [104 kB]
Fetched 104 kB in 0s (269 kB/s)
Selecting previously unselected package libsasl2-modules:amd64.
(Reading database ... 47545 files and directories currently installed.)
Preparing to unpack .../libsasl2-modules_2.1.27+dfsg-2.1+deb11u1_amd64.deb ...
Unpacking libsasl2-modules:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
Setting up libsasl2-modules:amd64 (2.1.27+dfsg-2.1+deb11u1) ...
```

`sasl` module을 설치 했다면, 본격적으로 세팅을 해본다.  
설정 파일은 `postfix`와 마찬가지로 `/etc/postfix/main.cf`의 파일로 진행하면 된다.  
Proxmox의 기본 값들이 존재하지만, 이 중 `relayhost` 항목만 주석 처리 후 아래에 다시 추가를 해준다.

```shell
# /etc/postfix/main.cf
root@is-pve1:~$ vi /etc/postfix/main.cf
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

myhostname=is-pve-test1.local

smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, localhost.$mydomain, localhost
# relayhost 주석 처리
#relayhost =
mynetworks = 127.0.0.0/8
inet_interfaces = loopback-only
recipient_delimiter = +

compatibility_level = 2

# add config for use email
relayhost = [smtp.office365.com]:587

inet_protocols = all

smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd

smtp_use_tls = yes
smtp_tls_security_level = may
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

인증을 위해 `sasl_passwd`파일도 생성해준다.

```shell
# /etc/postfix/sasl_passwd
root@is-pve1:~$ vi /etc/postfix/sasl_passwd
# username@domain.com:password 부분은 사용자에 맞게 수정
[smtp.office365.com]:587 username@domain.com:password
```

이후 적용을 위해 다음과 같이 진행한다.

```shell
# sasl_passwd postfix에 적용
root@is-pve1:~$ postmap /etc/postfix/sasl_passwd

# 서비스 재시작
root@is-pve1:~$ systemctl restart postfix
```

## E-Mail 테스트
---

테스트는 다음과 같은 한 줄로 가능하다.

```shell
root@is-pve1:~$ echo "테스트 입니다." | mail -a "From: send@senddomain.com" -s "제목 없음" dest@destdomain.com
```

Error의 발생 여부를 확인하기 위해선 `/var/log/mail.log`를 통하여 확인이 가능하다.