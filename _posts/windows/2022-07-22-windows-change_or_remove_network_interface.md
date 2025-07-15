---
title: Windows - 네트워크 인터페이스(Netowrk interface)의 이름 변경 또는 삭제 방법
date: 2022-07-22 14:30:00 +0900
categories: [Windows]
tags: [windows, network]
description: Windows의 ncpa.cpl 상의 네트워크 인터페이스의 이름 변경 또는 삭제하는 방법이다.
---

>Windows 11 21H2 (Build 22000.588)
{: .prompt-info}

>Host
{: .prompt-warning}

>GUI
{: .prompt-tip}

## 개요
---

* 네트워크 연결에서 네트워크 이름(SSID)를 변경하기 위해 작성 되었다.

![Default Ethernet](/assets/img/post/windows/2022-07-22-windows-change_or_remove_network_interface/1.png)
_Default Ethernet_

## Service & Registry
---

두 가지의 방법 중 하나로 진행한다.

### 1. secpol.msc

* Local Security Policy(로컬 보안 정책) -> Network List Manager Poclicies(네트워크 목록 관리자 정책) -> 자신의 네트워크 이름 -> Name 수정 후 Apply(적용) 또는 OK(확인)

![secpol.msc](/assets/img/post/windows/2022-07-22-windows-change_or_remove_network_interface/2.png)
_secpol.msc: Interface properties_

### 2. regedit

- 실행 창(Winkey + r)을 켜서 regedit를 입력하여 레지스트리 편집기로 가준다.
- 레지스트리 편집기에서 다음과 같은 경로로 이동한다.

```powershell
# Profile
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles

# Signatures
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures
```

- `Profile`에서 하위 UUID 값 이루어진 폴더 클릭한다.
  `Description` 또는 `ProfileName`의 항목들이 나오는데 확인 후 수정해주거나 지우면 된다.
- `Signatures`에서는 따로 Static으로 관리해주지 않는 이상 `Unmanaged` 하위에 16진수로 이루어진 폴더가 있다.  
  여기서 16진수 폴더 항목들을 클릭 한 뒤 `Description` 또는 `FirstNetwork`를 확인 후 수정 해주거나 지우면 된다.

![HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles](/assets/img/post/windows/2022-07-22-windows-change_or_remove_network_interface/3.png)
_HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles_

![HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures](/assets/img/post/windows/2022-07-22-windows-change_or_remove_network_interface/4.png)
_"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures_