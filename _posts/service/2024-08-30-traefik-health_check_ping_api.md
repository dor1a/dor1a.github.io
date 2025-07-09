---
title: Traefik - ping API로 health check 하기
date: 2024-08-30 10:16:00 +0900
categories: [Service]
tags: [service, traefik, docker]
description: Docker에서 동작 중인 Traefik에서 ping API로 이용해 health check를 해 본다.
---


>traefik:v3.1.2
{: .prompt-info}

>Container
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `operations` 메뉴 중 ping API를 활성화 시켜서 health check가 가능하게 끔 하는 메뉴가 있다.
* 이것을 통해서 container의 health check 및 Traefik에 대해서도 health check가 가능하다.

> ref.
> - <https://doc.traefik.io/traefik/operations/ping/>

## ping API 활성화 하기
---

개인적으로는 처음 들어오는 network traffic에 대해 처리는 `docker-compose.yml` 파일의 `labels`로 처리하는 방법이 좋다고 생각하고,  
Traefik의 기능들은 따로 파일로 관리하는 편이 좋다고 생각한다.

우선 `docker-compose.yml`에서 Traefik의 기능을 관리하는 파일을 volume mapping을 시켜준 뒤 container health check를 위해 `healthcheck` 부분도 추가해준다.

아래의 예시인 `docker-compose.yml` 파일은 실제로 사용하는 중이다.

```yaml
# docker-compose.yml
  traefik:
    image: docker.io/library/traefik:latest
    
    #--------------------- 생략 ---------------------#    

    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      # /etc/treafik/treafik.yml이 실제로 traefik의 기능에 대해서 관리하는 파일이며 /etc/treafik 이하에 파일명을 다르게 maaping 시에도 동일하게 작동 함
      - "$PROXY_BACKEND_DATA_PATH/traefik/config/traefik.yml:/etc/traefik/traefik.yml:ro"

    #--------------------- 생략 ---------------------#
    
    # 해당 healthcheck 부분을 추가 해줘야 container에 대한 health check가 가능 함
    healthcheck:
      test: ["CMD", "wget", "http://localhost:8082/ping", "--spider"]
      interval: 10s
      timeout: 2s
      retries: 3
      start_period: 5s
```

그 다음 예시는 Traefik의 기능을 관리하는 파일인 `traefik.yml`이며 위와 동일하게 실제로 사용하는 파일이다.

```yaml
# traefik.yml

################################################################
# Global configuration
################################################################

global:
  checkNewVersion: true
  sendAnonymousUsage: true

################################################################
# EntryPoints configuration
################################################################

entryPoints:
  web:
    address: :80
  websecure:
    address: :443
    http3:
      advertisedPort: 443
  traefik:
    address: :8080
  ping:
    address: :8082

################################################################
# Traefik logs configuration
################################################################

log:
  level: DEBUG
  filepath: "/etc/traefik/log/traefik.log"

################################################################
# Access logs configuration
################################################################

accessLog:
  filePath: "/etc/traefik/log/access.log"
  bufferingSize: 100
  filters:
    statusCodes: ["400-499"]

################################################################
# API and dashboard configuration
################################################################

api:
  insecure: false
  dashboard: true
  debug: true

################################################################
# Ping configuration
################################################################

# envtryPoint를 추가해줘야 작동한다.
ping:
  entryPoint: "ping"
      
#--------------------- 생략 ---------------------#
```

추가 후 container의 health check를 확인 해 본다.

```shell
# docker ps에서 STATUS 부분에 healthy를 확인 할 수 있음
dor1@Nukumori ~ ❯ docker ps -a
CONTAINER ID   IMAGE                                    COMMAND                  CREATED        STATUS                  PORTS                                                                                                                                                            NAMES
b043d86daaa4   traefik:latest                           "/entrypoint.sh trae…"   18 hours ago   Up 18 hours (healthy)   0.0.0.0:8080->80/tcp, :::8080->80/tcp, 0.0.0.0:4443->443/tcp, 0.0.0.0:4443->443/udp, :::4443->443/tcp, :::4443->443/udp, 0.0.0.0:81->8080/tcp, :::81->8080/tcp   traefik
```

Dashboard를 통해서도 확인이 가능하다.

 ![:8082 port를 통해 ping 확인이 가능](/assets/img/post/service/2024-08-30-traefik-health_check_ping_api/1.png)
 _:8082 port를 통해 ping 확인이 가능_