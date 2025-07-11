---
title: Linux(Ubuntu) - rsyslog 설정
date: 2023-10-16 10:19:00 +0900
categories: [Linux]
tags: [linux, ubuntu, rsyslog]
description: Ubuntu에서 logging 패키지인 rsyslog의 설정하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `syslog`는 Linux의 대다수의 logging을 담당하는 log이다.
* 대다수의 Linux에서도 설정 법은 동일하다.
* `rsyslog`는 `syslog`에 대해 logging도 해주지만, client를 설정해주면 log를 모아주는 server로서의 기능도 해준다.

## Server 설정
---

`rsyslog`의 패키지는 기본적으로 Ubuntu에는 설치가 되어있다.  
설정 값은 기본적으로 `/etc/rsyslog.conf`에 들어 가있다.  
최초에는 `module`에 UDP, TCP 포트가 따로 열려있지는 않고, localhost에 대한 logging을 진행하게 되며, default rule을 기반으로 동작한다.  
우선 서버에 대한 구성을 진행하려면 `/etc/rsyslog.conf`의 config를 변경 해준다.

```shell
dor1@hq-is-lxc-syslog:~$ sudo vi /etc/rsyslog.conf
# /etc/rsyslog.conf configuration file for rsyslog
#
# For more information install rsyslog-doc and see
# /usr/share/doc/rsyslog-doc/html/configuration/index.html
#
# Default logging rules can be found in /etc/rsyslog.d/50-default.conf


#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
#module(load="immark")  # provides --MARK-- message capability

## 주석 해제
# provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")

## 주석 해제
# provides TCP syslog reception
module(load="imtcp")
input(type="imtcp" port="514")

# provides kernel logging support and enable non-kernel klog messages
module(load="imklog" permitnonkernelfacility="on")

## 추가(해당 IP와 Domain만 허용)
#
## Limit access to specific subnet, ip host or domain
#
$AllowedSender UDP, 127.0.0.1, 10.250.0.0/16

## 추가(Log의 저장 경로)
#
## Template for received remote messages
#
$template remote-incoming-logs, "/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?remote-incoming-logs

###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Filter duplicated messages
$RepeatedMsgReduction on

#
# Set the default permissions for all log files.
#
$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

#
# Where to place spool and state files
#
$WorkDirectory /var/spool/rsyslog

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf 
```

위와 같이 수정이 되었다면, Linux의 기본적인 log들을 받을 준비는 됐다.  
서비스 재시작을 하여 config를 적용해준다.

```shell
dor1@hq-is-lxc-syslog:~$ sudo systemctl restart rsyslog
dor1@hq-is-lxc-syslog:~$ sudo systemctl status rsyslog
* rsyslog.service - System Logging Service
     Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-10-16 10:40:03 KST; 7s ago
TriggeredBy: * syslog.socket
       Docs: man:rsyslogd(8)
             man:rsyslog.conf(5)
             https://www.rsyslog.com/doc/
   Main PID: 1393917 (rsyslogd)
      Tasks: 10 (limit: 462678)
     Memory: 1.5M
        CPU: 5ms
     CGroup: /system.slice/rsyslog.service
             `-1393917 /usr/sbin/rsyslogd -n -iNONE

Oct 16 10:40:03 hq-is-lxc-syslog systemd[1]: Starting System Logging Service...
Oct 16 10:40:03 hq-is-lxc-syslog rsyslogd[1393917]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.  [v8.2112.0]
Oct 16 10:40:03 hq-is-lxc-syslog rsyslogd[1393917]: rsyslogd's groupid changed to 103
Oct 16 10:40:03 hq-is-lxc-syslog systemd[1]: Started System Logging Service.
Oct 16 10:40:03 hq-is-lxc-syslog rsyslogd[1393917]: rsyslogd's userid changed to 101
Oct 16 10:40:03 hq-is-lxc-syslog rsyslogd[1393917]: [origin software="rsyslogd" swVersion="8.2112.0" x-pid="1393917" x-info="https://www.rsyslog.com"] start
```

## Client 설정
---

Server와 같이 `/etc/rsyslog.conf`를 수정해주면 된다.  
Log를 보낼 server만 적어주면 되어서 간단하게 수정 하면 된다.

```shell
root@hq-is-lxc-test1:~$ cat /etc/rsyslog.conf
# /etc/rsyslog.conf configuration file for rsyslog
#
# For more information install rsyslog-doc and see
# /usr/share/doc/rsyslog-doc/html/configuration/index.html
#
# Default logging rules can be found in /etc/rsyslog.d/50-default.conf


#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
#module(load="immark")  # provides --MARK-- message capability

# provides UDP syslog reception
#module(load="imudp")
#input(type="imudp" port="514")

# provides TCP syslog reception
#module(load="imtcp")
#input(type="imtcp" port="514")

# provides kernel logging support and enable non-kernel klog messages
module(load="imklog" permitnonkernelfacility="on")

###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Filter duplicated messages
$RepeatedMsgReduction on

#
# Set the default permissions for all log files.
#
$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

#
# Where to place spool and state files
#
$WorkDirectory /var/spool/rsyslog

## 추가
#
# Syslog server
#
*.* @10.50.1.101:514


## 추가
#
# If the Syslog server will be down
#
$ActionQueueFileName queue
$ActionQueueMaxDiskSpace 1G
$ActionQueueSaveOnShutdown on
$ActionQueueType LinkedList
$ActionResumeRetryCount -1

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf
```

이후 server와 동일하게 서비스 재 시작을 한 뒤 server에서 확인하면 된다.

```shell
root@hq-is-lxc-test1:~$ systemctl restart rsyslog
root@hq-is-lxc-test1:~$ systemctl status rsyslog
* rsyslog.service - System Logging Service
     Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-10-16 01:48:19 UTC; 3s ago
TriggeredBy: * syslog.socket
       Docs: man:rsyslogd(8)
             man:rsyslog.conf(5)
             https://www.rsyslog.com/doc/
   Main PID: 1015 (rsyslogd)
      Tasks: 4 (limit: 462678)
     Memory: 1.1M
        CPU: 3ms
     CGroup: /system.slice/rsyslog.service
             `-1015 /usr/sbin/rsyslogd -n -iNONE

Oct 16 01:48:19 hq-is-lxc-test1 systemd[1]: Starting System Logging Service...
Oct 16 01:48:19 hq-is-lxc-test1 systemd[1]: Started System Logging Service.
Oct 16 01:48:19 hq-is-lxc-test1 rsyslogd[1015]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.  [v8.2112.0]
Oct 16 01:48:19 hq-is-lxc-test1 rsyslogd[1015]: rsyslogd's groupid changed to 103
Oct 16 01:48:19 hq-is-lxc-test1 rsyslogd[1015]: rsyslogd's userid changed to 101
Oct 16 01:48:19 hq-is-lxc-test1 rsyslogd[1015]: [origin software="rsyslogd" swVersion="8.2112.0" x-pid="1015" x-info="https://www.rsyslog.com"] start
```

## 설정 확인
---

Server에서 log의 저장 경로에 제대로 저장되는지 확인 해본다.

```shell
dor1@hq-is-lxc-syslog:~$ sudo tree -h /var/log/rsyslog
[4.0K]  /var/log/rsyslog
|-- [4.0K]  hq-is-lxc-syslog
|   |-- [ 379]  CRON.log
|   |-- [  75]  PackageKit.log
|   |-- [1.2K]  dbus-daemon.log
|   |-- [2.6K]  kernel.log
|   |-- [ 152]  polkitd.log
|   |-- [ 745]  postfix.log
|   |-- [2.6K]  rsyslogd.log
|   |-- [4.0K]  sudo.log
|   `-- [2.1K]  systemd.log
`-- [4.0K]  localhost
    `-- [6.7K]  kernel.log

2 directories, 10 files
```