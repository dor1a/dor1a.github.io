---
title: Windows - L2TP/IPsec VPN Error (code 787)
date: 2022-07-22 14:19:00 +0900
categories: [Windows]
tags: [windows, vpn]
description: Windows L2TP/IPsec VPN에서 발생 하는 787 error에 대해 해결 방법이다.
---

>Windows 11 21H2 (Build 22000.588)
{: .prompt-info}

>Host
{: .prompt-warning}

>GUI
{: .prompt-tip}

## 개요
---

* Windows에서 L2TP/IPsec VPN 연결 시 787 Error가 출력이 되어 확인하게 되었다.

> ref.
> - <https://docs.microsoft.com/ko-kr/troubleshoot/windows-server/networking/l2tp-vpn-fails-with-error-787>

## 해결 방법
---

>보안 계층에서 원격 컴퓨터를 인증할 수 아니기 때문에 L2TP 연결 시도가 실패했습니다.
{: .prompt-danger}

- 우선 실행 창(Winkey + r)을 켜서 regedit를 입력하여 레지스트리 편집기로 가준다.
- 레지스트리 편집기에서 다음과 같은 주소로 이동한다.
  `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters`
- 편집 -> 새로만들기 -> DWORD(32비트)로 `ProhibitIpSec`라는 이름의 새로운 항목을 만들어준다.
- 값에는 1(숫자 1는 16진수, 10진수 둘다 동일한 값)을 넣고 저장한다.
- System rebooting을 해준다.

 ![HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters](/assets/img/post/windows/2022-07-22-windows-l2tp_ipsec_vpn_error_787/1.png)
 _HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters_