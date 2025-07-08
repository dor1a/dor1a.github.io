---
title: Postgres - user 생성과 권한 추가 및 변경 방법
date: 2023-09-19 10:09:00 +0900
categories: [db]
tags: [db, docker, postgres]
description: Postgres에서 처음 구성 시 user 생성과 권한 추가 및 변경하는 방법이다.
---

>Postgres 16
{: .prompt-info}

>Container
{: .prompt-warning}

>CLI
{: .prompt-tip}

# 개요
---

* `docker`로 `postgres`를 만든 뒤 `Superuser`가 필요하여 작성하게 되었다.
* `host`에서 package로 설치한 postgres 또한 동일하게 동작한다.

# user 생성과 권한 추가 및 변경 방법
---

우선 container내의 `psql`을 통하여 DB에 접근한다.  
접근 후 기존 user를 확인 후 필요 시 생성한다.

```shell
dor1@hq-is-lxc-rdb:~$ docker exec -it postgres psql -U postgres
psql (16.0 (Debian 16.0-1.pgdg120+1))
Type "help" for help.

postgres=# \

# 기존 user 확인
postgres=# \du
                             List of roles
 Role name |                         Attributes
-----------+------------------------------------------------------------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS

# user 생성
postgres=# CREATE USER deepadmin WITH PASSWORD 'qwer4321!';
CREATE ROLE

# user 확인
postgres=# \du
                             List of roles
 Role name |                         Attributes
-----------+------------------------------------------------------------
 deepadmin |
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS

# role 추가(변경 또한 가능)
postgres=# ALTER ROLE deepadmin SUPERUSER CREATEROLE CREATEDB REPLICATION BYPASSRLS;
ALTER ROLE

# 확인
postgres=# \du
                             List of roles
 Role name |                         Attributes
-----------+------------------------------------------------------------
 deepadmin | Superuser, Create role, Create DB, Replication, Bypass RLS
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS
```