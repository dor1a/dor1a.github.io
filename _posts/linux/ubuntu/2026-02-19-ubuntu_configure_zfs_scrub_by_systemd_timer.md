---
title: Ubuntu - ZFS Scrub을 systemd timer로 자동화하기
date: 2026-02-19 13:00:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, systemd, zfs]
description: cron 대신 systemd timer를 사용하여 ZFS Scrub을 주기적으로 자동화하는 방법이다.
---

>Ubuntu 24.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* ZFS 파일시스템의 데이터 무결성을 유지하기 위해 주기적으로 `zpool scrub`을 실행해야 한다.  
  `zpool scrub`은 ZFS의 모든 data block을 읽어 checksum을 검증하는 작업으로 Mirror 또는 RAID-Z 구성에서는 손상된 블록을 자동으로 복구해준다.  
  scrub 주기는 데이터 중요도에 따라 조절하지만 일반적으로 월 1회 또는 분기에 1회를 권장한다.
* 설정은 `cron`으로도 가능하지만 `systemd timer`를 사용하면 `journalctl`을 통한 log 확인이 가능하며, `systemctl list-timers`를 통한 스케줄 확인 등 운영 편의성이 크게 향상된다.
* `Persistent=true` 옵션을 통해 서버가 꺼져있어 스케줄을 놓쳤을 때에도 부팅 후 자동 실행이 보장된다.

> ref.
> - <https://openzfs.github.io/openzfs-docs/man/master/8/zpool-scrub.8.html>
> - <https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html?__goaway_challenge=meta-refresh&__goaway_id=79347bbe0cded4676842bf657a2846db&__goaway_referer=https%3A%2F%2Fwww.google.com%2F>

## systemd timer가 cron보다 나은 점
---

`cron`의 경우 log 확인을 위해 별도 설정이 필요하지만 `systemd timer`는 `journalctl -u <service_name>`으로 바로 확인할 수 있다.  
스케줄 확인도 `crontab -l` 대신 `systemctl list-timers`를 사용하면 다음 실행 시점과 마지막 실행 결과를 한눈에 볼 수 있다.  
가장 큰 차이는 `Persistent=true` 옵션인데 서버가 꺼져있어 예정된 실행을 놓쳤을 경우 부팅 후 자동으로 실행해준다.  
`cron`에서는 이러한 기능이 없어 놓친 스케줄은 그대로 넘어간다.  
그 외에도 `Restart=on-failure` 같은 재시도 설정이나 `After=` 또는 `Wants=` 등의 의존성 관리를 선언적으로 설정할 수 있다는 점에서 `systemd timer`가 운영 편의성이 높다.

## 구성하기
---

새롭게 파일을 생성하여 구성한다.
`/etc/systemd/system` 이하는 실제로 `systemd`에서 사용하는 config들이 존재하는 곳이다.

### 1. service 작성

`zpool scrub` 명령은 kernel에 scrub 요청만 하고 즉시 return(non-blocking)하므로, 여러 zpool을 `ExecStart`에 나열하면 사실상 거의 동시에 scrub이 시작된다.

```shell
# 새로 작성
admin@ubuntu-2404-test:~$ sudo vi /etc/systemd/system/zfs-scrub.service

[Unit]
Description=ZFS Scrub All Pools

[Service]
Type=oneshot
ExecStart=/sbin/zpool scrub raid-z1
ExecStart=/sbin/zpool scrub raid-z1-ssd
```

`Type=oneshot`에서 `ExecStart`를 여러 개 사용하면 순차적으로 실행된다.  
다만 `zpool scrub`은 non-blocking이므로 첫 번째 명령이 즉시 return하여 두 번째도 바로 실행된다.  
zpool이 물리적으로 다른 디스크라면 병렬로 돌아가도 문제없다.

### 2. timer 작성

1월부터 3개월 마다 분기별 기준으로 1일 새벽 2시에 scrub을 실행하도록 설정한다.

```shell
# 새로 작성
admin@ubuntu-2404-test:~$ sudo vi /etc/systemd/system/zfs-scrub.timer

[Unit]
Description=ZFS Scrub Every 3 Months

[Timer]
OnCalendar=*-01,04,07,10-01 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

`Persistent=true`는 서버가 꺼져있어 예정된 실행을 놓쳤을 경우 부팅 후 즉시 실행해주는 옵션이다.

### 3. 적용 및 확인

```shell
# systemd에 config 인식
admin@ubuntu-2404-test:~$ sudo systemctl daemon-reload

# timer 활성화 및 시작
admin@ubuntu-2404-test:~$ sudo systemctl enable --now zfs-scrub.timer
Created symlink /etc/systemd/system/timers.target.wants/zfs-scrub.timer → /etc/systemd/system/zfs-scrub.timer.

# timer 등록 확인
admin@ubuntu-2404-test:~$ sudo systemctl list-timers | grep zfs
Wed 2026-04-01 02:00:00 KST 1 month 10 days -                                      - zfs-scrub.timer                zfs-scrub.service
```

## 수동 실행 및 log 확인
---

timer를 기다리지 않고 수동으로 scrub을 실행하려면 service를 직접 start 하면 된다.

```shell
# 수동 실행
admin@ubuntu-2404-test:~$ sudo systemctl start zfs-scrub.service

# scrub 진행 상태 확인
admin@ubuntu-2404-test:~$ sudo zpool status
  pool: raid-z1
 state: ONLINE
  scan: scrub in progress since Thu Feb 19 14:14:02 2026
        21.1T / 21.1T scanned, 49.4G / 21.1T issued at 790M/s
        0B repaired, 0.23% done, 07:46:04 to go
config:

        NAME        STATE     READ WRITE CKSUM
        raid-z1     ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            sda     ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0
            sdd     ONLINE       0     0     0

errors: No known data errors

  pool: raid-z1-ssd
 state: ONLINE
  scan: scrub in progress since Thu Feb 19 14:15:36 2026
        326G / 326G scanned, 9.02G / 326G issued at 769M/s
        0B repaired, 2.77% done, 00:07:01 to go
config:

        NAME         STATE     READ WRITE CKSUM
        raid-z1-ssd  ONLINE       0     0     0
          raidz1-0   ONLINE       0     0     0
            sde      ONLINE       0     0     0
            sdf      ONLINE       0     0     0
            sdg      ONLINE       0     0     0
            sdh      ONLINE       0     0     0

errors: No known data errors

# service log 확인
admin@ubuntu-2404-test:~$ journalctl -u zfs-scrub.service
Feb 19 14:15:36 atsui systemd[1]: Starting zfs-scrub.service - ZFS Scrub All Pools...
Feb 19 14:15:37 atsui systemd[1]: zfs-scrub.service: Deactivated successfully.
Feb 19 14:15:37 atsui systemd[1]: Finished zfs-scrub.service - ZFS Scrub All Pools.
```