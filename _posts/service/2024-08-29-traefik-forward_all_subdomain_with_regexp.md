---
title: Traefik - Regexp로 모든 subdomain에 대하여 forward 처리하기
date: 2024-08-29 17:19:00 +0900
categories: [Service]
tags: [service, traefik, docker]
description: Docker에서 동작 중인 Traefik에서 Regexp로 모든 subdomain에 대하여 forward 처리하는 방법이다.
---

>traefik:v3.1.2
{: .prompt-info}

>Container
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Traefik은 `haproxy`, `nginx`와 같이 proxy 해주는 강력한 서비스이다.  
  Docker 및 Kubernetes와 같은 container base의 platform에서도 강력한 기능은 자랑하며 쉽게 호환되게 만들어 놨다.
* 기본적으로 L7(http)로 사용하는경우가 거의 다 이며, 특수한 경우 L4(TCP)로 사용도 가능하다.
* `router`, `middleware`와 같은 용어가 어색할 수 있으머 구성에 어려움을 느낄 수 있지만, web server 및 header를 만지던 사람에게는 나름대로 쉬울 수 있다고 생각한다.
* CNAME으로 등록되어 있는 subdomain들에 대해서 그대로 사용이 가능해야하며, 안쓰는 모든 subdomain들에 대해서 어떠한 URL로 forward 시키는 방법에 대해 찾아보다 official page에 나와있는 내용이 있어 참고하였다.
* **사용해보니 uptime check 시 모든 traffic이 main domain을 향하고 있으니 실제로 subdomain의 service에 대해서 확인이 안된다는 단점이 있다.**  
  이것을 우회하려면 각각의 service에 대해서 직접적으로 port 또는 service 기반으로 확인 해야한다.

> ref.  
> - <https://doc.traefik.io/traefik/routing/routers/#host-and-hostregexp>

## Regexp로 forward 처리하기
---

제목에 regexp라고 적어놓은 것과 같이 Traefik에서는 regular express(정규 표현식)를 지원을 한다.  
여기서 regexp는 `*.domain.com`에서 `*`을 처리를 해야해서 필요하다.  
또한 여기서 `priority`부분이 나오는데 이것은 굉장히 중요하다.

`router`에는 priority(우선순위)가 존재하는데, **이 우선 순위는 숫자가 높을수록 순위가 높다.**

Traefik에서 file로 처리도 가능하지만, docker로 구성하면 traffic은 docker의 label 처리가 가장 깔끔하다.  
아래의 예시는 `docker-compose.yml`의 일부이며 실제로 사용하는 `labels`이다. 

```yaml
## docker-compose.yml

#--------------------- 생략 ---------------------#
    labels:
      traefik.enable: true
      traefik.docker.network: proxy

      # ...
      
      # host-redirect
      traefik.http.routers.host-redirect.rule: Host(`$HOST_URL`) || HostRegexp(`^.+\.$HOST_URL$`)
      traefik.http.routers.host-redirect.priority: 1
      traefik.http.routers.host-redirect.tls: true
      traefik.http.routers.host-redirect.tls.options: $TLS_OPTION
      traefik.http.routers.host-redirect.entrypoints: websecure
      traefik.http.routers.host-redirect.middlewares: host-redirect@docker
      traefik.http.middlewares.host-redirect.redirectregex.regex: ^.+
      traefik.http.middlewares.host-redirect.redirectregex.replacement: https://kuma.$HOST_URL
      traefik.http.middlewares.host-redirect.redirectregex.permanent: true
#--------------------- 생략 ---------------------#
```

해당 파일에서 ``HostRegexp(`^.+\.$HOST_URL$`)`` 부분은 중요하다.  
`router` 구조상 처음에 진입하게 되는 곳이 `rule` 부분이며, `domain.com`과 `*domain.com`가 가장 먼저 진입하게 되는 곳이다.  
그리고 그 아래에 `priority`부분이 있는데, 이 부분 또한 위에서 쓴 것과 같이 우선 순위가 낮으면 가장 마지막에 작동한다.  
즉, 개별적으로 `label` 또는 파일로 처리 된 모든 CNAME의 대해서 작동 후 마지막에 작동하는 것이다.  
이건 subdomain을 처리하는 모든 proxy server라도 똑같다.

`router`에 대해서 처리가 되었으면 `middleware`부분이 나오는데 위의 도메인을 그대로 받아들이는 `^.+`의 `redirectregex.regex`의 처리로 인해 `https://kuma.$HOST_URL`로 빠지게 되는 구조다.

Dashboard를 통해서도 확인이 가능하다.

![host-redirect@docker에 대한 dashboard](/assets/img/post/service/2024-08-29-traefik-forward_all_subdomain_with_regexp/1.png)
_host-redirect@docker에 대한 dashboard_