---
title: Linux - Bash 변수 정리
date: 2022-07-22 11:07:00 +0900
categories: [Linux]
tags: [linux, bash]
description: Linux에서 가장 많이 사용하는 shell인 bash에서 사용하는 변수에 대해 정리해 보았다.
---

>대부분의 Linux 배포판
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Bash 사용 시 필요한 변수에 대해 정리하였다.
* 문서는 필요 시 지속적으로 업데이트한다.

## 변수(Variable)
---

>bash 변수는 `/etc/profile.d/` 이하에 sh 파일을 생성 (export 처리)하여 넣으면 모든 사용자 ID에 적용이 가능하다.  
또한 `~/.bashrc`에 입력하면 각각의 사용자 ID에 적용이 가능하다.
{: .prompt-info}

대부분의 변수는 `man bash`에서 확인이 가능하다.  
기존의 변수에 추가 항목을 넣을 때는 해당 변수를 그대로 입력 해주면 된다.

```shell
# 추가 항목 예제
VAR2=$VAR1:/home/ubuntu
```

### 1. PATH

```shell
# bin 파일 또는 python(py) 파일 등의 위치
PATH=$PATH:/usr/local/cuda/bin
```

## 2. HISTTIME

```shell
# History 시간 설정
HISTTIMEFORMAT="[%F %T] "

# Stack 크기 설정
HISTSIZE=1000

# History 파일 위치 지정
HISTFILE=~/.bash_history

# History 파일 크기 지정
HISTFILESIZE=1000
```