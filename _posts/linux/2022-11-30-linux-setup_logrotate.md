---
title: Linux - logrotate 설정
date: 2022-11-30 11:45:00 +0900
categories: [Linux]
tags: [linux, logrotate]
description: Linux에서 Log의 관리를 위해 logrotate를 설정하는 방법이다.
---

>Synology DSM 7.1.1-42962 Update 2
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 작성은 **Synology DSM** 기준으로 되어있지만 대부분의 Linux에서도 동일하다.
* Log는 관리자로서 Application 등의 logging으로 error를 찾아내기 위해선 신경 써 줘야 하며, 제대로 관리를 못할 시 용량의 압박으로 인하여 OS상의 문제 또는 SSH 로그인이 안되는 현상도 생길 수 있다.
* `traefik` container를 구축 후 Log size가 너무 커서 xz로 압축하여 보관하는 방법을 설정하여 보았다.
* 기본적으로 대다수 Linux 배포판에는 `logrotate` 패키지는 설치가 되어 있다.

## 구성 및 설정
---

### 기본 패키지 구성 확인

```shell
dor1@Nukumori ~ > cat /etc/logrotate.conf
rotate 4
size 1M
create
compress
compresscmd /usr/bin/xz
compressext .xz
compressoptions -3
notifempty

include /etc/logrotate.d
include /usr/local/etc/logrotate.d
```

### 설정

기본적으로 `logrotate.conf`에 값을 입력하여 설정도 가능하겠지만, 패키지 구조 상 `/etc/logrotate.d` 이하에 넣어 설정하는 것이 관리 목적으로 좋다.

```shell
# 설정 예시
dor1@Nukumori ~ > cat /etc/logrotate.d/traefik
/volume1/docker/data/proxy-backend/traefik/log/access.log
/volume1/docker/data/proxy-backend/traefik/log/traefik.log
{
    daily
    rotate 3
    missingok
    notifempty
    compress
    dateext
    dateformat .%Y-%m-%d
    create 0644 root root
    postrotate
      docker kill --signal="USR1" $(docker ps | grep traefik | awk '{print $1}')
    endscript
}
```

위의 기준으로 확인하였을 때 설정 값은 다음과 같다.

| /volume1/docker/data/proxy-backend/traefik/log/access.log <br> /volume1/docker/data/proxy-backend/traefik/log/traefik.log | logrotate 돌릴 대상 log의 파일 경로이다. (다중 설정이 가능)                                                                              |
| ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| daily                                                                                                                     | `daily`(일) 단위로 설정한다. <br> (date 기준 00:00:00 에 작동 하며, `weekly` / `monthly` / `hourly` 등 원하는 rotate 기간으로 설정 가능) |
| rotate 3                                                                                                                  | Log 파일을 최대 3개까지 유지한다. <br> (대상 파일 포함 4개로 유지되며 4개가 초과되면 삭제 함)                                            |
| missingok                                                                                                                 | Log 파일(대상 파일이 아님)이 없는 경우에도 error 없이 다음 파일로 skip한다. <br> (missingok가 없을 시 rotate가 돌면서 error가 발생함)    |
| notifempty                                                                                                                | Log 대상 파일이 비어(0 byte) 있으면 rotate를 하지 않는다.                                                                                |
| compress                                                                                                                  | `gzip`을 통한 압축을 진행 한다.                                                                                                          |
| dateext                                                                                                                   | date(YYYY-MM-DD) 형식의 format을 갖고 생성 한다.                                                                                         |
| dateformat .%Y-%m-%d                                                                                                      | `dateext`형식에서 추가적으로 부분을 수정한다.                                                                                            |
| create 0644 root root                                                                                                     | Log 파일 생성 시 권한(0644)로 수정한다.                                                                                                  |
| postrotate                                                                                                                | rotate 파일 생성 시 다음(post) 명령어를 실행한다.                                                                                        |
| docker kill --signal="USR1" $(docker ps \| grep traefik \| awk '{print $1}')                                              | taefik container를 찾아내어 sigkill을 한다. <br> Container build 시 entrypoint 등으로 만들어진 ps를 sigkill 시키고 다시 동작하게 만든다. |
| endscript                                                                                                                 | Script 종료한다.                                                                                                                         |

### 생성 확인

```shell
# 날짜별 생성 확인
dor1@Nukumori ~ > ll /volume1/docker/data/proxy-backend/traefik/log/
total 677844
drwxr-xr-x 1 root root       192 Nov 30 00:37 .
drwxr-xr-x 1 root root        26 Jun  5 23:21 ..
-rw-r--r-- 1 root root  10407377 Nov 30 11:41 access.log
-rw-r--r-- 1 root root 665229329 Nov 30 11:41 traefik.log
-rw-r--r-- 1 root root   6356096 Nov 28 00:32 traefik.log.2022-11-28.xz
-rw-r--r-- 1 root root   6311196 Nov 29 00:34 traefik.log.2022-11-29.xz
-rw-r--r-- 1 root root   5797240 Nov 30 00:36 traefik.log.2022-11-30.xz
```