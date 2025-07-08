---
title: Proxmox - LXC에서 NFS mount 하기
date: 2023-09-20 17:49:00 +0900
categories: [virualization]
tags: [virualization, proxmox, lxc]
description: Proxmox의 LXC에서 NFS mount 하는 방법이다.
---

>Proxmox Virtual Environment 8.0.3
{: .prompt-info}

>Linux Container(LXC)
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `LXC`에서는 `fstab`이 작동하지 않는다.
* 이것을 해결하기 위해서`systemd`를 이용한 mount를 하면 가능할 거라고 생각하고 해보니 잘 된다.
* Proxmox에서 필수로 `lxc` option 중 `nfs`를 체크 해줘야 작동한다.

## systemd를 이용한 NFS mount
---

`systemd`는 만드는 법 자체는 간단하다.  
다만, 항목들에 대해 옵션을 잘 넣어줘야 작동하기에 작성을 잘 해줘야 한다.  
옵션에 대해서는 아래의 링크에서 확인할 수 있다.

<https://www.freedesktop.org/software/systemd/man/systemd.mount.html>

**User가 만드는 `systemd`항목은 `/etc/systemd/system`이 기본 경로이며, Kernel & system에 필요한`systemd`는 `/lib/systemd/system`가 기본 경로이다.**

```shell
# systemd 생성(항목 체크 필수)
dor1@hq-is-lxc-monitoring:~$ sudo vi /etc/systemd/system/data.mount
[Unit]
Description=Mount NFS(10.50.1.254)

[Mount]
What=10.50.1.254:/mnt/hdd/data
Where=/data
Type=nfs
Options=mountproto=udp

[Install]
WantedBy=multi-user.target

# service 실행 후 확인
dor1@hq-is-lxc-monitoring:~$ sudo systemctl start data.mount
dor1@hq-is-lxc-monitoring:~$ sudo systemctl status data.mount
* data.mount - Mount NFS(10.50.1.254)
     Loaded: loaded (/etc/systemd/system/data.mount; enabled; vendor preset: enabled)
     Active: active (mounted) since Wed 2023-09-20 09:04:56 UTC; 6s ago
      Where: /data
       What: 10.50.1.254:/mnt/hdd/data
      Tasks: 0 (limit: 462678)
     Memory: 56.0K
        CPU: 5ms
     CGroup: /system.slice/data.mount

Sep 20 09:04:56 hq-is-lxc-monitoring systemd[1]: Mounting Mount NFS(10.50.1.254)...
Sep 20 09:04:56 hq-is-lxc-monitoring systemd[1]: Mounted Mount NFS(10.50.1.254).

# mount 확인
dor1@hq-is-lxc-monitoring:~$ df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/loop0                 125G  1.4G  118G   2% /
none                       492K  4.0K  488K   1% /dev
tmpfs                      189G     0  189G   0% /dev/shm
tmpfs                       76G  128K   76G   1% /run
tmpfs                      5.0M     0  5.0M   0% /run/lock
tmpfs                       38G     0   38G   0% /run/user/1000
10.50.1.254:/mnt/hdd/data   27T  1.0M   27T   1% /data
```

reboot 시에 올라오게 하는 방법은 다른 `systemd`항목들과 똑같이 enable 하면 된다.

```shell
# service 등록
dor1@hq-is-lxc-monitoring:~$ sudo systemctl enable data.mount
Created symlink /etc/systemd/system/multi-user.target.wants/data.mount -> /etc/systemd/system/data.mount.
```