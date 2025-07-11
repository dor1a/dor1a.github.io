---
title: Linux(Ubuntu) - rsync 사용 해보기
date: 2023-01-02 16:36:00 +0900
categories: [Linux]
tags: [linux, ubuntu, rsync]
description: 잘 알려진 rsync를 사용 해보았다.
---

>Ubuntu 20.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 사용법은 대부분의 Linux에서도 동일하다.
* `Debian`에서도 설정 파일이 같은 위치에 있어 세팅이 동일하다.

## 설치
---

`Ubuntu` 사용한다면 기본적으로 설치가 되어있다.

```shell
dor1@is-m1 ~ ❯ sudo apt install rsync
Reading package lists... Done
Building dependency tree
Reading state information... Done
rsync is already the newest version (3.1.3-8ubuntu0.4).
0 upgraded, 0 newly installed, 0 to remove and 13 not upgraded.
```

## 설정
---

서비스를 확인 해보면 `service`는 load 되어있지만 `inactive`상태로 제대로 작동은 하지 않는다.

```shell
dor1@is-m1 ~ ❯ sudo systemctl status rsync
● rsync.service - fast remote file copy program daemon
     Loaded: loaded (/lib/systemd/system/rsync.service; enabled; vendor preset: enabled)
     Active: inactive (dead)
  Condition: start condition failed at Mon 2023-01-02 13:47:59 KST; 2s ago
             └─ ConditionPathExists=/etc/rsyncd.conf was not met
       Docs: man:rsync(1)
             man:rsyncd.conf(5)

Dec 19 14:29:37 is-m1 systemd[1]: Condition check resulted in fast remote file copy program daemon being skipped.
```

제대로 작동을 하기 위해서는 `ConditionPathExists`에 있는 것과 같이 `/etc/rsyncd.conf`에 config 값을 입력 해줘야 한다.

### /etc/rsyncd.conf

config 값은 간단하게 아래와 같이 정리해본다.

```shell
dor1@is-m1 ~ ❯ cat /etc/rsyncd.conf
# global setting
log file = /var/log/rsync.log

[rsync_test]
path = /rsync
comment = test
uid = root
gid = root
use chroot = yes
read only = no
hosts allow = *
max connections = 10
timeout = 30s
```

위의 기준으로 확인하였을 때 설정 값은 다음과 같다.

| log file = /var/log/rsync.log | rsync의 log파일이 저장되는 위치                                                                                |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------- |
| \[rsync_test\]                | service명이며, sync 시 service 명으로 진행                                                                     |
| path = /rsync                 | service에서 사용되며, backup 및 sync 시 사용되는 경로                                                          |
| comment = test                | service에서 사용 될 comment                                                                                    |
| uid = root                    | 접속하는 user의 권한                                                                                           |
| gid = root                    | 접속하는 user의 group  권한                                                                                    |
| use chroot = yes              | `chroot`을 이용하여 rsync 사용 시 옮길 때 격리 시킨다.<br>`chroot`은 가상 환경을 이용하는 root file system     |
| read only = no                | 읽기 전용 설정이며, 만약 `yes`로 한다면 server to client로 sync는 가능하지만, 반대로 client to server는 불가능 |
| hosts allow = \*              | 접속을 허용할 `hosts`들 리스트이며, IP or Hostname을 통하여 가능                                               |
| max connections = 10          | 최대 접속 가능 수를 의미                                                                                       |
| timeout = 30s                 | Client에서 `connection timeout`이 발생 하였을 때 지정되는 시간 설정                                            |

추가 설정을 더 보고 싶으면 다음과 같은 링크에서 확인이 가능하다.  
<https://www.samba.org/ftp/rsync/rsyncd.conf.html>  
작성을 다 하였다면 서비스 재 시작을 해주고 정상 작동을 확인한다.

```shell
# 서비스 재시작
dor1@is-m1 ~ ❯ sudo systemctl restart rsync

# 서비스 확인인
dor1@is-m1 ~ ❯ sudo systemctl status rsync
● rsync.service - fast remote file copy program daemon
     Loaded: loaded (/lib/systemd/system/rsync.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-01-02 18:33:41 KST; 15h ago
       Docs: man:rsync(1)
             man:rsyncd.conf(5)
   Main PID: 739 (rsync)
      Tasks: 1 (limit: 9830)
     Memory: 1.0M
     CGroup: /system.slice/rsync.service
             └─739 /usr/bin/rsync --daemon --no-detach

Jan 02 18:33:41 is-m1 systemd[1]: Started fast remote file copy program daemon.
Jan 02 18:33:41 is-m1 rsyncd[739]: rsyncd version 3.1.3 starting, listening on port 873
```

## rsync를 통하여 One-way sync 테스트
---

설정이 다 되었다면 단방향(one-way)로 sync를 해보면서 테스트를 진행한다.  
테스트는 dummy 파일을 생성 한 뒤 진행해본다.

```shell
dor1@is-m1 ~ ❯ cd /rsync

# dummy 파일 생성 및 확인
dor1@is-m1 /rsync ❯ fallocate -l 1G test_file
dor1@is-m1 /rsync ❯ ll
total 1048588
drwxr-xr-x  2 root root       4096 Jan  2 16:27 .
drwxr-xr-x 20 root root       4096 Jan  2 14:08 ..
-rw-rw-r--  1 deep deep 1073741824 Jan  2 16:18 test_file
```

우선 server to client을 통하여 server의 파일이 제대로 sync 되는 지 확인해본다.

```shell
# is-m1(server) to is-w1(client)
deep@is-w1:~/test$ rsync -avzurt is-m1::test /home/deep/test
receiving incremental file list
./
test_file

sent 50 bytes  received 1,044,342 bytes  77,362.37 bytes/sec
total size is 1,073,741,824  speedup is 1,028.10

# client 확인
deep@is-w1:~/test$ ll
total 1048588
drwxr-xr-x 2 deep deep       4096 Jan  2 16:27 ./
drwxr-xr-x 7 deep deep       4096 Jan  3 10:42 ../
-rw-rw-r-- 1 deep deep 1073741824 Jan  2 16:18 test_file
```

정상적으로 되는 것을 확인 하였다면, 반대로 client to server도 진행해본다.

```shell
# server에 있는 파일 삭제
dor1@is-m1 /rsync ❯ sudo rm test_file

# is-w1(client) to is-m1(server)
deep@is-w1:~/test$ rsync -avzurt /home/deep/test/test_file is-m1::test
sending incremental file list
test_file

sent 1,044,319 bytes  received 43 bytes  99,463.05 bytes/sec
total size is 1,073,741,824  speedup is 1,028.13

# server 확인
dor1@is-m1 /rsync ❯ ll
total 1048584
drwxr-xr-x  2 root root       4096 Jan  3 10:59 .
drwxr-xr-x 20 root root       4096 Jan  2 14:08 ..
-rw-rw-r--  1 deep deep 1073741824 Jan  2 16:18 test_file
```