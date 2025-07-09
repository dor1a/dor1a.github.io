---
title: Postgres - DB dump 및 restore
date: 2023-09-19 10:10:00 +0900
categories: [DB]
tags: [db, postgres]
description: Postgres에서 모든 DB에 대해 dump와 restore 하는 방법을 간단히 작성해 보았다.
---

>Postgres 15.4
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `MariaDB`와 거의 같다고 봐도 된다.

# 백업 및 복구
---

### 1. 백업(dump)

* DB 전체 백업

```shell
pg_dumpall -U [username] > ./dumpfile.sql
```

### 2. 복구(restore)

```shell
psql -U {postgres_user} < ./dumpfile.sql
```