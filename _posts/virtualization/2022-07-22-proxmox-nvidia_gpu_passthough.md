---
title: Proxmox - NVIDIA GPU Passthrough
date: 2022-07-22 11:27:00 +0900
categories: [Virualization]
tags: [virualization, proxmox, nvidia, vm]
description: Proxmox의 VM에서 NVIDIA GPU Passthrough를 사용하는 방법이다.
---

>Proxmox Virtual Environment 7.2-3
{: .prompt-info}

>Virtual Machine(VM)
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

>PVE 8버전부터는 BIOS 및 VT 설정만 되어있으면 아래의 작업 없이도 사용 가능하다.
{: .prompt-info}

* Test-bed로 GPU가 달려있는 VM이 필요하여 진행하게 되었다.
* BIOS에서 `IOMMU` 세팅이 필수적으로 필요하다.
* BIOS에서 `Intel VT-D`와 같은 세팅이 필수적으로 필요하다.
* Proxmox는 `debian` 기반으로 이루어져 있다.
* `VFIO(UEFI)`가 기본적으로 설치되어 있어 비교적 `VMware` 대비 진행이 간단하다.

## Passthrough 방법
---

Proxmox host machine에서 Shell에 접근 또는 ssh로 접속하여 다음과 같이 진행한다.

```shell
# HW ID 값 확인
root@pve:~$ lspci -nn | grep -i nvidia
06:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
07:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
0a:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
0b:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
85:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
86:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
89:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)
8a:00.0 3D controller [0302]: NVIDIA Corporation GP100GL [Tesla P100 SXM2 16GB] [10de:15f9] (rev a1)

root@pve:~$ vi /etc/default/grub
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
# GRUB DEFAULT 부분 추가(위에서 알아낸 HW ID값 또한 같이 입력)
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on vfio-pci.ids=10de:15f9"
GRUB_CMDLINE_LINUX=""

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
#GRUB_GFXMODE=640x480

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"

# grub 적용
root@pve:~$ update-grub
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.30-2-pve
Found initrd image: /boot/initrd.img-5.15.30-2-pve
Found memtest86+ image: /boot/memtest86+.bin
Found memtest86+ multiboot image: /boot/memtest86+_multiboot.bin
Adding boot menu entry for EFI firmware configuration
done

# 모듈 값 추가
root@pve:~$ echo "vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd" | tee -a /etc/modules
```

전부 추가 되었으면 이후 **rebootng**을 해준다.  
VM 생성 시 GPU 정상 사용을 하려면 GPU 메모리의 2배 정도 시스템 메모리를 할당 해주면 된다.  
생성 후 다음과 같이 진행하면 정상적으로 사용이 가능하다.

1. VM에서 좌측 `Hardware` 탭
2. Add `PCI Device`에서 사용 할 GPU를 추가
3. OS 설치 진행
4. NVIDIA 드라이버 설치 진행