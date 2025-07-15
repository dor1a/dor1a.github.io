---
title: Windows - 네트워크(Network) Metric 조절하기
date: 2024-09-10 16:55:00 +0900
categories: [Windows]
tags: [windows, network]
description: Windows에서 Network의 Metric(우선순위)를 조절하는 방법이다.
---

>Windows 11 23H2 (Build 22631.4112)
{: .prompt-info}

>Host
{: .prompt-warning}

>GUI
{: .prompt-tip}

## 개요
---

* Wi-Fi와 Ethernet을 같이 사용하거나, 여러개의 Ethernet을 사용하게 된다 했을때 메인으로 traffic이 오가는 interface를 지정하기 위해서는 기본적으로 metric을 높혀야 한다.
* Linux에서는 conf 또는 yaml 작성으로 비교적 쉽게 변경이 가능하지만 Windows는 옵션을 숨겨놓아서 찾아야한다.

> ref.
> - <https://www.elevenforum.com/t/change-network-adapter-interface-connection-priority-order-in-windows-11.13548/>

## Metric 값에 대해 확인 하기
---

Powershell에서 한 줄의 명령어로 확인이 가능하다.(Powershell 특성 상 외우기가 까다로울 뿐...)

`Winkey + r`을 통해 실행(Run)에 진입 후 `powershell`을 입력 후 shift + ctrl + enter를 입력하여 관리자(Administrator)권한으로 진입한다.

이후 다음과 같이 입력하면 확인이 가능하다.

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> Get-NetIPInterface | Format-Table -AutoSize

ifIndex InterfaceAlias              AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionState PolicyStore
------- --------------              ------------- ------------ --------------- ----     --------------- -----------
18      로컬 영역 연결* 10          IPv6                  1500              25 Disabled Disconnected    ActiveStore
15      로컬 영역 연결* 9           IPv6                  1500              25 Disabled Disconnected    ActiveStore
1       Loopback Pseudo-Interface 1 IPv6            4294967295              75 Disabled Connected       ActiveStore
19      이더넷 5                    IPv4                  1500              25 Disabled Connected       ActiveStore
9       이더넷 2                    IPv4                  1500              25 Enabled  Disconnected    ActiveStore
6       Bluetooth 네트워크 연결     IPv4                  1500              65 Enabled  Disconnected    ActiveStore
18      로컬 영역 연결* 10          IPv4                  1500              25 Disabled Disconnected    ActiveStore
15      로컬 영역 연결* 9           IPv4                  1500              25 Enabled  Disconnected    ActiveStore
10      Wi-Fi                       IPv4                  1500              50 Enabled  Disconnected    ActiveStore
1       Loopback Pseudo-Interface 1 IPv4            4294967295              75 Disabled Connected       ActiveStore


PS C:\Windows\system32>
```

여기서 `InterfaceMetric`부분을 확인 하면 된다.  
기본적으로 **Wi-Fi는 50, Ethernet은 25**의 metric을 갖고 있다.

## Metric 변경
---

다시 `Winkey + r`을 통해 실행(Run)을 진입 후 이번엔 `ncpa.cpl`이이라고 입력 후 진입한다.  
`ncpa.cpl`은 과거 `네트워크 연결(Network Connections)`과 같은 곳이다.  
이 곳에서 바꾸고 싶은 Ethernet 또는 Wi-Fi을 선택 후 `속성(Properties)`에 진입 한다.  
이후 `인터넷 프로토콜 버전 4(TCP/IPv4)`를 선택 후 `속성(Properties)`에 진입 한다.

![인터넷 프로토콜 버전4(TCP/IPv4) -> 속성(Properties)](/assets/img/post/windows/2024-09-10-windows-adjust_network_metric/1.png)
_인터넷 프로토콜 버전4(TCP/IPv4) -> 속성(Properties)_

진입하면 한번 더 창이 뜨는데 이 곳은 고정(static) IP를 입력하는 부분이다.  
여기서도 `Advanced…`으로 진입한다.

![Advanced...으로 이동](assets/img/post/windows/2024-09-10-windows-adjust_network_metric/2.png)
_Advanced...으로 이동_

한번 더 창이 뜨면 자동 매트릭(Automatic metric)의 체크를 해제 후 metric을 입력하면 된다.  
Metric은 높을수록 우선 순위가 높다.

![Metric 입력](assets/img/post/windows/2024-09-10-windows-adjust_network_metric/3.png)
_Metric 입력_

이후 Powershell에서 다시 확인해보면 metric이 높아진 것을 확인 할 수 있다.

```powershell
PS C:\Windows\system32> Get-NetIPInterface | Format-Table -AutoSize

ifIndex InterfaceAlias              AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionState PolicyStore
------- --------------              ------------- ------------ --------------- ----     --------------- -----------
18      로컬 영역 연결* 10          IPv6                  1500              25 Disabled Disconnected    ActiveStore
15      로컬 영역 연결* 9           IPv6                  1500              25 Disabled Disconnected    ActiveStore
1       Loopback Pseudo-Interface 1 IPv6            4294967295              75 Disabled Connected       ActiveStore
# 이더넷 5에 대해 InterfaceMetric이 25에서 60으로 변경 됨
19      이더넷 5                    IPv4                  1500              60 Disabled Connected       ActiveStore
9       이더넷 2                    IPv4                  1500              25 Enabled  Disconnected    ActiveStore
6       Bluetooth 네트워크 연결     IPv4                  1500              65 Enabled  Disconnected    ActiveStore
18      로컬 영역 연결* 10          IPv4                  1500              25 Disabled Disconnected    ActiveStore
15      로컬 영역 연결* 9           IPv4                  1500              25 Enabled  Disconnected    ActiveStore
10      Wi-Fi                       IPv4                  1500              30 Enabled  Connected       ActiveStore
1       Loopback Pseudo-Interface 1 IPv4            4294967295              75 Disabled Connected       ActiveStore


PS C:\Windows\system32>
```