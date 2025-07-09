---
title: Traefik - http to https 적용하기
date: 2024-08-30 09:58:00 +0900
categories: [Service]
tags: [service, traefik, docker]
description: Docker에서 동작 중인 Traefik에서 http to https를 적용하는 방법이다.
---

>traefik:v3.1.2
{: .prompt-info}

>Container
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `domain.com`에 대해서 들어오는 모든 트래픽은 HSTS(HTTP Strict Transport Security)와 비슷하게 301 permanent로 처리하는 방법이다.

> ref.
> - <https://doc.traefik.io/traefik/middlewares/http/redirectscheme/>

## http to https 적용 하기
---

`redirectscheme`으로 간단히 처리가 가능하다.  
아래의 예시는 실제로 사용하는 `docker-compose.yml` 파일의 `labels`이다.

```yaml
# docker-compose.yml

#--------------------- 생략 ---------------------#
    labels:
      traefik.enable: true
      traefik.docker.network: proxy

      # ...
      
      # http-to-https
      traefik.http.routers.http-to-https.rule: Host(`$HOST_URL`) || HostRegexp(`^.+\.$HOST_URL$`)
      traefik.http.routers.http-to-https.entrypoints: web
      traefik.http.routers.http-to-https.middlewares: http-to-https@docker
      traefik.http.middlewares.http-to-https.redirectscheme.scheme: https
      traefik.http.middlewares.http-to-https.redirectscheme.permanent: true
#--------------------- 생략 ---------------------#
```

해당 파일에서 ``HostRegexp(`^.+\.$HOST_URL$`)`` 부분은 중요하다.  
`router` 구조상 처음에 진입하게 되는 곳이 `rule` 부분이며, `domain.com`과 `*domain.com`가 가장 먼저 진입하게 되는 곳이다.  
이후 `middleware`에서 `redirectscheme.scheme`처리로 https로 넘겨준다.

Dashboard를 통해서도 확인이 가능하다.

![http-to-https@docker에 대한 dashboard](/assets/img/post/service/2024-08-30-treafik-http_to_https/1.png)
_http-to-https@docker에 대한 dashboard_