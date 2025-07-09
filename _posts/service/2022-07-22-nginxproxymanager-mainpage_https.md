---
title: Nginx Proxy Manager(NPM) - 메인 페이지 https 처리
date: 2022-07-22 14:06:00 +0900
categories: [Service]
tags: [service, nginx proxy manager, docker]
description: Docker에서 동작 중인 Nginx Proxy Manager의 메인 페이지에 대해 https 처리하는 방법이다.
---

>jc21/nginx-proxy-manager:2.9.18
{: .prompt-info}

>Container
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `https://$http_host`에 접근 시 `ERR_HTTP2_PROTOCOL_ERROR`가 뜨며 접근이 불가능 하다.
* NPM 공식 페이지에서도 따로 언급이 없어 443(https)에 대해 추가 우회에 대하여 생각하다 conf를 발견하여 처리 하게 되었다.

## Docker 설정
---

Volume mount 할 때 아래의 `site.conf` 파일을 docker image의 `/data/nginx/default_host/site.conf`에 mount 해준다.

```shell
docker run -d -v ./site.conf:/data/nginx/default_host/site.conf
```

### site.conf

`site.conf`의 내용은 다음과 같다.

```shell
# ------------------------------------------------------------
# Default Site
# ------------------------------------------------------------

# GUI에서 자동으로 생성 
server {
  listen 80 default;
  #listen [::]:80 default;

  server_name default-host.localhost;
  access_log /data/logs/default-host_access.log combined;
  error_log /data/logs/default-host_error.log warn;

  include conf.d/include/letsencrypt-acme-challenge.conf;
  location / {
    return 301 https://$http_host;
  }
}

# https(443) 추가 부분
server {
  listen 443 ssl default;
  #listen [::]:443 ssl default;

  server_name default-host.localhost;
  access_log /data/logs/default-host_access.log combined;
  error_log /data/logs/default-host_error.log warn;

  ssl_certificate /data/nginx/dummycert.pem;
  ssl_certificate_key /data/nginx/dummykey.pem;

  location / {
  return 301 http://$http_host;
  }
}
```