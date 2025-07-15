---
title: Ubuntu - LDAP Client 설정
date: 2024-01-09 16:45:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, ldap]
description: Ubuntu에서 LDAP Client를 설정하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* LDAP(Lightweight Directory Access Protocol)의 client의 세팅 방법을 제공한다.
* client의 종류는 여러가지가 있는데 대표적으로 사용되는건 다음과 같다.  
  1. **sssd(System Security Services Daemon)**
     * **sssd**는 LDAP를 포함한 다양한 인증 서비스와 통합할 수 있다.
     * 오프라인 인증 지원 및 캐싱 기능을 제공하므로 네트워크 연결이 불안정한 환경에서 유용하다.
     * LDAP 이외에도 `Kerberos`, `Samba`, `Active Directory` 등 다양한 인증 체계와의 통합이 가능하다.
  2. **nslcd(Name Service LDAP Client Daemon)**
     * **nslcd**는 LDAP 디렉토리 서버와 통신하여 사용자 및 그룹 정보 등을 관리하는 데 사용된다.
     * 간단하고 가벼운 구성이 장점이다.
     * NSS(Name Service Switch)를 통해 시스템의 사용자 및 그룹 정보를 관리하는 데 주로 사용된다.
  3. **nss_ldap**
     * NSS(Name Service Switch)를 통해 시스템의 사용자와 그룹 정보를 LDAP 서버에서 가져오는 데 사용된다.
     * 주로 사용자 계정 정보를 관리하는 데 사용된다.
  4. **pam_ldap**
     * PAM(Pluggable Authentication Modules)을 통해 LDAP 서버를 인증에 사용하도록 설정한다.
     * 주로 시스템 로그인과 같은 인증 작업에 사용된다.
* 선택이 어려울때는 다음과 같이 선택하면 된다.
  * 복잡한 인증 요구 사항이나 다양한 인증 시스템과의 통합, 오프라인 인증 지원같은 기능이 필요하다면 **sssd**가 좋은 선택이 될 수 있다.
  * 단순한 LDAP 인증 및 사용자 정보 관리가 주 목적이라면 **nslcd**을 선택할 수 있다.
  * 특정 시스템 인증이나 사용자/그룹 관리에 집중하고자 한다면 **pam_ldap**나 **nss_ldap**을 선택할 수 있다.
* 이 글에서는 **pam_ldap**을 제외한 방법을 다룬다.
* Server 구축 방법은 다음과 같은 link로 가면 된다.  
  [Linux(Ubuntu) - LDAP Server 구축](/posts/ubuntu-build_ldap_server/)

> ref.
> - <https://www.labsrc.com/setting-up-openldap-sssd-w-sudo-on-ubuntu-22-04/>

## sssd로 client 설치
---

많은 기능과 보안적으로 가장 강력하다.  
위에 나열한 client 중 가장 최근에 나왔다.  
모든 기능을 정상적으로 사용하기 위해선 LDAP Server에 TLS/SSL이 필수로 필요하다.

### 1. 설치

기능이 많다보니 패키지가 다른 client 대비 많다.

```shell
admin@lxc-wg:~$ sudo apt install sssd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  cracklib-runtime ldap-utils libavahi-client3 libavahi-common-data libavahi-common3 libbasicobjects0 libc-ares2 libcollection4 libcrack2 libcups2 libdhash1 libini-config5 libipa-hbac0 libldb2 libnfsidmap1 libnl-3-200 libnl-route-3-200 libnss-sss
  libpam-pwquality libpam-sss libpath-utils1 libpwquality-common libpwquality1 libref-array1 libsasl2-modules libsasl2-modules-gssapi-mit libsmbclient libsss-certmap0 libsss-idmap0 libsss-nss-idmap0 libtalloc2 libtdb1 libtevent0 libwbclient0 python3-ldb
  python3-sss python3-talloc samba-libs sssd-ad sssd-ad-common sssd-common sssd-ipa sssd-krb5 sssd-krb5-common sssd-ldap sssd-proxy wamerican
Suggested packages:
  cups-common libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql adcli apparmor libsss-sudo sssd-tools
The following NEW packages will be installed:
  cracklib-runtime ldap-utils libavahi-client3 libavahi-common-data libavahi-common3 libbasicobjects0 libc-ares2 libcollection4 libcrack2 libcups2 libdhash1 libini-config5 libipa-hbac0 libldb2 libnfsidmap1 libnl-3-200 libnl-route-3-200 libnss-sss
  libpam-pwquality libpam-sss libpath-utils1 libpwquality-common libpwquality1 libref-array1 libsasl2-modules libsasl2-modules-gssapi-mit libsmbclient libsss-certmap0 libsss-idmap0 libsss-nss-idmap0 libtalloc2 libtdb1 libtevent0 libwbclient0 python3-ldb
  python3-sss python3-talloc samba-libs sssd sssd-ad sssd-ad-common sssd-common sssd-ipa sssd-krb5 sssd-krb5-common sssd-ldap sssd-proxy wamerican
0 upgraded, 48 newly installed, 0 to remove and 32 not upgraded.
Need to get 10.3 MB of archives.
After this operation, 41.6 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://mirror.kakao.com/ubuntu jammy/main amd64 libtalloc2 amd64 2.3.3-2build1 [25.6 kB]
Get:2 http://mirror.kakao.com/ubuntu jammy/main amd64 libtevent0 amd64 0.11.0-1build1 [39.2 kB]
Get:3 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 libwbclient0 amd64 2:4.15.13+dfsg-0ubuntu1.6 [266 kB]
Get:4 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libavahi-common-data amd64 0.8-5ubuntu5.2 [23.8 kB]
Get:5 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libavahi-common3 amd64 0.8-5ubuntu5.2 [23.9 kB]
Get:6 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libavahi-client3 amd64 0.8-5ubuntu5.2 [28.0 kB]
Get:7 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libcups2 amd64 2.4.1op1-1ubuntu4.10 [263 kB]
Get:8 http://mirror.kakao.com/ubuntu jammy/main amd64 libtdb1 amd64 1.4.5-2build1 [46.4 kB]
Get:9 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libldb2 amd64 2:2.4.4-0ubuntu0.22.04.2 [154 kB]
Get:10 http://mirror.kakao.com/ubuntu jammy-security/main amd64 python3-ldb amd64 2:2.4.4-0ubuntu0.22.04.2 [41.7 kB]
Get:11 http://mirror.kakao.com/ubuntu jammy/main amd64 python3-talloc amd64 2.3.3-2build1 [13.0 kB]
Get:12 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 samba-libs amd64 2:4.15.13+dfsg-0ubuntu1.6 [6276 kB]
Get:13 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 libsmbclient amd64 2:4.15.13+dfsg-0ubuntu1.6 [65.9 kB]
Get:14 http://mirror.kakao.com/ubuntu jammy/main amd64 libcrack2 amd64 2.9.6-3.4build4 [29.6 kB]
Get:15 http://mirror.kakao.com/ubuntu jammy/main amd64 cracklib-runtime amd64 2.9.6-3.4build4 [149 kB]
Get:16 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 ldap-utils amd64 2.5.18+dfsg-0ubuntu0.22.04.2 [147 kB]
Get:17 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 libnfsidmap1 amd64 1:2.6.1-1ubuntu1.2 [42.9 kB]
Get:18 http://mirror.kakao.com/ubuntu jammy/main amd64 libnl-3-200 amd64 3.5.0-0.1 [59.1 kB]
Get:19 http://mirror.kakao.com/ubuntu jammy/main amd64 libnl-route-3-200 amd64 3.5.0-0.1 [180 kB]
Get:20 http://mirror.kakao.com/ubuntu jammy/main amd64 libpwquality-common all 1.4.4-1build2 [7642 B]
Get:21 http://mirror.kakao.com/ubuntu jammy/main amd64 libpwquality1 amd64 1.4.4-1build2 [13.4 kB]
Get:22 http://mirror.kakao.com/ubuntu jammy/main amd64 libpam-pwquality amd64 1.4.4-1build2 [11.8 kB]
Get:23 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 libsasl2-modules amd64 2.1.27+dfsg2-3ubuntu1.2 [68.8 kB]
Get:24 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 libsasl2-modules-gssapi-mit amd64 2.1.27+dfsg2-3ubuntu1.2 [31.5 kB]
Get:25 http://mirror.kakao.com/ubuntu jammy/main amd64 wamerican all 2020.12.07-2 [236 kB]
Get:26 http://mirror.kakao.com/ubuntu jammy/main amd64 libbasicobjects0 amd64 0.6.2-1 [6160 B]
Get:27 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libc-ares2 amd64 1.18.1-1ubuntu0.22.04.3 [45.1 kB]
Get:28 http://mirror.kakao.com/ubuntu jammy/main amd64 libcollection4 amd64 0.6.2-1 [23.9 kB]
Get:29 http://mirror.kakao.com/ubuntu jammy/main amd64 libdhash1 amd64 0.6.2-1 [9150 B]
Get:30 http://mirror.kakao.com/ubuntu jammy/main amd64 libpath-utils1 amd64 0.6.2-1 [9254 B]
Get:31 http://mirror.kakao.com/ubuntu jammy/main amd64 libref-array1 amd64 0.6.2-1 [7720 B]
Get:32 http://mirror.kakao.com/ubuntu jammy/main amd64 libini-config5 amd64 0.6.2-1 [44.5 kB]
Get:33 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libipa-hbac0 amd64 2.6.3-1ubuntu3.3 [11.2 kB]
Get:34 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libnss-sss amd64 2.6.3-1ubuntu3.3 [23.7 kB]
Get:35 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libpam-sss amd64 2.6.3-1ubuntu3.3 [40.6 kB]
Get:36 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libsss-certmap0 amd64 2.6.3-1ubuntu3.3 [34.8 kB]
Get:37 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libsss-idmap0 amd64 2.6.3-1ubuntu3.3 [15.9 kB]
Get:38 http://mirror.kakao.com/ubuntu jammy-security/main amd64 libsss-nss-idmap0 amd64 2.6.3-1ubuntu3.3 [21.8 kB]
Get:39 http://mirror.kakao.com/ubuntu jammy-security/main amd64 python3-sss amd64 2.6.3-1ubuntu3.3 [41.2 kB]
Get:40 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-common amd64 2.6.3-1ubuntu3.3 [1131 kB]
Get:41 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-ad-common amd64 2.6.3-1ubuntu3.3 [76.0 kB]
Get:42 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-krb5-common amd64 2.6.3-1ubuntu3.3 [80.5 kB]
Get:43 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-ad amd64 2.6.3-1ubuntu3.3 [137 kB]
Get:44 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-ipa amd64 2.6.3-1ubuntu3.3 [221 kB]
Get:45 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-krb5 amd64 2.6.3-1ubuntu3.3 [14.0 kB]
Get:46 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-ldap amd64 2.6.3-1ubuntu3.3 [32.5 kB]
Get:47 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd-proxy amd64 2.6.3-1ubuntu3.3 [43.2 kB]
Get:48 http://mirror.kakao.com/ubuntu jammy-security/main amd64 sssd amd64 2.6.3-1ubuntu3.3 [4112 B]
Fetched 10.3 MB in 0s (29.5 MB/s)
Extracting templates from packages: 100%
Preconfiguring packages ...
Selecting previously unselected package libtalloc2:amd64.
(Reading database ... 57931 files and directories currently installed.)
Preparing to unpack .../00-libtalloc2_2.3.3-2build1_amd64.deb ...
Unpacking libtalloc2:amd64 (2.3.3-2build1) ...
Selecting previously unselected package libtevent0:amd64.
Preparing to unpack .../01-libtevent0_0.11.0-1build1_amd64.deb ...
Unpacking libtevent0:amd64 (0.11.0-1build1) ...
Selecting previously unselected package libwbclient0:amd64.
Preparing to unpack .../02-libwbclient0_2%3a4.15.13+dfsg-0ubuntu1.6_amd64.deb ...
Unpacking libwbclient0:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Selecting previously unselected package libavahi-common-data:amd64.
Preparing to unpack .../03-libavahi-common-data_0.8-5ubuntu5.2_amd64.deb ...
Unpacking libavahi-common-data:amd64 (0.8-5ubuntu5.2) ...
Selecting previously unselected package libavahi-common3:amd64.
Preparing to unpack .../04-libavahi-common3_0.8-5ubuntu5.2_amd64.deb ...
Unpacking libavahi-common3:amd64 (0.8-5ubuntu5.2) ...
Selecting previously unselected package libavahi-client3:amd64.
Preparing to unpack .../05-libavahi-client3_0.8-5ubuntu5.2_amd64.deb ...
Unpacking libavahi-client3:amd64 (0.8-5ubuntu5.2) ...
Selecting previously unselected package libcups2:amd64.
Preparing to unpack .../06-libcups2_2.4.1op1-1ubuntu4.10_amd64.deb ...
Unpacking libcups2:amd64 (2.4.1op1-1ubuntu4.10) ...
Selecting previously unselected package libtdb1:amd64.
Preparing to unpack .../07-libtdb1_1.4.5-2build1_amd64.deb ...
Unpacking libtdb1:amd64 (1.4.5-2build1) ...
Selecting previously unselected package libldb2:amd64.
Preparing to unpack .../08-libldb2_2%3a2.4.4-0ubuntu0.22.04.2_amd64.deb ...
Unpacking libldb2:amd64 (2:2.4.4-0ubuntu0.22.04.2) ...
Selecting previously unselected package python3-ldb.
Preparing to unpack .../09-python3-ldb_2%3a2.4.4-0ubuntu0.22.04.2_amd64.deb ...
Unpacking python3-ldb (2:2.4.4-0ubuntu0.22.04.2) ...
Selecting previously unselected package python3-talloc:amd64.
Preparing to unpack .../10-python3-talloc_2.3.3-2build1_amd64.deb ...
Unpacking python3-talloc:amd64 (2.3.3-2build1) ...
Selecting previously unselected package samba-libs:amd64.
Preparing to unpack .../11-samba-libs_2%3a4.15.13+dfsg-0ubuntu1.6_amd64.deb ...
Unpacking samba-libs:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Selecting previously unselected package libsmbclient:amd64.
Preparing to unpack .../12-libsmbclient_2%3a4.15.13+dfsg-0ubuntu1.6_amd64.deb ...
Unpacking libsmbclient:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Selecting previously unselected package libcrack2:amd64.
Preparing to unpack .../13-libcrack2_2.9.6-3.4build4_amd64.deb ...
Unpacking libcrack2:amd64 (2.9.6-3.4build4) ...
Selecting previously unselected package cracklib-runtime.
Preparing to unpack .../14-cracklib-runtime_2.9.6-3.4build4_amd64.deb ...
Unpacking cracklib-runtime (2.9.6-3.4build4) ...
Selecting previously unselected package ldap-utils.
Preparing to unpack .../15-ldap-utils_2.5.18+dfsg-0ubuntu0.22.04.2_amd64.deb ...
Unpacking ldap-utils (2.5.18+dfsg-0ubuntu0.22.04.2) ...
Selecting previously unselected package libnfsidmap1:amd64.
Preparing to unpack .../16-libnfsidmap1_1%3a2.6.1-1ubuntu1.2_amd64.deb ...
Unpacking libnfsidmap1:amd64 (1:2.6.1-1ubuntu1.2) ...
Selecting previously unselected package libnl-3-200:amd64.
Preparing to unpack .../17-libnl-3-200_3.5.0-0.1_amd64.deb ...
Unpacking libnl-3-200:amd64 (3.5.0-0.1) ...
Selecting previously unselected package libnl-route-3-200:amd64.
Preparing to unpack .../18-libnl-route-3-200_3.5.0-0.1_amd64.deb ...
Unpacking libnl-route-3-200:amd64 (3.5.0-0.1) ...
Selecting previously unselected package libpwquality-common.
Preparing to unpack .../19-libpwquality-common_1.4.4-1build2_all.deb ...
Unpacking libpwquality-common (1.4.4-1build2) ...
Selecting previously unselected package libpwquality1:amd64.
Preparing to unpack .../20-libpwquality1_1.4.4-1build2_amd64.deb ...
Unpacking libpwquality1:amd64 (1.4.4-1build2) ...
Selecting previously unselected package libpam-pwquality:amd64.
Preparing to unpack .../21-libpam-pwquality_1.4.4-1build2_amd64.deb ...
Unpacking libpam-pwquality:amd64 (1.4.4-1build2) ...
Selecting previously unselected package libsasl2-modules:amd64.
Preparing to unpack .../22-libsasl2-modules_2.1.27+dfsg2-3ubuntu1.2_amd64.deb ...
Unpacking libsasl2-modules:amd64 (2.1.27+dfsg2-3ubuntu1.2) ...
Selecting previously unselected package libsasl2-modules-gssapi-mit:amd64.
Preparing to unpack .../23-libsasl2-modules-gssapi-mit_2.1.27+dfsg2-3ubuntu1.2_amd64.deb ...
Unpacking libsasl2-modules-gssapi-mit:amd64 (2.1.27+dfsg2-3ubuntu1.2) ...
Selecting previously unselected package wamerican.
Preparing to unpack .../24-wamerican_2020.12.07-2_all.deb ...
Unpacking wamerican (2020.12.07-2) ...
Selecting previously unselected package libbasicobjects0:amd64.
Preparing to unpack .../25-libbasicobjects0_0.6.2-1_amd64.deb ...
Unpacking libbasicobjects0:amd64 (0.6.2-1) ...
Selecting previously unselected package libc-ares2:amd64.
Preparing to unpack .../26-libc-ares2_1.18.1-1ubuntu0.22.04.3_amd64.deb ...
Unpacking libc-ares2:amd64 (1.18.1-1ubuntu0.22.04.3) ...
Selecting previously unselected package libcollection4:amd64.
Preparing to unpack .../27-libcollection4_0.6.2-1_amd64.deb ...
Unpacking libcollection4:amd64 (0.6.2-1) ...
Selecting previously unselected package libdhash1:amd64.
Preparing to unpack .../28-libdhash1_0.6.2-1_amd64.deb ...
Unpacking libdhash1:amd64 (0.6.2-1) ...
Selecting previously unselected package libpath-utils1:amd64.
Preparing to unpack .../29-libpath-utils1_0.6.2-1_amd64.deb ...
Unpacking libpath-utils1:amd64 (0.6.2-1) ...
Selecting previously unselected package libref-array1:amd64.
Preparing to unpack .../30-libref-array1_0.6.2-1_amd64.deb ...
Unpacking libref-array1:amd64 (0.6.2-1) ...
Selecting previously unselected package libini-config5:amd64.
Preparing to unpack .../31-libini-config5_0.6.2-1_amd64.deb ...
Unpacking libini-config5:amd64 (0.6.2-1) ...
Selecting previously unselected package libipa-hbac0.
Preparing to unpack .../32-libipa-hbac0_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking libipa-hbac0 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package libnss-sss:amd64.
Preparing to unpack .../33-libnss-sss_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking libnss-sss:amd64 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package libpam-sss:amd64.
Preparing to unpack .../34-libpam-sss_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking libpam-sss:amd64 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package libsss-certmap0.
Preparing to unpack .../35-libsss-certmap0_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking libsss-certmap0 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package libsss-idmap0.
Preparing to unpack .../36-libsss-idmap0_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking libsss-idmap0 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package libsss-nss-idmap0.
Preparing to unpack .../37-libsss-nss-idmap0_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking libsss-nss-idmap0 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package python3-sss.
Preparing to unpack .../38-python3-sss_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking python3-sss (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-common.
Preparing to unpack .../39-sssd-common_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-common (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-ad-common.
Preparing to unpack .../40-sssd-ad-common_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-ad-common (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-krb5-common.
Preparing to unpack .../41-sssd-krb5-common_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-krb5-common (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-ad.
Preparing to unpack .../42-sssd-ad_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-ad (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-ipa.
Preparing to unpack .../43-sssd-ipa_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-ipa (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-krb5.
Preparing to unpack .../44-sssd-krb5_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-krb5 (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-ldap.
Preparing to unpack .../45-sssd-ldap_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-ldap (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd-proxy.
Preparing to unpack .../46-sssd-proxy_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd-proxy (2.6.3-1ubuntu3.3) ...
Selecting previously unselected package sssd.
Preparing to unpack .../47-sssd_2.6.3-1ubuntu3.3_amd64.deb ...
Unpacking sssd (2.6.3-1ubuntu3.3) ...
Setting up libpwquality-common (1.4.4-1build2) ...
Setting up libpath-utils1:amd64 (0.6.2-1) ...
Setting up libnfsidmap1:amd64 (1:2.6.1-1ubuntu1.2) ...
Setting up libsss-idmap0 (2.6.3-1ubuntu3.3) ...
Setting up libbasicobjects0:amd64 (0.6.2-1) ...
Setting up libtdb1:amd64 (1.4.5-2build1) ...
Setting up libsasl2-modules:amd64 (2.1.27+dfsg2-3ubuntu1.2) ...
Setting up libc-ares2:amd64 (1.18.1-1ubuntu0.22.04.3) ...
Setting up ldap-utils (2.5.18+dfsg-0ubuntu0.22.04.2) ...
Setting up libtalloc2:amd64 (2.3.3-2build1) ...
Setting up libdhash1:amd64 (0.6.2-1) ...
Setting up libtevent0:amd64 (0.11.0-1build1) ...
Setting up libavahi-common-data:amd64 (0.8-5ubuntu5.2) ...
Setting up wamerican (2020.12.07-2) ...
Setting up libcrack2:amd64 (2.9.6-3.4build4) ...
Setting up libcollection4:amd64 (0.6.2-1) ...
Setting up libipa-hbac0 (2.6.3-1ubuntu3.3) ...
Setting up libnl-3-200:amd64 (3.5.0-0.1) ...
Setting up libref-array1:amd64 (0.6.2-1) ...
Setting up libldb2:amd64 (2:2.4.4-0ubuntu0.22.04.2) ...
Setting up libsss-nss-idmap0 (2.6.3-1ubuntu3.3) ...
Setting up libsasl2-modules-gssapi-mit:amd64 (2.1.27+dfsg2-3ubuntu1.2) ...
Setting up libnss-sss:amd64 (2.6.3-1ubuntu3.3) ...
First installation detected...
Checking NSS setup...
Adding an entry for automount.
Setting up python3-talloc:amd64 (2.3.3-2build1) ...
Setting up libini-config5:amd64 (0.6.2-1) ...
Setting up libavahi-common3:amd64 (0.8-5ubuntu5.2) ...
Setting up python3-sss (2.6.3-1ubuntu3.3) ...
Setting up libwbclient0:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Setting up libsss-certmap0 (2.6.3-1ubuntu3.3) ...
Setting up libnl-route-3-200:amd64 (3.5.0-0.1) ...
Setting up cracklib-runtime (2.9.6-3.4build4) ...
Setting up libpwquality1:amd64 (1.4.4-1build2) ...
Setting up python3-ldb (2:2.4.4-0ubuntu0.22.04.2) ...
Setting up libavahi-client3:amd64 (0.8-5ubuntu5.2) ...
Setting up libpam-pwquality:amd64 (1.4.4-1build2) ...
Setting up libcups2:amd64 (2.4.1op1-1ubuntu4.10) ...
Setting up libpam-sss:amd64 (2.6.3-1ubuntu3.3) ...
Setting up samba-libs:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Setting up sssd-common (2.6.3-1ubuntu3.3) ...
Creating SSSD system user & group...
adduser: Warning: The home directory `/var/lib/sss' does not belong to the user you are currently creating.
Created symlink /etc/systemd/system/sssd.service.wants/sssd-autofs.socket -> /lib/systemd/system/sssd-autofs.socket.
Created symlink /etc/systemd/system/sssd.service.wants/sssd-nss.socket -> /lib/systemd/system/sssd-nss.socket.
Created symlink /etc/systemd/system/sssd.service.wants/sssd-pam-priv.socket -> /lib/systemd/system/sssd-pam-priv.socket.
Created symlink /etc/systemd/system/sssd.service.wants/sssd-pam.socket -> /lib/systemd/system/sssd-pam.socket.
Created symlink /etc/systemd/system/sssd.service.wants/sssd-ssh.socket -> /lib/systemd/system/sssd-ssh.socket.
Created symlink /etc/systemd/system/sssd.service.wants/sssd-sudo.socket -> /lib/systemd/system/sssd-sudo.socket.
Created symlink /etc/systemd/system/multi-user.target.wants/sssd.service -> /lib/systemd/system/sssd.service.
sssd-autofs.service is a disabled or a static unit, not starting it.
sssd-nss.service is a disabled or a static unit, not starting it.
sssd-pam.service is a disabled or a static unit, not starting it.
sssd-ssh.service is a disabled or a static unit, not starting it.
sssd-sudo.service is a disabled or a static unit, not starting it.
Could not execute systemctl:  at /usr/bin/deb-systemd-invoke line 142.
Setting up sssd-proxy (2.6.3-1ubuntu3.3) ...
Setting up sssd-ad-common (2.6.3-1ubuntu3.3) ...
Created symlink /etc/systemd/system/sssd.service.wants/sssd-pac.socket -> /lib/systemd/system/sssd-pac.socket.
sssd-pac.service is a disabled or a static unit, not starting it.
Could not execute systemctl:  at /usr/bin/deb-systemd-invoke line 142.
Setting up sssd-krb5-common (2.6.3-1ubuntu3.3) ...
Setting up libsmbclient:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Setting up sssd-krb5 (2.6.3-1ubuntu3.3) ...
Setting up sssd-ldap (2.6.3-1ubuntu3.3) ...
Setting up sssd-ad (2.6.3-1ubuntu3.3) ...
Setting up sssd-ipa (2.6.3-1ubuntu3.3) ...
Setting up sssd (2.6.3-1ubuntu3.3) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.8) ...
```

### 2. 세팅

세팅은 다른 client 대비 config 파일 1개로 관리 하기 때문에 매우 간단하다.

```shell
# 신규 생성
admin@lxc-wg:~$ sudo vi /etc/sssd/sssd.conf
[sssd]
config_file_version = 2
services = nss,pam
# 아래 doamin 파트의 타이틀
domains = example.com 

[nss]
#filter_users =
#filter_groups =

[pam]

[domain/example.com]
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
sudo_provider = ldap
enumerate = true
#ignore_group_members = true
## 캐시 사용
cache_credentials = true
#ldap_schema = rfc2307
## 서버의 hosts 등록(TLS/SSL이 필요)
ldap_uri = ldap://example.com
ldap_search_base = dc=example,dc=com
#ldap_user_search_base = 
#ldap_user_object_class = posixAccount
#ldap_user_name = uid
#ldap_group_search_base = 
#ldap_group_object_class = posixGroup
#ldap_group_name = cn
## TLS 사용
ldap_id_use_start_tls = true
#ldap_tls_reqcert = demand
#ldap_tls_cacert = /etc/ssl/certs/ca-certificates.crt
#ldap_default_bind_dn = cn=admin,dc=example,dc=com
#ldap_default_authtok_type = password
#ldap_default_authtok =
access_provider = permit
#ldap_access_filter = (objectClass=posixAccount)
#min_id = 1
#max_id = 0
#ldap_user_uuid = entryUUID
#ldap_user_shell = loginShell
#ldap_user_home_directory = homeDirectory
#ldap_user_uid_number = uidNumber
#ldap_user_gid_number = gidNumber
#ldap_group_gid_number = gidNumber
#ldap_group_uuid = entryUUID
#ldap_group_member = memberUid
#use_fully_qualified_names = false
#ldap_access_order = filter
## logging 레벨 조절
debug_level=6
```

권한을 꼭 변경해준다.

```shell
admin@lxc-mgmt:/etc/sssd$ sudo chmod 0600 /etc/sssd/sssd.conf
```

### 3. hosts 등록

TLS/SSL은 도메인에 적용 되기 때문에 `/etc/hosts`에 LDAP Server를 등록해준다.

```shell
admin@lxc-wg:~$ cat /etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
# --- BEGIN PVE ---
192.168.0.254 lxc-wg.main lxc-wg
# --- END PVE ---

## LDAP Server
192.168.0.250 ldap-server-test2 example.com
```

### 4. CA 인증서 등록

Server에 접속 시 TLS의 CA 인증서가 필요하기 때문에 옮길 필요는 없이 `openssl`의 `s_client` 모듈로 간단하게 적용할 수 있다.

```shell
admin@lxc-wg:~$ sudo sh -c echo -n | openssl s_client -connect example.com:636 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | sudo tee /usr/local/share/ca-certificates/example_com-ca.crt
depth=1 CN = example.com
verify error:num=19:self-signed certificate in certificate chain
verify return:1
depth=1 CN = example.com
verify return:1
depth=0 O = example, CN = example.com
verify return:1
DONE
-----BEGIN CERTIFICATE-----
MIIETjCCAjagAwIBAgIUFENcpiIdyKn9XFruexau6PFtLkswDQYJKoZIhvcNAQEM
BQAwGDEWMBQGA1UEAxMNbGVleW9vbmhvLmNvbTAeFw0yNDA4MTMwNDE5NDBaFw0z
NDA4MTEwNDE5NDBaMDMxFDASBgNVBAoTC0xlZSBZb29uIEhvMRswGQYDVQQDExJs
ZGFwLmxlZXlvb25oby5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIB
AQC7xoQCG/83xnRZ/b3HBrrCcLe5pgno/IplJFOQFmeQwjrPlsetXGdfvL/DZ//Z
rOiXE4T1yHXQ1lgVOw2aSSHrgVFVCiiLomBPqAACq/dpoeSKN4Xczg3AK5Nf0K2S
orOhpnlhd3QsVqFf5M+gA6VW9YZXkFqVssSGN8hSDo3sErII7OLXLw80nlYjMt1j
ATJwryByYmecESn7zce26T5txtXQcwQqUADLnaPtWyLpyyi3rnEWXt1JyVMsW1pX
obkdEvA5ohi40tg59JgzQuwiVd8OZCTgNjyB3NVQW5CrHVk4tSfJW8aGI3pJ7Lkp
OqP4IoQNhquYwI8s70/vUumBAgMBAAGjdTBzMAwGA1UdEwEB/wQCMAAwEwYDVR0l
BAwwCgYIKwYBBQUHAwEwDgYDVR0PAQH/BAQDAgWgMB0GA1UdDgQWBBSKUP1FNJI3
vQV6tSC8DJQTNCA/7zAfBgNVHSMEGDAWgBT1HyteA+iUlOZaxi4dV8mBOVLbVjAN
BgkqhkiG9w0BAQwFAAOCAgEAXHy0AC1DupsZh2WO+GkzTdHs+zTFwHxVenTbn/DX
gatGXe23SIsXOfMnqQUrwRojPLwvHUjsmkDh5cEL+u5jonGM2tJayj1IHUV8VtOx
uJrccD5t/6Nsn2fRj9Stlc5tgCizsDa+/52sJ/dLAo1luq6fMC6b6KttZFrcpEA1
dZ5HKg3D2+NfjhzetS3hhV+K9gGeiuaD1eEovG/QwQvvYvT7FabATRQ96bnk6x2a
mcF5VGu8KLJ8JuRTqqhHVKXPLb577qarRDntom6NW3EgWBB21OTmvut7HfJqR8jH
nFtQWkYEWuY7E6tnnhkKfUohPOzn/J+fLewlppE7mHwYPHcGsVAZRAlw/Q92D0I2
+M3E4p9f2wLR9bT9AMZOxcjPgy1eoYBFBc9ryB4rYYZA3KzVEHAzK9HVpQnLtH7U
p24n7g47PtDtqbWk9b8dsGScFOeSkYZ9BfqcUAK+dE3DfZlhFZnLxN6BBnBVpXLo
VTuXSBwVasa4vFPXIix9IoIFY+hGbkZ7WMFy6Yb0SwxqikJre9Ra9w1Fg/QJ2S9e
iv8z0xYW/MvsbBgsbt74ZOhz/ggjwhXiFkPvY5sC3aD7+tP24aIE11n/VwVE98Oz
2bR3f8dATKbEJGCryUmlI+hd8X9/OWp2WSuC/8XFKSVNSbuQ438cvf2AlYffVM7j
Qy4=
-----END CERTIFICATE-----

admin@lxc-wg:~$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

### 5. 서비스 재시작

```shell
admin@lxc-wg:~$ sudo systemctl restart sssd

# 확인
admin@lxc-wg:~$ sudo systemctl status sssd
* sssd.service - System Security Services Daemon
     Loaded: loaded (/lib/systemd/system/sssd.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-08-13 15:27:57 KST; 27s ago
   Main PID: 598072 (sssd)
      Tasks: 4 (limit: 462607)
     Memory: 53.1M
        CPU: 248ms
     CGroup: /system.slice/sssd.service
             |-598072 /usr/sbin/sssd -i --logger=files
             |-598073 /usr/libexec/sssd/sssd_be --domain ldap.leeyoonho.com --uid 0 --gid 0 --logger=files
             |-598074 /usr/libexec/sssd/sssd_nss --uid 0 --gid 0 --logger=files
             `-598075 /usr/libexec/sssd/sssd_pam --uid 0 --gid 0 --logger=files

Aug 13 15:27:52 lxc-wg systemd[1]: Starting System Security Services Daemon...
Aug 13 15:27:52 lxc-wg sssd[598072]: Starting up
Aug 13 15:27:52 lxc-wg sssd_be[598073]: Starting up
Aug 13 15:27:52 lxc-wg sssd_nss[598074]: Starting up
Aug 13 15:27:52 lxc-wg sssd_pam[598075]: Starting up
Aug 13 15:27:57 lxc-wg systemd[1]: Started System Security Services Daemon.

# enable
admin@lxc-wg:~$ sudo systemctl enable sssd
Synchronizing state of sssd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable sssd
```

### 6. Linux PAM 세팅

Linux에는 **PAM(Pluggable Authentication Modules for Linux)**이라는 인증 모듈이 있다.  
여기에서 기본적으로 LDAP으로 연결되는 계정에 대해 login 시 home directory를 생성하는 config를 해줄 수 있다.  
CLI로 설정 하려면 다음과 같이 입력하면 된다.

```shell
sudo pam-auth-update
```

![Create home directory on login 선택](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/1.png)
_Create home directory on login 선택_

PAM의 session의 대한 config는`/etc/pam.d/common-session`에서 확인이 가능하다.

### 6. Test login

연동이 정상적으로 되었는지 확인한다.  
server에서 `test`라는 계정을 미리 생성하여서 만들어 놨다.

```shell
admin@lxc-wg:~$ su deep
Password:
deep@lxc-wg~$

# 권한 확인
deep@lxc-wg:~$ id
uid=10000(deep) gid=10000(deep) groups=10000(deep)
```

## nslcd로 client 설치
---

구성이 매우 간단하고 편하기 때문에 많이 사용된다.  
**nss_ldap**과 같은 오래된 방법을 보완하여 나왔다.  
**nss_ldap**에서는 NSS 설정을 따로 넣어줘야하지만, **nslcd**는 설치 시 config에 들어간다.  
설치는 명령어 한 줄로 간단하다.

### 1. 설치

```shell
admin@ldap-client-test1:~$ sudo apt install libnss-ldapd libpam-ldapd ldap-utils
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libldap-2.5-0 nscd nslcd nslcd-utils
Suggested packages:
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal kstart
The following NEW packages will be installed:
  ldap-utils libnss-ldapd libpam-ldapd nscd nslcd nslcd-utils
The following packages will be upgraded:
  libldap-2.5-0
1 upgraded, 6 newly installed, 0 to remove and 37 not upgraded.
Need to get 634 kB of archives.
After this operation, 1,934 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 libldap-2.5-0 amd64 2.5.17+dfsg-0ubuntu0.22.04.1 [183 kB]
Get:2 http://mirror.kakao.com/ubuntu jammy/universe amd64 nslcd amd64 0.9.12-2 [162 kB]
Get:3 http://mirror.kakao.com/ubuntu jammy-updates/main amd64 ldap-utils amd64 2.5.17+dfsg-0ubuntu0.22.04.1 [147 kB]
Get:4 http://mirror.kakao.com/ubuntu jammy-security/universe amd64 nscd amd64 2.35-0ubuntu3.7 [83.3 kB]
Get:5 http://mirror.kakao.com/ubuntu jammy/universe amd64 libnss-ldapd amd64 0.9.12-2 [29.1 kB]
Get:6 http://mirror.kakao.com/ubuntu jammy/universe amd64 libpam-ldapd amd64 0.9.12-2 [16.6 kB]
Get:7 http://mirror.kakao.com/ubuntu jammy/universe amd64 nslcd-utils all 0.9.12-2 [13.4 kB]
Fetched 634 kB in 0s (1,750 kB/s)
Preconfiguring packages ...
(Reading database ... 80623 files and directories currently installed.)
Preparing to unpack .../0-libldap-2.5-0_2.5.17+dfsg-0ubuntu0.22.04.1_amd64.deb ...
Unpacking libldap-2.5-0:amd64 (2.5.17+dfsg-0ubuntu0.22.04.1) over (2.5.16+dfsg-0ubuntu0.22.04.2) ...
Selecting previously unselected package nslcd.
Preparing to unpack .../1-nslcd_0.9.12-2_amd64.deb ...
Unpacking nslcd (0.9.12-2) ...
Selecting previously unselected package ldap-utils.
Preparing to unpack .../2-ldap-utils_2.5.17+dfsg-0ubuntu0.22.04.1_amd64.deb ...
Unpacking ldap-utils (2.5.17+dfsg-0ubuntu0.22.04.1) ...
Selecting previously unselected package nscd.
Preparing to unpack .../3-nscd_2.35-0ubuntu3.7_amd64.deb ...
Unpacking nscd (2.35-0ubuntu3.7) ...
Selecting previously unselected package libnss-ldapd:amd64.
Preparing to unpack .../4-libnss-ldapd_0.9.12-2_amd64.deb ...
Unpacking libnss-ldapd:amd64 (0.9.12-2) ...
Selecting previously unselected package libpam-ldapd:amd64.
Preparing to unpack .../5-libpam-ldapd_0.9.12-2_amd64.deb ...
Unpacking libpam-ldapd:amd64 (0.9.12-2) ...
Selecting previously unselected package nslcd-utils.
Preparing to unpack .../6-nslcd-utils_0.9.12-2_all.deb ...
Unpacking nslcd-utils (0.9.12-2) ...
Setting up libldap-2.5-0:amd64 (2.5.17+dfsg-0ubuntu0.22.04.1) ...
Setting up nscd (2.35-0ubuntu3.7) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nscd.service → /lib/systemd/system/nscd.service.
Setting up nslcd (0.9.12-2) ...
Warning: The home dir /run/nslcd you specified can't be accessed: No such file or directory
Adding system user `nslcd' (UID 114) ...
Adding new group `nslcd' (GID 119) ...
Adding new user `nslcd' (UID 114) with group `nslcd' ...
Not creating home directory `/run/nslcd'.
Setting up libpam-ldapd:amd64 (0.9.12-2) ...
Setting up ldap-utils (2.5.17+dfsg-0ubuntu0.22.04.1) ...
Setting up nslcd-utils (0.9.12-2) ...
Setting up libnss-ldapd:amd64 (0.9.12-2) ...
/etc/nsswitch.conf: enable LDAP lookups for group
/etc/nsswitch.conf: enable LDAP lookups for passwd
/etc/nsswitch.conf: enable LDAP lookups for shadow
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.7) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
Service restarts being deferred:
 systemctl restart packagekit.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

설치 중간에 prompt가 나오게 되며 입력시 config가 완성된다.

![1. LDAP server address 입력](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/2.png)
_1. LDAP server address 입력_

![2. LDAP server dn 입력](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/3.png)
_2. LDAP server dn 입력_

![3. Name service에 사용 될 config 체크(기본적으로 passwd/group/shadow 체크)](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/4.png)
_3. Name service에 사용 될 config 체크(기본적으로 passwd/group/shadow 체크)_

기본적인 설정을 끝내면 설치가 정상적으로 완료 된다.  
만약 설정을 하다가 잘못 입력하게 되었다면, 다음과 같은 명령어를 통하여 다시 설정이 가능하다.

```shell
sudo dpkg-reconfigure nslcd
```

또한 **nslcd**의 config의 설정은 `/etc/nslcd.conf`에서 확인이 가능하다.

### 2. Linux PAM 세팅

Linux에는 **PAM(Pluggable Authentication Modules for Linux)**이라는 인증 모듈이 있다.  
여기에서 기본적으로 LDAP으로 연결되는 계정에 대해 login 시 home directory를 생성하는 config를 해줄 수 있다.  
CLI로 설정 하려면 다음과 같이 입력하면 된다.

```shell
sudo pam-auth-update
```

![Create home directory on login 선택](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/5.png)
_Create home directory on login 선택_

PAM의 session의 대한 config는`/etc/pam.d/common-session`에서 확인이 가능하다.

### 3. Test login

연동이 정상적으로 되었는지 확인한다.  
server에서 `test`라는 계정을 미리 생성하여서 만들어 놨다.

```shell
admin@ldap-client-test1:~$ su deep
Password:
deep@ldap-client-test1:~$

# 권한 확인
deep@ldap-client-test1:~$ id
uid=10000(deep) gid=10000(deep) groups=10000(deep)
```

## nss_ldap로 client 설치
---

>**nss_ldap**은 현재는 오래된 방법으로 **nslcd** 방법이 더 간단하고 운영하기 편함
{: .prompt-info}

**nss_ldap**은 NSS(Name Service Switch)를 통해 시스템의 사용자와 그룹 정보를 LDAP 서버에서 가져오는 데 사용되며, 주로 사용자 계정 정보를 관리하는 데 사용된다.  
오로지 관리의 목적밖에 없기 때문에 `passwd`등의 명령어를 통한 사용자 비밀번호 변경도 상세한 세팅 없이는 불가능하다.  
해당 글의 server는 **Synlogy LDAP server**로 구성하였다.

### 1. 설치

```shell
admin@ldap-test1:~$ sudo apt install libnss-ldap libpam-ldap ldap-utils nscd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ldap-auth-client ldap-auth-config
Suggested packages:
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal
The following NEW packages will be installed:
  ldap-auth-client ldap-auth-config ldap-utils libnss-ldap libpam-ldap nscd
0 upgraded, 6 newly installed, 0 to remove and 43 not upgraded.
Need to get 82.9 kB/344 kB of archives.
After this operation, 1,563 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://kr.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 nscd amd64 2.35-0ubuntu3.5 [82.9 kB]
Fetched 82.9 kB in 1s (70.8 kB/s)
Preconfiguring packages ...
Selecting previously unselected package ldap-utils.
(Reading database ... 80483 files and directories currently installed.)
Preparing to unpack .../0-ldap-utils_2.5.16+dfsg-0ubuntu0.22.04.1_amd64.deb ...
Unpacking ldap-utils (2.5.16+dfsg-0ubuntu0.22.04.1) ...
Selecting previously unselected package nscd.
Preparing to unpack .../1-nscd_2.35-0ubuntu3.5_amd64.deb ...
Unpacking nscd (2.35-0ubuntu3.5) ...
Selecting previously unselected package ldap-auth-config.
Preparing to unpack .../2-ldap-auth-config_0.5.4_all.deb ...
Unpacking ldap-auth-config (0.5.4) ...
Selecting previously unselected package libpam-ldap:amd64.
Preparing to unpack .../3-libpam-ldap_186-4ubuntu2_amd64.deb ...
Unpacking libpam-ldap:amd64 (186-4ubuntu2) ...
Selecting previously unselected package libnss-ldap:amd64.
Preparing to unpack .../4-libnss-ldap_265-5ubuntu2_amd64.deb ...
Unpacking libnss-ldap:amd64 (265-5ubuntu2) ...
Selecting previously unselected package ldap-auth-client.
Preparing to unpack .../5-ldap-auth-client_0.5.4_all.deb ...
Unpacking ldap-auth-client (0.5.4) ...
Setting up ldap-utils (2.5.16+dfsg-0ubuntu0.22.04.1) ...
Setting up libnss-ldap:amd64 (265-5ubuntu2) ...
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
Setting up nscd (2.35-0ubuntu3.5) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nscd.service → /lib/systemd/system/nscd.service.
Setting up ldap-auth-client (0.5.4) ...
Setting up ldap-auth-config (0.5.4) ...
Setting up libpam-ldap:amd64 (186-4ubuntu2) ...
Processing triggers for man-db (2.10.2-1) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
Service restarts being deferred:
 systemctl restart packagekit.service
 systemctl restart ssh.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

설치 중간에 prompt가 출력 되며, config를 넣어주게 되어있다.  
prompt는 다음과 같이 순서대로 진행하면 된다.

![1. LDAP server address 입력](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/6.png)
_1. LDAP server address 입력_

![2. Domain 입력](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/7.png)
_2. Domain 입력_

![3. LDAP version 선택](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/8.png)
_3. LDAP version 선택_

![4. Local root DB 설정](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/9.png)
_4. Local root DB 설정_

![5. Local root DB login 설정](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/10.png)
_5. Local root DB login 설정_

![6. LDAP root 계정 입력](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/11.png)
_6. LDAP root 계정 입력_

![7. LDAP root 계정 비밀번호 입력](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/12.png)
_7. LDAP root 계정 비밀번호 입력_

기본적인 설정을 끝내면 설치가 정상적으로 완료 된다.  
만약 설정을 하다가 잘못 입력하게 되었다면, 다음과 같은 명령어를 통하여 다시 설정이 가능하다.

```shell
sudo dpkg-reconfigure ldap-auth-config
```

또한 LDAP의 config의 설정은 `/etc/ldap.conf`에서 확인이 가능하다.

### 2. Linux PAM 세팅

Linux에는 **PAM(Pluggable Authentication Modules for Linux)**이라는 인증 모듈이 있다.  
여기에서 기본적으로 LDAP으로 연결되는 계정에 대해 login 시 home directory를 생성하는 config를 해줄 수 있다.  
CLI로 설정 하려면 다음과 같이 입력하면 된다.

```shell
sudo pam-auth-update
```

![Create home directory on login 선택](/assets/img/post/linux/ubuntu/2024-01-09-ubuntu-setup_ldap_client/5.png)
_Create home directory on login 선택_

PAM의 session의 대한 config는`/etc/pam.d/common-session`에서 확인이 가능하다.

### 3. NSS 설정

LDAP server에 등록된 user 들을 사용하기 위해서는 NSS(Name Service Switch)에 LDAP을 필수로 등록해줘야 한다.  
NSS는 OS의 `user`, `group`, `shadow`등을 관리한다.  
config에`passwd`, `group`, `shadow`에 LDAP을 등록해준다.

```shell
admin@ldap-test1:~$ sudo cat /etc/nsswitch.conf
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

## passwd/group/shadow 해당부분에 ldap을 추가하여 준다.
passwd:         files systemd ldap
group:          files systemd ldap
shadow:         files ldap
gshadow:        files

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

### 4. NSCD 서비스 확인

Linux에서는 NSCD(Name Service Cashe Daemon)라는 것을 이용하여 LDAP의 서버의 정보를 가져온다.  
NSCD의 서비스가 정상적으로 작동하는지 확인 해본다.

```shell
# 위의 설정값 적용을 위한 restart service
admin@ldap-test1:~$ sudo systemctl restart nscd

# service 확인
admin@ldap-test1:~$ sudo systemctl status nscd
● nscd.service - Name Service Cache Daemon
     Loaded: loaded (/lib/systemd/system/nscd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-01-10 06:31:04 UTC; 37s ago
    Process: 6205 ExecStart=/usr/sbin/nscd (code=exited, status=0/SUCCESS)
   Main PID: 6206 (nscd)
      Tasks: 10 (limit: 2176)
     Memory: 1.3M
        CPU: 5ms
     CGroup: /system.slice/nscd.service
             └─6206 /usr/sbin/nscd

Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring directory `/etc` (2)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring file `/etc/nsswitch.conf` (7)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring directory `/etc` (2)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring file `/etc/nsswitch.conf` (7)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring directory `/etc` (2)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring file `/etc/nsswitch.conf` (7)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring directory `/etc` (2)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring file `/etc/nsswitch.conf` (7)
Jan 10 06:31:04 ldap-test1 nscd[6206]: 6206 monitoring directory `/etc` (2)
Jan 10 06:31:04 ldap-test1 systemd[1]: Started Name Service Cache Daemon.
```

### 5. Test login

연동이 정상적으로 되었는지 확인한다.  
server에서 `test`라는 계정을 미리 생성하여서 만들어 놨다.

```shell
admin@ldap-test1:~$ su test
Password:
# shell이 /bin/sh로 되어있음
$ exit

# Synology LDAP 계정의 uid/gid는 1000001부터 시작
admin@ldap-test1:~$ id test
uid=1000001(test) gid=1000001(users) groups=1000001(users),1000002(administrators)
```

### 6. 추가 사항

간혹 Synology NAS 같은 server를 사용할 때 기본 shell이 `/bin/sh`인 경우가 있어 client에서 변경해야 하는 경우가 있다.  
기본 shell 변경하는 방법 `/etc/ldap.conf`에 가장 마지막에 다음과 같은 내용을 추가하면 된다.

```shell
# 내용 추가
admin@ldap-test1:~$ vi /etc/ldap.conf
## 가장 마지막 줄
nss_override_attribute_value loginShell /bin/bash

# 적용
admin@ldap-test1:~$ sudo systemctl restart nscd
```

## Option
---

Server의 내용이 바로 반영이 안될 경우에는 다음과 같이 작업 하면 된다.  
기본적인 세팅에서는 `/etc/nscd.conf`의 내용 상 server의 내용이 반영되는데 계정(passwd)은 600초, 그룹(group)은 3600초가 걸린다.  
이럴때 아래와 같이 cache를 비워줄 수 있는 방법이 있다.

```bash
# 계정
sudo nscd -i passwd

# 그룹
sudo nscd -i group
```