---
title: Proxmox - qemu-guest-agent 설치해보기
date: 2025-09-15 14:20:00 +0900
categories: [Virtualization]
tags: [virtualization, proxmox, qemu, vm]
description: Proxmox VM에 qemu-guest-agent를 설치해서 Hypervisor에서 정보를 확인 해본다.
---

>Proxmox Virtual Environment 9.0.5
{: .prompt-info}

>Hypervisor
{: .prompt-warning}

>GUI
{: .prompt-tip}

## 개요
---

* Proxmox는 **Linux KVM(Kernel-based Virtual Machine)**과 **QEMU(Quick Emulator)**를 기반으로 한 Hypervisor 이다.
* Proxmox에서 `qemu-guest-agent`를 VM에 설치하게 된다면 VM의 정보와 CPU, MEM, Network 등의 정보를 Host에 전달한다.
* Hypervisor에서는 guest VM에게 시스템 종료 등을 명령어나 GUI로 손 쉽게 제어 할 수 있다.
* Snapshot 또는 Backup 시 Guest File system을 일시정지 할 수 있다.
* Guest VM이 pause 후 resume 될때 까지 Hypervisor와 시간을 동기화 할 수 있다.
* 정리하자면 결국 Promxox의 모든 기능을 정상적으로 사용하기 위해선 Guest OS에서 필수로 설치한다.

> ref.
> - <https://pve.proxmox.com/wiki/Qemu-guest-agent>

## Proxmox VM Options에서 QEMU Quest Agent 활성화
---
Promxox에서는 기본적으로 QEMU Agent는 반가상화로 사용하게 된다.  
그러기에 `KVM hardware virtualization`은 필수로 활성화 한다.  
`KVM hardware virtualization` 옵션은 CPU의 type을 **host**로 사용 시에도 기본적으로 사용한다.

![Virtual Machine 내 Options 메뉴](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/1.png)
_Virtual Machine 내 Options 메뉴_

`KVM hardware virtualization`는 기본적으로 활성화 되어있지만 한번 더 확인 해 본다.  

![QEMU Guest Agent](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/2.png)
_QEMU Guest Agent_

이후 `QEMU Guest Agent` 메뉴에 진입 하여 활성화 한다.  
반가상화기에 VirtIO를 사용하게 된다.

## Windows에서 Agent 설치 하기
---

Windows는 Agent가 기본적으로 설치되지 않기때문에 VirtIO 관련 iso 파일을 새로 받아 설치 해야한다.

최신 버전은 다음의 링크로 받는다.  
<https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso>

다른 버전을 보기 위해선 다음과 같은 링크를 참조하면 된다.  
<https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/?C=M;O=D>

iso 파일은 VirtIO 드라이버도 같이 내장 되어있기에 Disk 인식이나 Network 인식에 사용할 수도 있다.  
만약 VM 내의 Windows 설치 시에 사용한다면 iso를 mount하여 사용하면 된다.  
설치 이후에 쓴다하면 VM 내에서 iso를 받아 mount하여 설치하면 된다.

이 글에서는 iso를 mount하여 사용해 본다.

![Mount iso](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/3.png)
_Mount iso_

우선 iso를 mount 한다.

![Virtual Machine Summary](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/4.png)
_Virtual Machine Summary_

설치 전 VM의 Summary를 확인 해보면 Memory usage도 비정상으로 보이고 IP 또한 정상적으로 보이지는 않는다.  
이제 본격적으로 설치 해본다.

![VirtIO 드라이버 설치](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/5.png)
_VirtIO 드라이버 설치_

우선 VirtIO 드라이버를 설치 해준다.  
설치 해주기 위해선 iso 내 가장 아래의 `virtio-win-gt-x64.msi` 파일로 진행하면 된다.  
VirtIO 드라이버는 Agent도 Agent지만 VirtIO 전체 기능을 쓰려면 gt 내 모든 드라이버를 설치 해야한다.  
완료 되었으면 Agent를 설치한다.

![qemu-guest-agent 설치](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/6.png)
_qemu-guest-agent 설치_

iso 내 `guest-agent\qemu-ga-x86_64.msi` 파일을 통해 설치해주면 된다.  
따로 넘기는 창은 없고 자동으로 설치 된다.

![Virtual Machine Summary](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/7.png)
_Virtual Machine Summary_

다시 Summary를 확인 해보면 정상적으로 정보를 불러오게 된다.  
이외에도 Proxmox에서 VM에 대해 `Shutdown`, `Pause`, `Hibernate` 등의 기능도 정상적으로 작동하게 된다.

## Linux에서 Agent 설치 하기
---

Linux는 간단하게 설치가 가능하다.  
VM 내에서 다음과 같이 작업한다.

```shell
# Debian 계열(기본 패키지에 있음)
apt install qemu-guest-agent

# Redhat 계열(기본 패키지에 있음)
dnf install qemu-guest-agent

# 설치 완료 후에는 systemd start & enable
systemctl start qemu-guest-agent
systemctl enable qemu-guest-agent
```

![Virtual Machine Summary](/assets/img/post/virtualization/2025-09-15-proxmox-setup_qemu-guest-agent/8.png)
_Virtual Machine Summary_

설치가 완료 된 후 Summary를 보면 정상적으로 정보들을 불러오게 된다.