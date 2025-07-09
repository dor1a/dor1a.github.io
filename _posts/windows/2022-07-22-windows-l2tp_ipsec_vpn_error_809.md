---
title: Windows - L2TP/IPsec VPN Error (code 809)
date: 2022-07-22 14:24:00 +0900
categories: [Windows]
tags: [windows, vpn]
description: Windows L2TP/IPsec VPN에서 발생 하는 809 error에 대해 해결 방법이다.
---

>Windows 11 21H2 (Build 22000.588)
{: .prompt-info}

>Host
{: .prompt-warning}

>GUI
{: .prompt-tip}

## 개요
---

* Windows에서 L2TP/IPsec VPN 연결 시 809 error가 발생해서 확인 해 봤다.

> ref.
> - <https://docs.microsoft.com/ko-KR/troubleshoot/windows-server/networking/configure-l2tp-ipsec-server-behind-nat-t-device>

## 해결 방법
---

>원격 서버가 응답하지 않기 때문에 컴퓨터에서 VPN 서버로 네트워크 연결을 할 수 없습니다. 컴퓨터와 원격 서버 사이에 있는 네트워크 장치(예: 방화벽, NAT, 라우터 등) 중 하나가 VPN 연결을 허용하도록 구성되어 있지 않기 때문일 수 있습니다. 관리자나 서비스 공급자에게 문의하여 일으키는 장치를 확인하십시오.
{: .prompt-danger}

![VPN에서 error 발생](/assets/img/post/windows/2022-07-22-windows-l2tp_ipsec_vpn_error_809/1.png)
_VPN에서 error 발생_

- 우선 실행 창(Winkey + r)을 켜서 regedit를 입력하여 레지스트리 편집기로 가준다.
- 레지스트리 편집기에서 다음과 같은 주소로 이동한다.
  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent`
- 편집 -> 새로만들기 -> DWORD(32비트)로 `AssumeUDPEncapsulationContextOnSendRule`라는 이름의 새로운 항목을 만들어준다.
- 값에는 2(숫자 2는 16진수, 10진수 둘다 동일한 값)를 넣고 저장한다.
- System rebooting을 해준다.

![HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent](/assets/img/post/windows/2022-07-22-windows-l2tp_ipsec_vpn_error_809/2.png)
_HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent_

추가적으로 다음은 `AssumeUDPEncapsulationContextOnSendRule`의 항목에 대한 내용이다.

* 0 : NAT 장치 뒤에 있는 서버와 보안 연결을 설정할 수 있도록 Windows를 구성 (기본값)
* 1 : NAT 장치 뒤에 있는 서버와 보안 연결을 설정할 수 있도록 Windows를 구성
* 2 : 서버와 VPN 클라이언트 컴퓨터가 Windows Vista 이상 또는 Windows Server 2008 기반 NAT 장치 뒤에 있을 때 보안 연결을 설정할 수 있도록 Windows를 구성