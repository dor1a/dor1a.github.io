---
title: Ubuntu - Kernel crash dump 설정
date: 2022-09-01 13:54:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, kernel]
description: Ubuntu에서 Kernel creash dump를 설정하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Kernel Panic 및 H/W Failure가 발생 시 Log가 찍히도록 해 준다.

> ref.
> - <https://ubuntu.com/server/docs/kernel-crash-dump>

## 설치 및 확인
---

```shell
# 설치
dor1@hodu:~$ sudo apt update && sudo apt install linux-crashdump
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Hit:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  crash kdump-tools kexec-tools libsnappy1v5 makedumpfile
The following NEW packages will be installed:
  crash kdump-tools kexec-tools libsnappy1v5 linux-crashdump makedumpfile
0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
Need to get 4,495 kB of archives.
After this operation, 13.4 MB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 kexec-tools amd64 1:2.0.22-2ubuntu2.22.04.2 [86.8 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 kdump-tools amd64 1:1.6.10ubuntu2.2 [28.5 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy/main amd64 libsnappy1v5 amd64 1.1.8-1build3 [17.5 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 crash amd64 8.0.0-1ubuntu1.1 [4,180 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 linux-crashdump amd64 5.15.0.143.138 [2,592 B]
Get:6 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 makedumpfile amd64 1:1.7.0-1ubuntu0.1 [180 kB]
Fetched 4,495 kB in 4s (1,046 kB/s)
Preconfiguring packages ...
Selecting previously unselected package kexec-tools.
(Reading database ... 81060 files and directories currently installed.)
Preparing to unpack .../0-kexec-tools_1%3a2.0.22-2ubuntu2.22.04.2_amd64.deb ...
Unpacking kexec-tools (1:2.0.22-2ubuntu2.22.04.2) ...
Selecting previously unselected package kdump-tools.
Preparing to unpack .../1-kdump-tools_1%3a1.6.10ubuntu2.2_amd64.deb ...
Unpacking kdump-tools (1:1.6.10ubuntu2.2) ...
Selecting previously unselected package libsnappy1v5:amd64.
Preparing to unpack .../2-libsnappy1v5_1.1.8-1build3_amd64.deb ...
Unpacking libsnappy1v5:amd64 (1.1.8-1build3) ...
Selecting previously unselected package crash.
Preparing to unpack .../3-crash_8.0.0-1ubuntu1.1_amd64.deb ...
Unpacking crash (8.0.0-1ubuntu1.1) ...
Selecting previously unselected package linux-crashdump.
Preparing to unpack .../4-linux-crashdump_5.15.0.143.138_amd64.deb ...
Unpacking linux-crashdump (5.15.0.143.138) ...
Selecting previously unselected package makedumpfile.
Preparing to unpack .../5-makedumpfile_1%3a1.7.0-1ubuntu0.1_amd64.deb ...
Unpacking makedumpfile (1:1.7.0-1ubuntu0.1) ...
Setting up libsnappy1v5:amd64 (1.1.8-1build3) ...
Setting up crash (8.0.0-1ubuntu1.1) ...
Setting up makedumpfile (1:1.7.0-1ubuntu0.1) ...
Setting up kexec-tools (1:2.0.22-2ubuntu2.22.04.2) ...
Generating /etc/default/kexec...
Setting up kdump-tools (1:1.6.10ubuntu2.2) ...

# Kernel crash 시 때 reboots를 도와주므로 <Yes>

 |------------------------| Configuring kexec-tools |------------------------|
 |                                                                           |
 |                                                                           |
 | If you choose this option, a system reboot will trigger a restart into a  |
 | kernel loaded by kexec instead of going through the full system boot      |
 | loader process.                                                           |
 |                                                                           |
 | Should kexec-tools handle reboots (sysvinit only)?                        |
 |                                                                           |
 |                    <Yes>                       <No>                       |
 | _________________________________________________________________________ |
 | ------------------------------------------------------------------------- |
        

# enabled에 대한 체크, <Yes>

 |------------------------| Configuring kdump-tools |------------------------|
 |                                                                           |
 |                                                                           |
 | If you choose this option, the kdump-tools mechanism will be enabled.  A  |
 | reboot is still required in order to enable the crashkernel kernel        |
 | parameter.                                                                |
 |                                                                           |
 | Should kdump-tools be enabled be default?                                 |
 |                                                                           |
 |                    <Yes>                       <No>                       |
 | _________________________________________________________________________ |
 | ------------------------------------------------------------------------- |

Creating config file /etc/default/kdump-tools with new version
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Sourcing file `/etc/default/grub.d/kdump-tools.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-143-generic
Found initrd image: /boot/initrd.img-5.15.0-143-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
Created symlink /etc/systemd/system/multi-user.target.wants/kdump-tools.service → /lib/systemd/system/kdump-tools.service.
kdump-tools-dump.service is a disabled or a static unit, not starting it.
Setting up linux-crashdump (5.15.0.143.138) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for initramfs-tools (0.140ubuntu13.5) ...
update-initramfs: Generating /boot/initrd.img-5.15.0-143-generic
Processing triggers for libc-bin (2.35-0ubuntu3.10) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

### 구성 확인

서비스 확인 후 config도 같이 확인 해 본다.

```shell
# 서비스 확인
dor1@hodu:~$ sudo systemctl status kdump-tools
● kdump-tools.service - Kernel crash dump capture service
     Loaded: loaded (/lib/systemd/system/kdump-tools.service; enabled; vendor preset: enabled)
     Active: active (exited) since Mon 2022-09-14 01:45:21 UTC; 1min 16s ago
    Process: 3022 ExecStart=/etc/init.d/kdump-tools start (code=exited, status=0/SUCCESS)
   Main PID: 3022 (code=exited, status=0/SUCCESS)
        CPU: 24ms

Sep 14 01:45:21 test systemd[1]: Starting Kernel crash dump capture service...
Sep 14 01:45:21 test kdump-tools[3022]: Starting kdump-tools:
Sep 14 01:45:21 test kdump-tools[3027]:  * no crashkernel= parameter in the kernel cmdline
Sep 14 01:45:21 test systemd[1]: Finished Kernel crash dump capture service.

# config 확인
dor1@hodu:~$ sudo kdump-config show
DUMP_MODE:              kdump
USE_KDUMP:              1
KDUMP_COREDIR:          /var/crash
crashkernel addr: 0xbf000000
   /var/lib/kdump/vmlinuz: symbolic link to /boot/vmlinuz-5.15.0-46-generic
kdump initrd:
   /var/lib/kdump/initrd.img: symbolic link to /var/lib/kdump/initrd.img-5.15.0-46-generic
current state:    ready to kdump

kexec command:
  /sbin/kexec -p --command-line="BOOT_IMAGE=/boot/vmlinuz-5.15.0-46-generic root=UUID=ef2e8f1d-36ba-4cd3-81e8-711aada16075 ro reset_devices systemd.unit=kdump-tools-dump.service nr_cpus=1 irqpoll nousb" --initrd=/var/lib/kdum.img /var/lib/kdump/vmlinuz
```

## kdump 동작 확인 테스트
---

sysRQ의 mechanism을 이용하여 kdump가 실제로 동작하는지 테스트 해본다.  
rebooting이 필요하다.

```shell
# kenrel.sysrq 확인
dor1@hodu:~$ cat /proc/sys/kernel/sysrq
176

# sysRQ의 value를 1로 조정하여 dump enable
dor1@hodu:~$ sudo sysctl -w kernel.sysrq=1

# (Console에서 작업) sysRQ의 trigger 수정하여 동작 확인, 작업이 되면 자동으로 reboot
dor1@hodu:~$ sudo -i
root@hodu:~$ echo c > /proc/sysrq-trigger
[   31.659002] SysRq : Trigger a crash
[   31.659749] BUG: unable to handle kernel NULL pointer dereference at           (null)
[   31.662668] IP: [<ffffffff8139f166>] sysrq_handle_crash+0x16/0x20
[   31.662668] PGD 3bfb9067 PUD 368a7067 PMD 0 
[   31.662668] Oops: 0002 [#1] SMP 
[   31.662668] CPU 1 
...
Begin: Saving vmcore from kernel crash ...

# reboot 이후 crash log 확인
dor1@hodu:~$ ls -alh /var/crash
total 12
drwxrwxrwt  2 root root 4096 Sep  1 11:44 .
drwxr-xr-x 12 root root 4096 May 24 17:49 ..
-rw-r--r--  1 root root  259 Sep  1 15:06 kexec_cmd
-rw-r--r--  1 root root  259 Sep  1 15:11 linux-image-5.15.0-46-generic-202209011511.crash
```