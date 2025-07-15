---
title: Proxmox - hostname 변경
date: 2023-08-04 17:35:00 +0900
categories: [Virtualization]
tags: [virtualization, proxmox]
description: Proxmox의 Hypervisor에서 hostname을 변경하는 방법이다.
---

>Proxmox Virtual Environment 7.4-3
{: .prompt-info}

>Hypervisor
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Proxmox의 node에 대해 hostname 변경이 필요하게 되어 진행하게 되었다.

> ref.
> - <https://pve.proxmox.com/wiki/Renaming_a_PVE_node> 

## hostname 변경 방법
---

Proxmox가 `debian`기반이지만 `hostnamectl set-hostname <hostname>`만을 갖고 변경이 되지는 않는다.  
config 파일에 대한 변경도 필수적으로 필요하기 때문에 다음과 같은 명령어를 통하여 진행한다.

```shell
# NEW_HOSTNAME 변경 필수
echo "OLD_HOSTNAME=$HOSTNAME" > ~/pverename.ini
echo "NEW_HOSTNAME=is-test" >> ~/pverename.ini

# 설정값 적용
source <(grep = ~/pverename.ini)

# hostname 변경 후 .bak 파일에 기존 설정 저장
sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/gi" /etc/hostname
sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/gi" /etc/hosts

# main.cf 존재할 시 진행
[ -e "/etc/postfix/main.cf" ] && sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/gi" /etc/postfix/main.cf

# copy config files to new node name
cp -rv /var/lib/rrdcached/db/pve2-node/$OLD_HOSTNAME /var/lib/rrdcached/db/pve2-node/$NEW_HOSTNAME
cp -rv /var/lib/rrdcached/db/pve2-storage/$OLD_HOSTNAME /var/lib/rrdcached/db/pve2-storage/$NEW_HOSTNAME
```

이후 **reboot**을 해준다.  
**reboot**은 해당 설정이 우선 Proxmox에 적용이 될 수 있게 한다.  
설정이 된다면 다음과 같은 명령어를 통하여 마무리 해준다.

```shell
# read variables from ini
source <(grep = ~/pverename.ini)

# update storage config
sed -i.bak "s/nodes $OLD_HOSTNAME/nodes $NEW_HOSTNAME/gi" /etc/pve/storage.cfg

# mv vm configs
mv -v /etc/pve/nodes/$OLD_HOSTNAME/qemu-server/*.conf /etc/pve/nodes/$NEW_HOSTNAME/qemu-server/

# mv lxc configs
mv -v /etc/pve/nodes/$OLD_HOSTNAME/lxc/*.conf /etc/pve/nodes/$NEW_HOSTNAME/lxc/
```

## Clean Up
---

위의 작업이 다 끝나게 되면 필수로 **reboot** 후 진행한다.

```shell
# read variables from ini
source <(grep = ~/pverename.ini)
rm /etc/hostname.bak && rm /etc/hosts.bak
[ -e "/etc/postfix/main.cf.bak" ] && rm /etc/postfix/main.cf.bak
rm /var/lib/rrdcached/db/pve2-node/$OLD_HOSTNAME
rm -r /var/lib/rrdcached/db/pve2-storage/$OLD_HOSTNAME
rm -r /etc/pve/nodes/$OLD_HOSTNAME
rm /etc/pve/storage.cfg.bak
rm ~/pverename.ini
```