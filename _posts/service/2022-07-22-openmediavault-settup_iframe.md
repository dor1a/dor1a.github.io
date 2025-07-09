---
title: OpenMediaVault(OMV) - iframe 설정
date: 2022-07-22 14:06:00 +0900
categories: [Service]
tags: [service, openmediavault]
description: OpenMediaVault에서 iframe 설정하는 방법이다.
---

>OpenMediaVault 5.6.26-1 (Usul)
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* [Organizr](https://github.com/causefx/Organizr)과 같이 iframe이 필요한 곳에서 사용하기 위하여 찾게 되었다.

## 설정 방법
---

>root 계정에서 필수로 작업 진행
{: .prompt-danger}

```shell
# shell에서 error 발생 시 exit
set -e

. /etc/default/openmediavault
. /usr/share/openmediavault/scripts/helper-functions

# XFRAME Enable
omv_set_default OMV_NGINX_SITE_WEBGUI_SECURITY_CSP_ENABLE 0
omv_set_default OMV_NGINX_SITE_WEBGUI_SECURITY_XFRAMEOPTIONS_ENABLE false

# omv 메인화면(stage)에 위의 내용 적용
omv-salt stage run prepare
omv-salt deploy run nginx
omv_purge_internal_cache
```