---
title: Ubuntu - docker-compose에서 $HOST 변수 사용하기
date: 2023-10-04 11:57:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, docker-compose]
description: Ubuntu에서 docker-compose에서 $HOST 변수를 사용하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Debian 계열만 그런지는 모르겠지만 `docker-compose`에서 `hostname`을 의미하는 변수인 `$HOST`의 사용이 불가능 하다.
* 공식적이지는 않지만 `set`에 들어가 있는  변수를 `env`로 끌고 나오면 가능할 것 같아 진행하게 되었다.  

## export를 이용한 쉬운 처리
---

아주 간단하게 `export`의 command로 처리가 가능할 것으로 보인다.  
아래는 `watchtower`를 갖고 진행한 예시다.  
우선 yaml 파일을 확인 했을 때에는 `$HOST`가 정상으로 입력되어 있다.

```yaml
dor1@tera /docker/compose ❯ cat docker-compose.yml
version: '3'
services:
  watchtower:
    image: docker.io/containrrr/watchtower:latest
    container_name: watchtower
    hostname: watchtower
    restart: unless-stopped
    network_mode: "bridge"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    environment:
      TZ: Asia/Seoul
      WATCHTOWER_SCHEDULE: 0 0 0 * * *
      WATCHTOWER_CLEANUP: 1
      # 해당 부분
      WATCHTOWER_NOTIFICATIONS_HOSTNAME: $HOST
```

다음은 실제로 config의 구성을 확인 해본다.

```yaml
dor1@tera /docker/compose ❯ docker-compose config watchtower
# 변수 인식 불가
WARN[0000] The "HOST" variable is not set. Defaulting to a blank string.
name: compose
services:
  watchtower:
    container_name: watchtower
    environment:
      TZ: Asia/Seoul
      WATCHTOWER_CLEANUP: "1"
      # HOST 비어 있음
      WATCHTOWER_NOTIFICATIONS_HOSTNAME: ""
      WATCHTOWER_SCHEDULE: 0 0 0 * * *
    hostname: watchtower
    image: docker.io/containrrr/watchtower:latest
    network_mode: bridge
    restart: unless-stopped
    volumes:
    - type: bind
      source: /var/run/docker.sock
      target: /var/run/docker.sock
      read_only: true
      bind:
        create_host_path: true
```

이를 해결하기 위해서는 `export`를 이용하여서 `env`로 가져온다.  
`set`에 등록 되어있는 parameter를 가져오는 방식이다.

```shell
# HOST 없음
dor1@tera /docker/compose ❯ env | grep -i host
DISPLAY=localhost:10.0

# export 후 확인
# zsh
dor1@tera /docker/compose ❯ export HOST
dor1@tera /docker/compose ❯ env | grep -i host
DISPLAY=localhost:10.0
HOST=tera

# bash
dor1@tera /docker/compose ❯ export HOSTNAME
dor1@tera /docker/compose ❯ env | grep -i hostname
DISPLAY=localhost:10.0
HOSTNAME=tera
```

가져온 뒤 다시 config 진행 해본다.

```yaml
# warning 없음
dor1@tera /docker/compose ❯ docker-compose config watchtower
name: compose
services:
  watchtower:
    container_name: watchtower
    environment:
      TZ: Asia/Seoul
      WATCHTOWER_CLEANUP: "1"
      # 등록 확인
      WATCHTOWER_NOTIFICATIONS_HOSTNAME: tera
      WATCHTOWER_SCHEDULE: 0 0 0 * * *
    hostname: watchtower
    image: docker.io/containrrr/watchtower:latest
    network_mode: bridge
    restart: unless-stopped
    volumes:
    - type: bind
      source: /var/run/docker.sock
      target: /var/run/docker.sock
      read_only: true
      bind:
        create_host_path: true
```

이걸 persist하게 사용하고 싶다면 쉽게 `$HOME/.bashrc|.zshrc`에 등록 해서 사용하면 된다.