---
title: Virtualization - 전가상화(Full Virtualization)와 반가상화(Para-Virtualization)의 차이
date: 2025-09-12 17:40:00 +0900
categories: [Virtualization]
tags: [virtualization, vm]
description: 전가상화(Full Virtualization)와 반가상화(Para-Virtualization)의 차이를 설명한다.
---

## 개요
---

* 전가상화(Full Virtualization)와 반가상화(Para-Virtualization)는 가상화를 구성하다보면 자주 사용하는 용어이다.
* 간단하게는 Guest OS가 자기가 가상 환경인지 아는지 모르는지의 차이다.
* 그리고 전반적으로 안정성 또는 신뢰성에 연관 되어있기에 엔지니어는 알아야하는 정의다.
* 핵심적으로 아주 간단하게는 다음과 같이 정리가 된다.
  - 전가상화: 호환성↑ / 성능↓
  - 반가상화: 호환성↓ / 성능↑

## 전가상화(Full Virtualization)
---

![Proxmox VE Administration Guide 내의 Architecture](/assets/img/post/virtualization/2025-09-12-virtualization-compare_fullvirtual_and_paravirtual/1.svg)
_Proxmox VE Administration Guide 내의 Architecture_

전가상화란 다음과 같은 정의를 내포하고 있다.

1. Guest OS가 자기 자신이 가상 환경인지를 모른다.
2. Hardware에 대해 완전한 emulation을 하여(Hypervisor) 기존 OS 그대로 사용이 가능하다.
3. Hardware emulitaion을 사용하는 만큼 Host와의 호환성이 좋다.
4. Hypervisor 내에서 처리하는 요소(virtual network, virtual disk 등)가 많아 overhead가 발생한다.

아주 간단하게 정리하자면 시장에서 많이 쓰이면서 많은 사람들이 알고있는 가상화 솔루션(VMware, Proxmox, Linux KVM 등)은 전가상화이다.

## 반가상화(Para-Virtualization)
---

![Xen Project Software Overview 내의 Architecture](/assets/img/post/virtualization/2025-09-12-virtualization-compare_fullvirtual_and_paravirtual/2.png)
_Xen Project Software Overview 내의 Architecture_

반가상화란 다음과 같은 정의를 내포하고 있다.

1. Guest OS가 자기 자신이 가상 환경인지를 알고 있다.
2. Guest OS의 Kernel을 수정하여 Hypervisor와 직접적으로 통신을 한다.  
   (ex. Xen PV, VirtIO 등)
3. Kernel에 대한 직접적인 수정이 이뤄지기에 호환성은 전가상화보다 안좋다.  
   (ex. Driver mismatch, 정해진 IO driver 사용 등)
4. Hypervisor와 바로 통신을 하기에 overhead가 적고 성능이 매우 좋다.

정리하자면 전가상화 대비 반가상화는 호환성이 좋진 않지만 추가 설정(Kernel 수정 및 driver 적용)을 해주면 성능 극대화가 가능하다.

### VirtIO

반가상화에서 필수로 알아야하는 모듈은 **VirtIO**이다.  

>VirtIO란?
* 가상 환경에서 I/O device를 위한 반가상화 된(para-virtualized) 표준 인터페이스이다.
* QEMU/KVM, Xen 같은 Hypervisor에서 Guest OS와 Host 간의 I/O 통신을 단순화 및 표준화 하기 위해 만들어졌다.
* PCI, MMIO 같은 실제 hardware bus 대신 Virtqueue(A ring buffer data structure)로 Guest와 Host 간에 데이터를 주고받음.

구조는 다음과 같다.

1. Front-end (Guest OS)  
   VirtIO driver  
   Guest OS의 kernel module 형태로 동작하며 application은 그냥 일반 device와 같이 사용한다.
2. Back-end (Host, QEMU/KVM)  
   QEMU의 VirtIO device emulation code  
   Guest에서 보낸 요청을 실제 physical hardware device나 network stack으로 연결한다.
3. Transport Layer  
   주로 PCI device 형태로 노출 됨(Guest에서는 PCI device로 인식 그리고 driver 세팅)  
   MMIO(Memory-Mapped I/O), Channel I/O 같은 다른 전송 방식도 있음.

주요 VirtIO device는 다음과 같다.
- `virtio-net`: 가상 Network NIC
- `virtio-blk`: 가상 Disk block device
- `virtio-scsi`: SCSI 기반 가상 block device
- `virtio-balloon`: 가상 System memory, 유연하게 조절이 가능함
- `virtio-rng`: Random device, 실제로는 잘 안쓰임
- `virtio-gpu`: vGPU, 보통 VDI 또는 Desktop 가상화