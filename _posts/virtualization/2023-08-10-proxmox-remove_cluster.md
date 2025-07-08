---
title: Proxmox - Cluster 삭제
date: 2023-08-10 10:55:00 +0900
categories: [virualization]
tags: [virualization, proxmox, cluster]
description: Proxmox의 Hypervisor에서 Cluster를 삭제하는 방법이다.
---

>Proxmox Virtual Environment 7.4-3
{: .prompt-info}

>Hypervisor
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Proxmox에서 잘못 생성 된 Cluster에 대해 삭제가 필요하여 적어보았다.

## Cluster 삭제 방법
---

Proxmox Cluster는 `corosync`로 운영이 된다.

`corosync`에 대해서만 설정하면 되지만 PVE의 VM 및 LXC에도 영향이 가기 때문에 `pve-cluster` 서비스를 같이 내린 후 작업을 완료하고 올려주면 된다.

방법은 간단하다.

```shell
# service stop(pve-cluster, corosync)
root@is-pve1:~$ systemctl stop pve-cluster corosync

# pmxcfs(proxmox cluster filesystem)에 대해 localizing
root@is-pve1:~$ pmxcfs -l
[main] notice: forcing local mode (although corosync.conf exists)

# corosync config 삭제
root@is-pve1:~$ rm -r /etc/corosync/* && rm /etc/pve/corosync.conf

# pmxcfs(proxmox cluster filesystem)에 대해 process kill
root@is-pve1:~$ killall pmxcfs

# service start(pve-cluster)
root@is-pve1:~$ systemctl start pve-cluster
```

이후 웹에서 확인하면 정상적으로 Cluster가 없어진 것을 확인 할 수 있다.