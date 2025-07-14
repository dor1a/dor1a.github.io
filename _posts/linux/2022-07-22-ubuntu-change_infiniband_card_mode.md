---
title: Ubuntu - Mellanox Infiniband(IB) VPI 카드 mode 변경
date: 2022-07-22 14:09:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, mellanox, infiniband]
description: Ubuntu에서 Mellanox Infiniband VPI 카드에 대해 mode(IB to ETH)를 변경하는 방법이다.
---

>Ubuntu 20.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Mellanox의 VPI 카드에서는 Ethernet 또는 Infiniband로 mode 변경이 가능 하다.
* 테스트 모델: `Mellanox ConnectX-6 VPI Card / MLNX_OFED_LINUX-5.4-3.1.0.0`
* NVIDIA와 Mellanox의 Docs는 쓰레기라서 매번 변경되기 때문에 ***ref.***를 따로 첨부하진 않는다.

## 변경 방법
---

다음의 명령어를 따라 쳐주면 된다.

```shell
# Mellanox Software Tools (mst) 서비스 시작
sudo mst start

# mlxconfig에서 사용할 device number 확인
sudo ibv_devinfo | grep -i vendor_part_id

# LINK_TYPE 확인
sudo mlxconfig -d /dev/mst/mt4123_pciconf0 q | grep -i link_type

# LINK_TYPE 변경 (LINK_TYPE_P1은 물리적으로 1번 포트를 의미)
# IB(1) / Ethernet(2)
sudo mlxconfig -d /dev/mst/mt4123_pciconf0 set LINK_TYPE_P1=2
```