---
title: Ubuntu - fsnotify watcher erorr 해결
date: 2026-02-24 09:30:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, inotify, sysctl, kubernetes]
description: Kubernetes node에서 inotify watch 수 제한으로 인한 error 해결 방법이다.
---

>Ubuntu 24.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `Kubernetes`와 같은 cluster에서는 configmap, kubelet(k3s), exporter, log collector 등에서 파일을 많이 쓰기 때문에 `inotify`의 기본 값을 사용하면 금방 문제가 발생한다.
* `Kubernetes`에서는 매우 흔한 문제이니 node의 kernel parameter 변경을 통해 쉽게 변경이 가능하다.

> ref.
> - <https://man7.org/linux/man-pages/man7/inotify.7.html>

## 원인과 해결
---

잘 사용하던 k8s cluter app node에 다음과 같은 메세지가 pod마다 떠있었다.

```shell
failed to create fsnotify watcher: too many open files
```

찾아보니 `inotify`가 기본값으로 설정이 되어있었다.

```shell
admin@app_node1:~$ cat /proc/sys/fs/inotify/max_user_watches
1048576
admin@app-node1:~$ cat /proc/sys/fs/inotify/max_user_instances
128
```

`inotify`는 Linux kernel filesystem event watcher이며, 특정 파일이나 디렉토리에서의 이벤트(생성/수정/삭제/이동)가 발생하면 알려주는 역할이다.  
`inotify`는 Kernel memory에서 동작하므로 디스크 I/O 부하는 없다.  
watch 1개당 약 1KB 정도이며, **524288**로 올려도 system memory의 500MB 수준이라 부담이 없다.  
각 node마다 적용이 필요하므로 `ansible` 등으로 일괄 배포하면 간단하게 된다.  
보통 처음 `Kubernetes` 설치 시 기본으로 넣어두는 것이 편하고 좋다.

해결은 간단하다.  
Linux의 `sysctl` kernel parameter를 통해 변경하면 된다.  
`sysctl`은 필수로 **Host OS**에서 조정해야 하며 컨테이너 안에서는 변경할 수 없다.

```shell
# 영구 적용
admin@app-node1:~$ cat <<EOF | sudo tee /etc/sysctl.d/99-inotify.conf
fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=1024
EOF

# 확인
admin@app-node1:~$ sudo sysctl --system
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
kernel.sysrq = 176
* Applying /etc/sysctl.d/10-network-security.conf ...
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.all.rp_filter = 2
* Applying /etc/sysctl.d/10-ptrace.conf ...
kernel.yama.ptrace_scope = 1
* Applying /etc/sysctl.d/10-zeropage.conf ...
vm.mmap_min_addr = 65536
* Applying /usr/lib/sysctl.d/50-default.conf ...
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.default.accept_source_route = 0
sysctl: setting key "net.ipv4.conf.all.accept_source_route": Invalid argument
net.ipv4.conf.default.promote_secondaries = 1
sysctl: setting key "net.ipv4.conf.all.promote_secondaries": Invalid argument
net.ipv4.ping_group_range = 0 2147483647
net.core.default_qdisc = fq_codel
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_regular = 1
fs.protected_fifos = 1
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
kernel.pid_max = 4194304
* Applying /etc/sysctl.d/99-inotify.conf ...
fs.inotify.max_user_watches = 524288
fs.inotify.max_user_instances = 1024
* Applying /usr/lib/sysctl.d/99-protect-links.conf ...
fs.protected_fifos = 1
fs.protected_hardlinks = 1
fs.protected_regular = 2
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.conf ...
```

이후 다시 pod의 log로 가보면 error가 제거되어있는 것을 확인할 수 있다.