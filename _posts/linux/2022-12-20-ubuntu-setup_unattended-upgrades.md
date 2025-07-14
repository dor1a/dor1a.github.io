---
title: Ubuntu - unattended-upgrades 설정
date: 2022-12-20 12:11:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, unattended-upgrades]
description: Ubuntu에서 자동으로 apt upgrade 하는 것을 방지하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `unattended-upgrades`는 사전적인 의미와 같이 혼자 또는 무인이(unattended), 업그레이드(Upgrade) 하는 것이며, Background에서 자동 upgrade를 한다는 것과 같다.
* `crontab` 기반으로 돌아가며, 새벽마다 자동으로 stable 패키지에 대하여 자동으로 업그레이드 된다.
* GPU 사용 및 Kernel 패키지를 사용하다 보면 OS 업그레이드를 막아줘야 Kernel 패키지가 정상적으로 작동하여 신경을 써 줘야 하는데 이럴 때 꼭 필요하다.

## 설치
---

Ubuntu 사용한다면 기본적으로 설치가 되어있다.

```shell
dor1@is-m1 ~ ❯ sudo apt install unattended-upgrades -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
unattended-upgrades is already the newest version (2.3ubuntu0.3).
unattended-upgrades set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 13 not upgraded.
```

서비스를 보면 다음과 같이 정상적으로 작동하는 것을 볼 수 있다.

```shell
dor1@is-m1 ~ ❯ sudo systemctl status unattended-upgrades
● unattended-upgrades.service - Unattended Upgrades Shutdown
     Loaded: loaded (/lib/systemd/system/unattended-upgrades.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-12-19 14:29:38 KST; 21h ago
       Docs: man:unattended-upgrade(8)
   Main PID: 796 (unattended-upgr)
      Tasks: 2 (limit: 9830)
     Memory: 10.8M
     CGroup: /system.slice/unattended-upgrades.service
             └─796 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal

Dec 19 14:29:38 is-m1 systemd[1]: Started Unattended Upgrades Shutdown.
```

## 설정 확인 및 재구성
---

### 1. 설정 확인

`unattended-upgrades`에 들어가 있는 설정 값 중 핵심은 다음과 같다.

```shell
dor1@is-m1 ~ ❯ cat /etc/apt/apt.conf.d/50unattended-upgrades | egrep -v "//|^$"
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        "${distro_id}ESMApps:${distro_codename}-apps-security";
        "${distro_id}ESM:${distro_codename}-infra-security";
};
Unattended-Upgrade::Package-Blacklist {
};
Unattended-Upgrade::DevRelease "auto";
```

기본적으로 `/etc/apt/source.list`에 등록되어있는 Repository의 4개에 대해서는 Allowed 되어있다.  
`unattended-upgrades`가 수행되는 시간은 `date` 기준으로 진행되며 `crontab`으로 확인 시 매일 06:25분에 진행된다.

```shell
# crontab 확인
dor1@is-m1 ~ ❯ cat /etc/crontab | grep -i daily
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )

# crontab의 apt-compat 동작 수행 확인
dor1@is-m1 ~ ❯ cat /etc/cron.daily/apt-compat  | tail -1
exec /usr/lib/apt/apt.systemd.daily
```

### 2. Logging

Log는 `/var/log/unattened-upgrades`에 지속적으로 기록된다.  
또한 `/var/log/apt/history.log`에도 업그레이드 내역에 대해서 확인이 가능하다.

```shell
# /var/log/unattended-upgrades 이하 기록 내용
dor1@is-m1 ~ ❯ sudo cat /var/log/unattended-upgrades/unattended-upgrades-dpkg.log | tail -15
Log started: 2022-12-15  06:36:56
Preconfiguring packages ...
Preconfiguring packages ...
(Reading database ... 79343 files and directories currently installed.)
Preparing to unpack .../tzdata_2022g-0ubuntu0.20.04.1_all.deb ...
Unpacking tzdata (2022g-0ubuntu0.20.04.1) over (2022f-0ubuntu0.20.04.1) ...
Setting up tzdata (2022g-0ubuntu0.20.04.1) ...

Current default time zone: 'Asia/Seoul'
Local time is now:      Thu Dec 15 06:36:57 KST 2022.
Universal Time is now:  Wed Dec 14 21:36:57 UTC 2022.
Run 'dpkg-reconfigure tzdata' if you wish to change it.

Log ended: 2022-12-15  06:36:59

# /var/log/apt/history.log 기록 내용
dor1@is-m1 ~ ❯ cat /var/log/apt/history.log | tail -5

Start-Date: 2022-12-15  06:36:56
Commandline: /usr/bin/unattended-upgrade
Upgrade: tzdata:amd64 (2022f-0ubuntu0.20.04.1, 2022g-0ubuntu0.20.04.1)
End-Date: 2022-12-15  06:36:57
```

### 3. Service

Service는 enabled & disabled로 처리가 가능하다.

```shell
dor1@is-m1 ~ ❯ sudo dpkg-reconfigure unattended-upgrades
```

![CLI UI](/assets/img/post/linux/2022-12-20-ubuntu-setup_unattended-upgrades/1.png)
_CLI UI_

간단히 `<Yes>`하면 enabled, `<No>` 하면 disabled 이다.

## Package-Blacklist & Automatic-Reboot
---

추가적으로 몇 개의 기능이 더 있는데, 그 중 2가지인 `Package-Blacklist`와 `Automatic-Reboot`기능이 있다.  
`/etc/apt/apt.conf.d/50unattended-upgrades`에서 `Unattended-Upgrade::Package-Blacklist` 영역에 내용을 추가하여, `unattended-upgrades` 서비스가 특정 패키지에 대해 업그레이드를 못하도록 하는 설정한다.  
작성 후 저장하면 `crontab`에 맞춰 자동으로 적용이 된다.

```shell
dor1@is-m1 ~ ❯ sudo vi /etc/apt/apt.conf.d/50unattended-upgrades
// Automatically upgrade packages from these (origin:archive) pairs
//
// Note that in Ubuntu security updates may pull in new dependencies
// from non-security sources (e.g. chromium). By allowing the release
// pocket these get automatically pulled in.
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        // Extended Security Maintenance; doesnt necessarily exist for
        // every release and this system may not have it installed, but if
        // available, the policy for updates is such that unattended-upgrades
        // should also install from here by default.
        "${distro_id}ESMApps:${distro_codename}-apps-security";
        "${distro_id}ESM:${distro_codename}-infra-security";
//      "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};

// Python regular expressions, matching packages to exclude from upgrading

# Pacakage-Blacklist의 내용 추가 부분
Unattended-Upgrade::Package-Blacklist {
    // The following matches all packages starting with linux-
//  "linux-";

    // Use $ to explicitely define the end of a package name. Without
    // the $, "libc6" would match all of them.
//  "libc6$";
//  "libc6-dev$";
//  "libc6-i686$";

    // Special characters need escaping
//  "libstdc\+\+6$";

    // The following matches packages like xen-system-amd64, xen-utils-4.1,
    // xenstore-utils and libxenstore3.0
//  "(lib)?xen(store)?";

    // For more information about Python regular expressions, see
    // https://docs.python.org/3/howto/regex.html
};
#------------------------------------------------------------------#

// This option controls whether the development release of Ubuntu will be
// upgraded automatically. Valid values are "true", "false", and "auto".
Unattended-Upgrade::DevRelease "auto";

// This option allows you to control if on a unclean dpkg exit
// unattended-upgrades will automatically run
//   dpkg --force-confold --configure -a
// The default is true, to ensure updates keep getting installed
//Unattended-Upgrade::AutoFixInterruptedDpkg "true";

// Split the upgrade into the smallest possible chunks so that
// they can be interrupted with SIGTERM. This makes the upgrade
// a bit slower but it has the benefit that shutdown while a upgrade
// is running is possible (with a small delay)
//Unattended-Upgrade::MinimalSteps "true";

// Install all updates when the machine is shutting down
// instead of doing it in the background while the machine is running.
// This will (obviously) make shutdown slower.
// Unattended-upgrades increases loginds InhibitDelayMaxSec to 30s.
// This allows more time for unattended-upgrades to shut down gracefully
// or even install a few packages in InstallOnShutdown mode, but is still a
// big step back from the 30 minutes allowed for InstallOnShutdown previously.
// Users enabling InstallOnShutdown mode are advised to increase
// InhibitDelayMaxSec even further, possibly to 30 minutes.
//Unattended-Upgrade::InstallOnShutdown "false";

// Send email to this address for problems or packages upgrades
// If empty or unset then no email is sent, make sure that you
// have a working mail setup on your system. A package that provides
// 'mailx' must be installed. E.g. "user@example.com"
//Unattended-Upgrade::Mail "";

// Set this value to one of:
//    "always", "only-on-error" or "on-change"
// If this is not set, then any legacy MailOnlyOnError (boolean) value
// is used to chose between "only-on-error" and "on-change"
//Unattended-Upgrade::MailReport "on-change";

// Remove unused automatically installed kernel-related packages
// (kernel images, kernel headers and kernel version locked tools).
//Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

// Do automatic removal of newly unused dependencies after the upgrade
//Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

// Do automatic removal of unused packages after the upgrade
// (equivalent to apt-get autoremove)
//Unattended-Upgrade::Remove-Unused-Dependencies "false";

# Automatic-Reboot
// Automatically reboot *WITHOUT CONFIRMATION* if
//  the file /var/run/reboot-required is found after the upgrade
# Reboot의 여부 설정
//Unattended-Upgrade::Automatic-Reboot "false";

// Automatically reboot even if there are users currently logged in
// when Unattended-Upgrade::Automatic-Reboot is set to true
# Login 한 user에 대해 reoobt 여부 설정
//Unattended-Upgrade::Automatic-Reboot-WithUsers "true";

// If automatic reboot is enabled and needed, reboot at the specific
// time instead of immediately
//  Default: "now"
# Reboot 시간 설정
//Unattended-Upgrade::Automatic-Reboot-Time "02:00";
#------------------------------------------------------------------#

// Use apt bandwidth limit feature, this example limits the download
// speed to 70kb/sec
//Acquire::http::Dl-Limit "70";

// Enable logging to syslog. Default is False
// Unattended-Upgrade::SyslogEnable "false";

// Specify syslog facility. Default is daemon
// Unattended-Upgrade::SyslogFacility "daemon";

// Download and install upgrades only on AC power
// (i.e. skip or gracefully stop updates on battery)
// Unattended-Upgrade::OnlyOnACPower "true";

// Download and install upgrades only on non-metered connection
// (i.e. skip or gracefully stop updates on a metered connection)
// Unattended-Upgrade::Skip-Updates-On-Metered-Connections "true";

// Verbose logging
// Unattended-Upgrade::Verbose "false";

// Print debugging information both in unattended-upgrades and
// in unattended-upgrade-shutdown
// Unattended-Upgrade::Debug "false";

// Allow package downgrade if Pin-Priority exceeds 1000
// Unattended-Upgrade::Allow-downgrade "false";
```