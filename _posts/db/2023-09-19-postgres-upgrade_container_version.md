---
title: Postgres - Container 버전 업그레이드
date: 2023-09-19 10:09:00 +0900
categories: [db]
tags: [db, docker, postgres]
description: Docker Postgres의 오래된 container에 대해 버전 업그레이드 하는 방법이다.
---

>Postgres 15.4
{: .prompt-info}

>Container
{: .prompt-warning}

>CLI
{: .prompt-tip}

# 개요
---

* Docker를 사용하다 다음과 같은 error가 발생하면서 작동을 하지 않는다.

![docker logs](/assets/img/post/db/2023-09-19-postgres-upgrade_container_version/1.png)
_docker logs_

* Error의 내용을 보면 알다시피 메이저 버전이 올라가게 되면 이전의 DB와 호환이 되지 않아서 발생한다.
* 이럴 때 DB를 `pg_dumpall` 통하여 Dump 후 다시 복구하는 과정을 거치면서 진행하면 된다.

# 버전 업그레이드
---

사실 `container`가 아닌 `host`에서는 `pg_upgrade`라는 명령어를 통해서 진행하면 간단하지만, 이것은 OS에 현재 버전과 업그레이드 할 버전이 동시에 설치가 되어있을 때 가능한 명령어기 때문에 `docker`에서는 사용이 불가능하다.  
다만 이걸 또 `docker`에서 사용 가능하게 만들어 놓은 사람이 있다.  
<https://github.com/tianon/docker-postgres-upgrade>

그러나 굳이, 저걸 사용하면서 할 필요는 없기 때문에 나름의 정석(?) 방법을 통해 진행하는 것이 깔끔하다고 생각한다.

## 1. DB Dump

사실 DB Dump는 기본적으로 사용되는 명령어에 대한 글([Postgres - Dump 및 restore](/posts/postgres-dump_and_restore/))이 있긴 하다.  
물론 저 글에 있는 명령어가 기본이기 때문에 사용하여 진행하지만,`container` 환경이기 때문에 명령어의 `which`가 좀 다르긴 하다.

```shell
# container 확인
dor1@Nukumori > docker ps -a | grep -i postgres
a46da0b1794f   postgres:15                              "docker-entrypoint.s…"   23 minutes ago   Up 23 minutes (healthy)   0.0.0.0:54321->5432/tcp, :::54321->5432/tcp                                                                                                                      postgres

# Host에서 dump
dor1@Nukumori ~ ❯ docker exec -it postgres /usr/bin/pg_dumpall -U dor1 > ./dump.sql
dor1@Nukumori ~ ❯ ll | grep -i dump
-rwxrwxrwx+ 1 dor1 users          12913925 Sep 19 22:27 dump.sql
```

## 2. Upgrade 될 container에서 DB restore

간단하게 새로운 버전에 대해서 데이터 directory를 생성해 준 뒤, restore 절차를 통해 진행한다.

```shell
# directory 생성
dor1@Nukumori ~ ❯ sudo mkdir -p /volume1/docker/data/proxy-backend/postgres_16/data

# docker-compose.yml 작성 후 up
dor1@Nukumori ~ ❯ cd /volume1/docker/compose/proxy-backend
dor1@Nukumori:/volume1/docker/compose/proxy-backend ❯ sudo vi docker-compose.yml
....
#----------------------------BackEnd----------------------------#
  postgres_16:
    image: docker.io/library/postgres:latest
    container_name: postgres_16
    hostname: postgres_16
    restart: unless-stopped
    networks:
      - backend
    volumes:
      - "$PROXY_DATA_PATH/postgres_16/data:/var/lib/postgresql/data:rw"
    environment:
      - "TZ=Asia/Seoul"
      - "POSTGRES_PASSWORD=qwer4321!"
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 20s
....
dor1@Nukumori:/volume1/docker/compose/proxy-backend ❯ docker-compose up -d postgres_16
[+] Running 1/1
 ⠿ Container postgres_16  Started

# 확인
dor1@Nukumori:/volume1/docker/compose/proxy-backend ❯ docker ps -a | grep -i postgres
cab211956adf   postgres:latest                          "docker-entrypoint.s…"   42 seconds ago   Up 40 seconds (healthy)   5432/tcp                                                                                                                                                         postgres_16
a46da0b1794f   postgres:15                              "docker-entrypoint.s…"   40 minutes ago   Up 40 minutes (healthy)   0.0.0.0:54321->5432/tcp, :::54321->5432/tcp

# Host에서 신버전에 restore
dor1@Nukumori ~ ❯ docker exec -i postgres_16 psql -U postgres < ./dump.sql
```