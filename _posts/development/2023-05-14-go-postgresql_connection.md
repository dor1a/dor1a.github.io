---
title: Go - Postgresql connection
date: 2023-05-14 17:29:00 +0900
categories: [Development]
tags: [go]
description: Go를 이용해서 Postgres DB 접속을 테스트 해 본다.
---

>go v1.20.2
{: .prompt-info}

>Linux OS
{: .prompt-tip}

## 개요
---

* Postgresql 연동을 위한 sample 코드로 진행

> ref.
> * <https://github.com/lib/pq>

## Connection to PostgreSQL
---

`pq` library를 받아준다.

```shell
# Linux
root@ubuntu:~$ go get -u github.com/lib/pq
```

libarary import 후 진행

```go
// postgres_connection.go

// main
package main

// import library pacakge
import (
  "database/sql"
  "fmt"
  _ "github.com/lib/pq"
)

// database/sql애서 사용되는 변수
const (
  host     = "postgresql-ha-pgpool.postgresql-ha.svc.cluster.local"
  port     = 5432
  user     = "postgres"
  password = "wJ4P6g%uQm"
  dbname   = "postgres"
)

func main() {
  // sslmode 기본 diable
  psqlInfo := fmt.Sprintf("host=%s port=%d user=%s "+
    "password=%s dbname=%s sslmode=disable",
    host, port, user, password, dbname)
  db, err := sql.Open("postgres", psqlInfo)
  if err != nil {
    panic(err)
  }
  defer db.Close()

  // error 처리
  err = db.Ping()
  if err != nil {
    panic(err)
  }

  fmt.Println("Successfully connected!")
}
```

실행

```shell
root@ubuntu:~$ go run ./postgrtes_connection.go
Successfully connected!
```