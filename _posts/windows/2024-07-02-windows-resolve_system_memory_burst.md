---
title: Windows - System memory burst 대한 해결
categories: [Windows]
tags: [windows, memory, burst]
description: Windows 사용 중 비정상적이게 System memory를 사용하는 경우 해결하는 방법이다.
---

>Windows 11 23H2 (Build 22631.3810)
{: .prompt-info}

>Host
{: .prompt-warning}

>GUI
{: .prompt-tip}

# 개요
---

* Windows 사용 중 System memory가 95% 이상 차는 현상이 눈에 띄어 찾아보게 되었다.
* `poolmon.exe`을 통해 `pdjb`의 **Tag**를 찾아냈지만, 정확히 이 값이 무엇을 하는지는 확인이 안되어 `Non-paged pool`의 값을 제한하는 형식으로 진행했다.
* 또한 3rd-party 프로그램을 통해 process의 memory도 줄일 수 있는 방법을 찾게 되었다.

> **ref.**
> * <https://learn.microsoft.com/en-us/windows/win32/memory/memory-limits-for-windows-releases>
> * <https://ryuchan.kr/547>

# 원인과 발견
---

어떠한 이유인지 몰라도 Windows를 idle 상태로 하루정도 방치 했을 경우 `Task Manager` → `Process`에서 Memory의 사용율이 90% 이상 차는 현상이 발견 됐다.

![Memory title 또한 나타나질 않음](/assets/img/post/windows/2024-07-02-windows-resolve_system_memory_burst/1.png)
_Memory title 또한 나타나질 않음_

System memory는 분명 16GB로 아래의 process를 합쳐도 차고 넘치는데 이상하게 포화상태로 확인 된다.  
그래서 `Performance`에서 확인하기로 하였다.

![Non-paged pool이 비정상적으로 확인 됨](/assets/img/post/windows/2024-07-02-windows-resolve_system_memory_burst/2.png)
_Non-paged pool이 비정상적으로 확인 됨_

확인해보니 `Non-paged pool`쪽이 이상하게 6.4GB나 먹고 있는 상황으로 확인되었다.

# Non-paged pool의 메모리 누수 현상 확인
---

일반적으로 paged 된 것들은 System memory가 아닌 disk의 일부분에 정리하게 되는데, `Non-paged pool`은 page fault가 되지 않은 상태로 있기 때문에 System memory 위에서 그대로 작업이 유지 되고있는 형태다.  
`Non-paged pool`에서 사용하는 것들이 무엇이 있나 찾아보니, 다음과 같은 방법으로 확인 해볼 수 있긴하다.

## 1. Windows Driver Kit(WDK) 설치

다음의 페이지에서 WDK를 설치한다.

<https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk>

설치 과정은 그대로 따라하면 된다.

1. Visual Studio 2022 설치  
   설치 시 `search box`에 **64 latest spectre**를 적어 아래에 나오는 항목을 모두 체크하고 install 한다.
2. SDK 설치
3. WDK 설치

## 2. poolmon.exe를 통한 tag 확인

위의 WDK는 일반적으로 다음과 같은 경로에 설치가 된다.

`C:\Program Files (x86)\Windows Kits\10`

`poolmon.exe`은 Windows path로 환경설정이 바로 되어있지는 않기에 `cmd`를 통해 개별적으로 확인 해줄 수 있다.  
“winkey + r”을 통해 실행을 키고 `cmd`를 “shift + ctrl + enter”를 통해 관리자 권한으로 켜준다.  
켜준 뒤 `poolmon.exe`가 있는 경로로 들어가준 뒤 `poolmon.exe`를 실행한다.

```powershell
Microsoft Windows [Version 10.0.22631.3810]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\System32>cd C:\Program Files (x86)\Windows Kits\10\Tools\10.0.26100.0\x64

C:\Program Files (x86)\Windows Kits\10\Tools\10.0.26100.0\x64>poolmon.exe
```

그러면 파랑색 배경으로 된 창이 뜨는데 여기서 키보드의 `B`키 또는 `M`키를 눌러 **Bytes**, **Alloc** 항목에서 가장 많이 차지하고 잇는 항목을 찾아보면 된다.

![pdjb...? 넌 뭐냐...?](/assets/img/post/windows/2024-07-02-windows-resolve_system_memory_burst/3.png)
_pdjb...? 넌 뭐냐...?_

확인 해보니 **Tag**에 `pdjb`라는 놈이 찍혀 있고, **Bytes**를 가장 많이 사용하고 있는 것을 확인하였다.

## 3. 해결?

`poolmon.exe`의 **Tag** 중 잘 알려진 값은 다음과 같은 위치에 나와있다.

`C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\triage\pooltag.txt`

여기에서 나온 `pdjb`라는 값은 잘 알려진 값은 아닌지 나오지는 않았다.  
그래서 `drivers`에서 사용하는 것이 있나 확인 해봤다.  
확인 방법은 다음과 같이 진행할 수 있다.  
위와 똑같이 `cmd`에서 진행 해본다.

```powershell
Microsoft Windows [Version 10.0.22631.3810]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\System32>cd %SystemRoot%\System32\drivers

C:\Windows\System32\drivers>findstr /m /l pdjb *.sys

C:\Windows\System32\drivers>
```

return 값으로 따로 나오지는 않았다.  
제대로 찾아보려면 `Livekv`같은 프로그램을 통해 찾아봐야하지만, 굳이 `pdjb`의 **Tag**가 아니더라도 다른 **Tag**가 문제를 일으키면 그걸 또 하나하나 찾아 없애는 것 보다는 `Non-paged pool`의 size를 제한 두는것으로 생각하게 되었다.

# Non-paged pool의 size 제한
---

간단하게 registry로 제한이 가능하다.

“winkey + r”으로 실행을 킨 후 “regedit”를 입력하여 레지스트리 편집기를 들어간다.  
다음의 경로로 이동한다

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management`

해당 경로에서 `NonPagedPoolQuota`와 `NonPagedPoolSize` 항목을 바꿔 준다.  
1GB 정도가 적당하여 Decimal(10진수)로 byte로 환산한 값(1073741824)을 적어주면 된다.  
그리고 reboot을 시키면 제한 된 것을 확인 할 수 있다.

![Non-paged pool에서 제한이 걸려 1GB 이상 올라가지는 않음](/assets/img/post/windows/2024-07-02-windows-resolve_system_memory_burst/4.png)
_Non-paged pool에서 제한이 걸려 1GB 이상 올라가지는 않음_

# 3rd-party tool: Mem Reduct
---

일반 process에 대한 memory 가 비정상적일 때 사용하는 3rd-party tool이 있다.  
다음의 링크에서 받을 수 있다.

<https://memreduct.org/mem-reduct-download/>

설치 후 실행 해보면 간단한 창으로 구성 되어있는 것을 확인할 수 있다.

![Mem Reduct: Before clean](/assets/img/post/windows/2024-07-02-windows-resolve_system_memory_burst/5.png)
_Mem Reduct: Before clean_

창에서 `Clean memory`를 하면 많이 줄어드는 것을 확인 할 수 있다.

![Mem Reduct: After clean](/assets/img/post/windows/2024-07-02-windows-resolve_system_memory_burst/6.png)
_Mem Reduct: After clean_

보통의 프로그램은 사용자가 사용하고 있는 process를 종료하여서 memory 여유를 찾는데 해당 프로그램은 그런 것이 전혀 없다.  
아마도, 사용자가 사용하는 process가 아닌 **Windows**에서 사용하는 memory에 대해서 정리해 주는 것으로 보인다.  
여러가지 옵션을 통해 clean 할 수 있는 부분을 조절할 수 있다.  
일시적으로 줄어드는 현상이라하면 사용자가 clean 이후에도 무언가의 지속적인 작업이 이뤄질 가능성이 높다.  
schedule, start-up의 세팅도 가능하니 여러 방법으로 사용하면 좋을 듯 하다.