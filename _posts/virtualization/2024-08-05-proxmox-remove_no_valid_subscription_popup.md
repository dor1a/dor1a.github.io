---
title: Proxmox - "No valid subscription" 창 없애기
date: 2024-08-05 11:28:00 +0900
categories: [Virtualization]
tags: [virtualization, proxmox, lxc]
description: Proxmox의 Hypervisor에서 귀찮게 하는 "No valid subscription" 창을 없애는 방법이다.
---

>Proxmox Virtual Environment 8.2.4
{: .prompt-info}

>Hypervisor
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Proxmox에서 귀찮게 하는 **No valid subscription** 창을 없애기 위해서 알아봤다.

![No valid subscription](/assets/img/post/virtualization/2024-08-05-proxmox-remove_no_valid_subscription_popup/1.png)
_No valid subscription_

> ref.
> - <https://svrforum.com/os/138940>

## 수동으로 창 없애기
---

Proxmox은 오픈소스로 운영되고 있지만, Enterprise repository & support를 받는 경우 license를 별도로 구매할 수 있다.  
다만, 보통 홈 서버와 같이 운영하는 경우 굳이 연간 10만원이 넘는 돈을 지불하기에는 부담도 되고 enterprise의 서비스를 굳이 받을 필요도 없다고 생각한다.  
위의 창이 계속 귀찮게 굴기 때문에 없애 주는것도 현명하다고 생각한다.

### 1. title이 있는 곳에서 void 처리

Proxmox는 widget으로 front의 창을 띄우기 때문에 js 파일에 대해 처리하면 된다.

```shell
# bak 파일 처리
dor1@proxmox:~$ cd /usr/share/javascript/proxmox-widget-toolkit
dor1@proxmox:/usr/share/javascript/proxmox-widget-toolkit$ sudo cp proxmoxlib.js proxmoxlib.js.bak

# 편집기로 proxmoxlib.js의 title 부분을 void 처리
dor1@proxmox:/usr/share/javascript/proxmox-widget-toolkit$ sudo vi proxmoxlib.js
```

아래와 같이 처리한다.

![/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js](/assets/img/post/virtualization/2024-08-05-proxmox-remove_no_valid_subscription_popup/2.png)
_/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js_

### 2. systemd 재시작(pveproxy.service)

```shell
# 재시작 시 웹에서 보는 frontend는 끊킬 수 있음
dor1@proxmox:/usr/share/javascript/proxmox-widget-toolkit$ sudo systemctl restart pveproxy.service

# 확인
dor1@proxmox:/usr/share/javascript/proxmox-widget-toolkit$ sudo systemctl status pveproxy.service
● pveproxy.service - PVE API Proxy Server
     Loaded: loaded (/lib/systemd/system/pveproxy.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-08-05 12:15:21 KST; 25s ago
    Process: 3739660 ExecStartPre=/usr/bin/pvecm updatecerts --silent (code=exited, status=0/SUCCESS)
    Process: 3739672 ExecStart=/usr/bin/pveproxy start (code=exited, status=0/SUCCESS)
   Main PID: 3739691 (pveproxy)
      Tasks: 4 (limit: 462607)
     Memory: 183.5M
        CPU: 2.052s
     CGroup: /system.slice/pveproxy.service
             ├─3739691 pveproxy
             ├─3739692 "pveproxy worker"
             ├─3739693 "pveproxy worker"
             └─3739694 "pveproxy worker"

Aug 05 12:15:19 proxmox systemd[1]: Starting pveproxy.service - PVE API Proxy Server...
Aug 05 12:15:21 proxmox pveproxy[3739691]: starting server
Aug 05 12:15:21 proxmox pveproxy[3739691]: starting 3 worker(s)
Aug 05 12:15:21 proxmox pveproxy[3739691]: worker 3739692 started
Aug 05 12:15:21 proxmox pveproxy[3739691]: worker 3739693 started
Aug 05 12:15:21 proxmox pveproxy[3739691]: worker 3739694 started
Aug 05 12:15:21 proxmox systemd[1]: Started pveproxy.service - PVE API Proxy Server.
```

## Script로 간단히 없애기
---

한 줄로 간단하게 가능하다.

```shell
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

