---
title: Mikrotik - Log 설정
date: 2022-07-22 11:33:00 +0900
categories: [Network, HW]
tags: [mikrotik]
description: Mikrotik 장비에서 Log 설정을 해본다.
---

>CCR1016-12G (tile) v7.1.2
{: .prompt-info}

>Host
{: .prompt-warning}

>GUI
{: .prompt-tip}

# 개요
---

* Mikrotik은 기본적으로 System memory에 logging을 하게 된다.
* Memory logging을 하다 보니 "Log" 탭에서 보여지는 내용이 한정적이라서 지난 내역에 대해 확인이 어려웠으며, Log들을 USB Memory에 저장하여 필요할 때 받아서 볼 수 있으면 좋을 것 같다고 생각했다.

# System logging
---

다음과 같은 Step을 따라하여 진행 해주면 된다.

## 1. Disk 설정

Mikrotik 내장 디스크는 용량이 많지 않으므로 Logging 데이터를 오래 쌓기 위해서는 USB Memory 사용을 추천 함

## 2. Winbox 설치 및 켜기

><https://mikrotik.com/download>

Winbox를 다운로드 받고 실행 시켜주면 된다.

## 3. Retention policy 및 Log 저장 위치 설정

1. Winbox 에서 "System -> Logging" 메뉴 선택
2. Actions 탭 클릭
3. 다음과 같이 설정  
   `Name: usb`  
   `Type: disk`  
   `File Name: /disk1/log`  
   `Line Per File: 1000`  
   `File Count: 3`
4. OK 버튼으로 생성

![Log Action](/assets/img/post/network/2022-07-22-mikrotik-setup-log/1.png)
_Log Action_

## 4. Persistent log rules (로컬디스크 저장 룰 설정)

1. Rules 탭에서 "+" 버튼을 통하여 새로 생성
2. Topics는 **info** / Actions은 **usb**
3. OK 버튼으로 생성
4. Topics에 대해 여러개 생성을 위하여 "critical", "error", "warning"에 대해서도 위 작업 반복
5. 기본 설정되어 있던 Rules 상단 "X" 버튼을 통해 사용 해제

![Log Rule](/assets/img/post/network/2022-07-22-mikrotik-setup-log/2.png)
_Log Rule_

## 5. Log 확인

- 메뉴에서 Log 선택
- "Buffer" 탭에서 usb에 저장 되는지 확인

![Log](/assets/img/post/network/2022-07-22-mikrotik-setup-log/3.png)
_Log_

- 메뉴에서 Files 선택
- 정상적으로 저장되는지 확인

![File List](/assets/img/post/network/2022-07-22-mikrotik-setup-log/4.png)
_File List_