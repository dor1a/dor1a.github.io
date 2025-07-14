---
title: Linux - sudo 권한에 대한 간단 정리
date: 2022-08-11 09:35:00 +0900
categories: [Linux]
tags: [linux, sudo]
description: Linux에서 root 권한에 대해 사용할 수 있는 sudo 권한에 대해 간단하게 정리해 보았다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `sudo -i`,  `sudo su`, `su -` 차이점에 대하여 정리한다.

## 차이점
---

1. sudo -i 는 ENV를 root 유저의 것으로 가져옴
2. su와 su root는 root 계정으로 로그인 함 (ENV 초기화를 안함)
3. su -는 root 계정으로 로그인 후 ENV는 root 유저의 것으로 가져옴
4. sudo su는 user가 sudo 권한을 가져오며 root가 su 했을때와 같이 동작함