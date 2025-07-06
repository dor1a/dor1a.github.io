---
title: Server - BMC(IPMI) 관리 및 ipmitool 사용 법
date: 2022-07-22 14:04:00 +0900
categories: [Hardware]
tags: [bmc]
description: BMC(IPMI)의 관리 및 ipmitool의 사용 법에 대해 기록 해봤다.
---

>Ubuntu 20.04
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

# 개요
---

* 서버 사용 시 BMC 기능을 CLI로 사용하는 방법 제시한다.
* `ipmitool` 명령어를 통하여 사용한다.
* DGX 서버 같은 경우엔 `nvipmitool`와 같이 custom 되어있는 명령어를 사용하기도 한다.

# BMC
---

다음과 같은 명령어로 `ipmitool` 설치 및 사용이 가능하다.

```shell
sudo apt install ipmitool
sudo modprobe ipmi_devintf
sudo modprobe ipmi_si
```

## 1. IP 확인

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan print 1
Set in Progress         : Set Complete
Auth Type Support       : MD5
Auth Type Enable        : Callback : MD5
                        : User     : MD5
                        : Operator : MD5
                        : Admin    : MD5
                        : OEM      : MD5
IP Address Source       : DHCP Address
IP Address              : 192.168.11.183
Subnet Mask             : 255.255.255.0
MAC Address             : 5c:ff:35:e3:50:a2
SNMP Community String   : AMI
IP Header               : TTL=0x40 Flags=0x40 Precedence=0x00 TOS=0x10
BMC ARP Control         : ARP Responses Enabled, Gratuitous ARP Disabled
Gratituous ARP Intrvl   : 0.0 seconds
Default Gateway IP      : 192.168.11.254
Default Gateway MAC     : 4c:5e:0c:02:d9:ca
Backup Gateway IP       : 0.0.0.0
Backup Gateway MAC      : 00:00:00:00:00:00
802.1q VLAN ID          : Disabled
802.1q VLAN Priority    : 0
RMCP+ Cipher Suites     : 0,1,2,3,6,7,8,11,12,15,16,17
Cipher Suite Priv Max   : caaaaaaaaaaaXXX
                        :     X=Cipher Suite Unused
                        :     c=CALLBACK
                        :     u=USER
                        :     o=OPERATOR
                        :     a=ADMIN
                        :     O=OEM
Bad Password Threshold  : 0
Invalid password disable: no
Attempt Count Reset Int.: 0
User Lockout Interval   : 0
```

## 2. IP 설정

전부 설정하였으면 `access on` 처리를 해줘야 접속이 된다.

```shell
sudo ipmitool lan set 1 access on
```

### DHCP IP 설정

DHCP는 `default 설정 값`이다.

```shell
sudo ipmitool lan set 1 ipsrc dhcp
```

### Static IP 설정

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan set 1 ipsrc static
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan set 1 ipaddr 192.168.11.183
Setting LAN IP Address to 192.168.11.183
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan set 1 netmask 255.255.255.0
Setting LAN Subnet Mask to 255.255.255.0
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan set 1 defgw ipaddr 192.168.11.254
Setting LAN Default Gateway IP to 192.168.11.254
```

### (Option 1) MAC 및 arp respond 설정

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan set 1 defgw macaddr 00:0e:0c:aa:8e:13
Setting LAN Default Gateway MAC to 00:0e:0c:aa:8e:13
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool lan set 1 arp respond on
Enabling BMC-generated ARP responses
```

### (Option 2) USER MD5 설정

```shell
sudo ipmitool lan set 1 auth USER MD5
```

## 3. 계정 관리

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user
User Commands:
               summary      [<channel number>]
               list         [<channel number>]
               set name     <user id> <username>
               set password <user id> [<password> <16|20>]
               disable      <user id>
               enable       <user id>
               priv         <user id> <privilege level> [<channel number>]
                     Privilege levels:
                      * 0x1 - Callback
                      * 0x2 - User
                      * 0x3 - Operator
                      * 0x4 - Administrator
                      * 0x5 - OEM Proprietary
                      * 0xF - No Access
               test         <user id> <16|20> [<password]>
```

User 관리에서는 `User`, `Password`, `Privileged` 등 설정이 가능하다.

### User 확인

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user list 1
ID  Name             Callin  Link Auth  IPMI Msg   Channel Priv Limit
1                    false   false      true       ADMINISTRATOR
2   admin            false   false      true       ADMINISTRATOR
3   Administrator    true    true       true       ADMINISTRATOR
4                    true    false      false      NO ACCESS
5                    true    false      false      NO ACCESS
6                    true    true       false      NO ACCESS
7                    true    false      false      NO ACCESS
8                    true    false      false      NO ACCESS
9                    true    false      false      NO ACCESS
10                   true    false      false      NO ACCESS
```

### User 및 Password 설정

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user set name 4 nvadmin
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user set password 4
Password for user 4: 
Password for user 4: 
# user 활성화
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user enable 4
```

### Channel 및 Privileged Limit 설정

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool channel
Channel Commands: authcap   <channel number> <max privilege>
                  getaccess <channel number> [user id]
                  setaccess <channel number> <user id>
                            [callin=on|off] [ipmi=on|off] [link=on|off] [privilege=level]
                  info      [channel number]
                  getciphers <ipmi | sol> [channel]
Possible privilege levels are:
   1   Callback level
   2   User level
   3   Operator level
   4   Administrator level
   5   OEM Proprietary level
  15   No access
```

Channel과 Privileged Limit은 같이 적용해야 한다.  
또한 이 곳에서 Admin 권한 및 Web access 같은 것을 관리 할 수 있다.

다음은 Admin 권한(모든 권한)을 주는 방법이다.

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool channel setaccess 1 4 link=on ipmi=on callin=on
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user priv 4 4 1
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool user list 1
ID  Name             Callin  Link Auth  IPMI Msg   Channel Priv Limit
1                    false   false      true       ADMINISTRATOR
2   admin            false   false      true       ADMINISTRATOR
3   Administrator    true    true       true       ADMINISTRATOR
4   nvadmin          true    true       true       ADMINISTRATOR
5                    true    false      false      NO ACCESS
6                    true    false      false      NO ACCESS
7                    true    false      false      NO ACCESS
8                    true    false      false      NO ACCESS
9                    true    false      false      NO ACCESS
10                   true    false      false      NO ACCESS
```

## 4. 관리

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool mc
Not enough parameters given.
MC Commands:
  reset <warm|cold>
  guid
  info
  watchdog <get|reset|off>
  selftest
  getenables
  setenables <option=on|off> ...
    recv_msg_intr         Receive Message Queue Interrupt
    event_msg_intr        Event Message Buffer Full Interrupt
    event_msg             Event Message Buffer
    system_event_log      System Event Logging
    oem0                  OEM 0
    oem1                  OEM 1
    oem2                  OEM 2
  getsysinfo <argument>
  setsysinfo <argument> <string>
    system_fw_version   System firmware (e.g. BIOS) version
    primary_os_name     Primary operating system name
    os_name             Operating system name
    system_name         System Name of server(vendor dependent)
    delloem_os_version  Running version of operating system
    delloem_url         URL of BMC webserver
```

### BMC selftest

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool mc selftest
Selftest: passed
```

### BMC reset

```shell
nvadmin@DGX-A100-80G-n1:~$ sudo ipmitool mc reset cold
Sent cold reset command to MC
```

보통 BMC 동작 불능 일 때 사용하며 `warm reset` 보다 `cold reset`을 더 많이 사용한다.

## 5. BMC 초기화

간혹 BMC 사용 불능 되는 경우가 있어 해결 법 추가

* **raw 값^(16진수)^은 서버마다 다를 수 있으며 다른 값을 넣으면 장애가 생길 수 있음**
* 만약 Static IP를 사용한다면 BMC 초기화 시 IP를 다시 설정해야 함

### DGX System

>DGX A100, DGX A100 Station 테스트 함
{: .prompt-info}

```shell
sudo nvipmitool raw 0x32 0x66
```