---
title: MariaDB - Dump 및 restore
date: 2022-07-22 13:56:00 +0900
categories: [db]
tags: [db, mariadb]
description: MariaDB에서 모든 DB에 대해 dump와 restore 하는 방법을 간단히 작성해 보았다.
---

>MariaDB 15.1
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

# 개요
---

* Web Service 도중 DB 백업의 중요성을 요구하게 되었다.
* DB 백업 파일은 `.sql` 파일 확장자를 갖고 있으며, 해당 파일은 에디터로 수정 또한 가능하다.

# 백업 및 복구
---

## 1. 백업(Dump)

* DB 전체 백업

```shell
mysqldump -u [user_name] -p[password] [db_name] > file_name.sql
```

* DB Table 백업

```shell
mysqldump -u [user_name] -p[password] [db_name] [table_name] > file_name.sql
```

비밀번호에 !@ 등과 같이 특수 문자가 들어가게 되면 `<password>` 부분을 빼고 입력해도 된다.

## 2. 복구(Restore)

```shell
mysql -u [user_name] -p[password] [db_name] < file_name.sql
```