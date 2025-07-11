---
title: Linux(Ubuntu) - Let's Encrypt의 *(Wildcard) 인증서를 certbot을 통해 발급 받기
date: 2024-08-08 11:22:00 +0900
categories: [Linux]
tags: [linux, ubuntu, certbot]
description: Ubuntu에서 도메인에 대해 무료 인증서인 Let's Encrypt사의 *(Wildcard) 인증서를 certbot으로 발급 받는 방법이다.
---

>Ubuntu 24.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Let’s Encrypt 사는 무료로 공인 된 인증서를 발급해준다.
* \*(Wildcard) 인증서는 CNAME에 대해서 모두 사용이 가능하기 때문에 받을 수 있으면 가장 효율적으로 사용이 가능하다.
* 솔루선으로 봤을 때 `NPM(Nginx Proxy Manager)`은 자동 갱신을 해주긴 하지만 wildcard(\*) 인증서가 발급이 안되며, `traefik`은 발급과 자동 갱신(**Cloudflare**와 같이 API 지원이 되는 곳만)이 가능하다.  
  다만, `traefik`은 익숙하지 않은 사람이 다루기에는 까다로운 점이 있다.
* `certbot`은 local 또는 container를 통해 파일 형태로 발급이 가능하다.
* `certbot` 또한 `traefik`와 같이 자동 갱신을 하려면 **Cloudflare**와 같이 API 지원이 되는 곳만 된다.

> ref.
> - <https://velog.io/@suhongkim98/Lets-Encrypted-certbot%EC%9C%BC%EB%A1%9C-%EC%99%80%EC%9D%BC%EB%93%9C%EC%B9%B4%EB%93%9C-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B0%9C%EA%B8%89%EB%B0%9B%EA%B8%B0>

## 무료 도메인으로 인증서 발급 받기 (with. 내도메인.한국)
---

>자동 갱신을 하려면 Cloudflare와 같이 DNS challenge가 지원되는 곳을 사용해야 함
{: .prompt-danger}

### 1. certbot 설치

거의 모든 ubuntu repository에 있기 때문에 간단히 한줄로 설치가 가능하다.

```shell
admin@lxc-haproxy:~$ sudo apt install certbot
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-acme python3-certbot python3-certifi python3-chardet python3-configargparse
  python3-configobj python3-icu python3-idna python3-josepy python3-openssl python3-parsedatetime
  python3-requests python3-rfc3339 python3-tz python3-urllib3
Suggested packages:
  python-certbot-doc python3-certbot-apache python3-certbot-nginx python-acme-doc
  python-configobj-doc python-openssl-doc python3-openssl-dbg python3-socks python-requests-doc
  python3-brotli
The following NEW packages will be installed:
  certbot python3-acme python3-certbot python3-certifi python3-chardet python3-configargparse
  python3-configobj python3-icu python3-idna python3-josepy python3-openssl python3-parsedatetime
  python3-requests python3-rfc3339 python3-tz python3-urllib3
0 upgraded, 16 newly installed, 0 to remove and 0 not upgraded.
Need to get 1638 kB of archives.
After this operation, 8397 kB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://mirror.kakao.com/ubuntu noble/main amd64 python3-openssl all 23.2.0-1 [47.8 kB]
Get:2 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-josepy all 1.14.0-1 [22.1 kB]
Get:3 http://mirror.kakao.com/ubuntu noble/main amd64 python3-certifi all 2023.11.17-1 [165 kB]
Get:4 http://mirror.kakao.com/ubuntu noble/main amd64 python3-chardet all 5.2.0+dfsg-1 [117 kB]
Get:5 http://mirror.kakao.com/ubuntu noble-security/main amd64 python3-idna all 3.6-2ubuntu0.1 [49.0 kB]
Get:6 http://mirror.kakao.com/ubuntu noble/main amd64 python3-urllib3 all 2.0.7-1 [113 kB]
Get:7 http://mirror.kakao.com/ubuntu noble/main amd64 python3-requests all 2.31.0+dfsg-1ubuntu1 [50.7 kB]
Get:8 http://mirror.kakao.com/ubuntu noble/main amd64 python3-tz all 2024.1-2 [31.4 kB]
Get:9 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-rfc3339 all 1.1-4 [6744 B]
Get:10 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-acme all 2.9.0-1 [48.5 kB]
Get:11 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-configargparse all 1.7-1 [31.7 kB]
Get:12 http://mirror.kakao.com/ubuntu noble/main amd64 python3-configobj all 5.0.8-3 [33.8 kB]
Get:13 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-parsedatetime all 2.6-3 [32.8 kB]
Get:14 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-certbot all 2.9.0-1 [267 kB]
Get:15 http://mirror.kakao.com/ubuntu noble/main amd64 python3-icu amd64 2.12-1build2 [534 kB]
Get:16 http://mirror.kakao.com/ubuntu noble/universe amd64 certbot all 2.9.0-1 [89.2 kB]
Fetched 1638 kB in 0s (8344 kB/s)   
Preconfiguring packages ...
Selecting previously unselected package python3-openssl.
(Reading database ... 61319 files and directories currently installed.)
Preparing to unpack .../00-python3-openssl_23.2.0-1_all.deb ...
Unpacking python3-openssl (23.2.0-1) ...
Selecting previously unselected package python3-josepy.
Preparing to unpack .../01-python3-josepy_1.14.0-1_all.deb ...
Unpacking python3-josepy (1.14.0-1) ...
Selecting previously unselected package python3-certifi.
Preparing to unpack .../02-python3-certifi_2023.11.17-1_all.deb ...
Unpacking python3-certifi (2023.11.17-1) ...
Selecting previously unselected package python3-chardet.
Preparing to unpack .../03-python3-chardet_5.2.0+dfsg-1_all.deb ...
Unpacking python3-chardet (5.2.0+dfsg-1) ...
Selecting previously unselected package python3-idna.
Preparing to unpack .../04-python3-idna_3.6-2ubuntu0.1_all.deb ...
Unpacking python3-idna (3.6-2ubuntu0.1) ...
Selecting previously unselected package python3-urllib3.
Preparing to unpack .../05-python3-urllib3_2.0.7-1_all.deb ...
Unpacking python3-urllib3 (2.0.7-1) ...
Selecting previously unselected package python3-requests.
Preparing to unpack .../06-python3-requests_2.31.0+dfsg-1ubuntu1_all.deb ...
Unpacking python3-requests (2.31.0+dfsg-1ubuntu1) ...
Selecting previously unselected package python3-tz.
Preparing to unpack .../07-python3-tz_2024.1-2_all.deb ...
Unpacking python3-tz (2024.1-2) ...
Selecting previously unselected package python3-rfc3339.
Preparing to unpack .../08-python3-rfc3339_1.1-4_all.deb ...
Unpacking python3-rfc3339 (1.1-4) ...
Selecting previously unselected package python3-acme.
Preparing to unpack .../09-python3-acme_2.9.0-1_all.deb ...
Unpacking python3-acme (2.9.0-1) ...
Selecting previously unselected package python3-configargparse.
Preparing to unpack .../10-python3-configargparse_1.7-1_all.deb ...
Unpacking python3-configargparse (1.7-1) ...
Selecting previously unselected package python3-configobj.
Preparing to unpack .../11-python3-configobj_5.0.8-3_all.deb ...
Unpacking python3-configobj (5.0.8-3) ...
Selecting previously unselected package python3-parsedatetime.
Preparing to unpack .../12-python3-parsedatetime_2.6-3_all.deb ...
Unpacking python3-parsedatetime (2.6-3) ...
Selecting previously unselected package python3-certbot.
Preparing to unpack .../13-python3-certbot_2.9.0-1_all.deb ...
Unpacking python3-certbot (2.9.0-1) ...
Selecting previously unselected package python3-icu.
Preparing to unpack .../14-python3-icu_2.12-1build2_amd64.deb ...
Unpacking python3-icu (2.12-1build2) ...
Selecting previously unselected package certbot.
Preparing to unpack .../15-certbot_2.9.0-1_all.deb ...
Unpacking certbot (2.9.0-1) ...
Setting up python3-configargparse (1.7-1) ...
Setting up python3-parsedatetime (2.6-3) ...
Setting up python3-icu (2.12-1build2) ...
Setting up python3-openssl (23.2.0-1) ...
Setting up python3-tz (2024.1-2) ...
Setting up python3-chardet (5.2.0+dfsg-1) ...
Setting up python3-configobj (5.0.8-3) ...
Setting up python3-certifi (2023.11.17-1) ...
Setting up python3-idna (3.6-2ubuntu0.1) ...
Setting up python3-urllib3 (2.0.7-1) ...
Setting up python3-josepy (1.14.0-1) ...
Setting up python3-rfc3339 (1.1-4) ...
Setting up python3-requests (2.31.0+dfsg-1ubuntu1) ...
Setting up python3-acme (2.9.0-1) ...
Setting up python3-certbot (2.9.0-1) ...
Setting up certbot (2.9.0-1) ...
Created symlink /etc/systemd/system/timers.target.wants/certbot.timer -> /usr/lib/systemd/system/certbot.timer.
Processing triggers for man-db (2.12.0-4build2) ...
```

설치가 완료되었으면 다음으로 넘어간다.

### 2. 발급 시도 및 DNS TXT record 입력

>certbot으로 DNS challenge를 시도할 때마다 DNS TXT의 value는 변경이 되며 확인 후 TXT record에 입력해야 함
{: .prompt-danger}

Let’s Encrypt의 *(Wildcard) 인증서는 DNS Challenge를 통해 발급이 가능하다.  
처음 발급 시도 시 DNS TXT record에 value를 입력하라고 나온다.

```shell
admin@lxc-haproxy:~$ sudo certbot certonly --manual --preferred-challenges dns -d "*.norikae.kro.kr" -d
 "norikae.kro.kr"
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): me@formellow.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y
Account registered.
Requesting a certificate for *.norikae.kro.kr and norikae.kro.kr

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name:

_acme-challenge.norikae.kro.kr.

with the following value:

x9ynEumSFum5gAEZY922eliTkrnv4NVOT6bV7vqfyAM

(This must be set up in addition to the previous challenges; do not remove,
replace, or undo the previous challenge tasks yet. Note that you might be
asked to create multiple distinct TXT records with the same name. This is
permitted by DNS standards.)

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.norikae.kro.kr.
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

이 상태에서 domain에 DNS TXT record를 입력한다.  
위에 나와있는 것과 같이 value를 같이 넣어줘야 한다.

![TXT(SPF)](/assets/img/post/linux/2024-08-08-linux-obtain_letsencrypt_wildcard_cert_using_certbot/1.png)
_TXT(SPF)_

다 넣었으면 위에 나와있는 Admin Toolbox와 같이 입력하여서 확인해본다.  
<https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.norikae.kro.kr.>

![;ANSWER 값 확인](/assets/img/post/linux/2024-08-08-linux-obtain_letsencrypt_wildcard_cert_using_certbot/2.png)
_;ANSWER 값 확인_

**;ANSWER** 값에 나와있는 것과 같이 TXT가 제대로 입력이 되었으면 진행이 가능하다.

```shell
# 위 prompt를 이어서 진행
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/norikae.kro.kr/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/norikae.kro.kr/privkey.pem
This certificate expires on 2024-11-06.
These files will be updated when the certificate renews.

NEXT STEPS:
- This certificate will not be renewed automatically. Autorenewal of --manual certificates requires the use of an authentication hook script (--manual-auth-hook) but one was not provided. To renew this certificate, repeat this same certbot command before the certificate's expiry date.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

### 3. LB(Load Balancer)에 적용하기 위한 pem 키 합치기

`fullchain.pem`과 `privkey.pem` 두개의 키는 분리가 되어있기 때문에 `haproxy`와 같은 곳에서는 바로 적용이 안된다.  
간단히 한 줄의 명령어를 통해서 합친 pem 키를 만들어서 사용하면 된다.

```shell
# 기본 ssl 보관 경로인 /etc/ssl/private 이하에 생성
admin@lxc-haproxy:/etc/ssl$ sudo cat /etc/letsencrypt/live/norikae.kro.kr/fullchain.pem /etc/letsencrypt/live/norikae.kro.kr/privkey.pem | sudo tee /etc/ssl/private/norikae.kro.kr.pem
-----BEGIN CERTIFICATE-----
MIIDkTCCAxegAwIBAgISBCnr4dAuGDyZ/Hc9i79iELKGMAoGCCqGSM49BAMDMDIx
CzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQDEwJF
NjAeFw0yNDA4MDgwMjU5MzJaFw0yNDExMDYwMjU5MzFaMBsxGTAXBgNVBAMMECou
bm9yaWthZS5rcm8ua3IwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAATyVFPWS/YF
kc5ggagRlO7QBBFjHUXXtOWTx7wZ9GklJwm50hprSu6oTJjhTzVhyaihdWCecBGN
8xlmT8T3J19Uo4ICIjCCAh4wDgYDVR0PAQH/BAQDAgeAMB0GA1UdJQQWMBQGCCsG
AQUFBwMBBggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBT730O7XBa3
9/xk/FfCGFcXrK/gzDAfBgNVHSMEGDAWgBSTJ0aYA6lRaI6Y1sRCSNsjv1iU0jBV
BggrBgEFBQcBAQRJMEcwIQYIKwYBBQUHMAGGFWh0dHA6Ly9lNi5vLmxlbmNyLm9y
ZzAiBggrBgEFBQcwAoYWaHR0cDovL2U2LmkubGVuY3Iub3JnLzArBgNVHREEJDAi
ghAqLm5vcmlrYWUua3JvLmtygg5ub3Jpa2FlLmtyby5rcjATBgNVHSAEDDAKMAgG
BmeBDAECATCCAQQGCisGAQQB1nkCBAIEgfUEgfIA8AB2AD8XS0/XIkdYlB1lHIS+
DRLtkDd/H4Vq68G/KIXs+GRuAAABkTAi8v8AAAQDAEcwRQIgagM76hD3cFbAIZBc
xPUdCKnfkl9bwTHt2VU5/r35dQgCIQDxxOFjG2cYCbQvFtXCf9E0JincF4ZlUcMG
yvaNZRuicQB2AN/hVuuqBa+1nA+GcY2owDJOrlbZbqf1pWoB0cE7vlJcAAABkTAi
874AAAQDAEcwRQIgDtwtVxZZq34uyfkN8JpLS1lDKbVPcgd8q07d4+u9+ZACIQCJ
zUZCoaOZBrFPENaRR7OfKFBRjIcLFneG5Ev80l/71TAKBggqhkjOPQQDAwNoADBl
AjEA0BEqDFSo/LU04tPErNMek7VseVgE9co/gOKWGdLPE+EixxS4BLgZBMKW4yKb
h7iNAjAFO4i1xh/A+tnP/LNHmbIE0R7NaN+NuaFF+NtNqHtcBkh8ghbb8OWtLSok
sKf3saY=
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIEVzCCAj+gAwIBAgIRALBXPpFzlydw27SHyzpFKzgwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMjQwMzEzMDAwMDAw
WhcNMjcwMzEyMjM1OTU5WjAyMQswCQYDVQQGEwJVUzEWMBQGA1UEChMNTGV0J3Mg
RW5jcnlwdDELMAkGA1UEAxMCRTYwdjAQBgcqhkjOPQIBBgUrgQQAIgNiAATZ8Z5G
h/ghcWCoJuuj+rnq2h25EqfUJtlRFLFhfHWWvyILOR/VvtEKRqotPEoJhC6+QJVV
6RlAN2Z17TJOdwRJ+HB7wxjnzvdxEP6sdNgA1O1tHHMWMxCcOrLqbGL0vbijgfgw
gfUwDgYDVR0PAQH/BAQDAgGGMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
ATASBgNVHRMBAf8ECDAGAQH/AgEAMB0GA1UdDgQWBBSTJ0aYA6lRaI6Y1sRCSNsj
v1iU0jAfBgNVHSMEGDAWgBR5tFnme7bl5AFzgAiIyBpY9umbbjAyBggrBgEFBQcB
AQQmMCQwIgYIKwYBBQUHMAKGFmh0dHA6Ly94MS5pLmxlbmNyLm9yZy8wEwYDVR0g
BAwwCjAIBgZngQwBAgEwJwYDVR0fBCAwHjAcoBqgGIYWaHR0cDovL3gxLmMubGVu
Y3Iub3JnLzANBgkqhkiG9w0BAQsFAAOCAgEAfYt7SiA1sgWGCIpunk46r4AExIRc
MxkKgUhNlrrv1B21hOaXN/5miE+LOTbrcmU/M9yvC6MVY730GNFoL8IhJ8j8vrOL
pMY22OP6baS1k9YMrtDTlwJHoGby04ThTUeBDksS9RiuHvicZqBedQdIF65pZuhp
eDcGBcLiYasQr/EO5gxxtLyTmgsHSOVSBcFOn9lgv7LECPq9i7mfH3mpxgrRKSxH
pOoZ0KXMcB+hHuvlklHntvcI0mMMQ0mhYj6qtMFStkF1RpCG3IPdIwpVCQqu8GV7
s8ubknRzs+3C/Bm19RFOoiPpDkwvyNfvmQ14XkyqqKK5oZ8zhD32kFRQkxa8uZSu
h4aTImFxknu39waBxIRXE4jKxlAmQc4QjFZoq1KmQqQg0J/1JF8RlFvJas1VcjLv
YlvUB2t6npO6oQjB3l+PNf0DpQH7iUx3Wz5AjQCi6L25FjyE06q6BZ/QlmtYdl/8
ZYao4SRqPEs/6cAiF+Qf5zg2UkaWtDphl1LKMuTNLotvsX99HP69V2faNyegodQ0
LyTApr/vT01YPE46vNsDLgK+4cL6TrzC/a4WcmF5SRJ938zrv/duJHLXQIku5v0+
EwOy59Hdm0PT/Er/84dDV0CSjdR/2XuZM3kpysSKLgD1cKiDA+IRguODCxfO9cyY
Ig46v9mFmBvyH04=
-----END CERTIFICATE-----
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQg0LMTp5u7F6RMdPup
Luw4p9kaEHFt5tl1VtGt2vZ6SIahRANCAATyVFPWS/YFkc5ggagRlO7QBBFjHUXX
tOWTx7wZ9GklJwm50hprSu6oTJjhTzVhyaihdWCecBGN8xlmT8T3J19U
-----END PRIVATE KEY-----
```

이렇게 합쳐진 pem 파일은 LB 또는 Proxy server(`haproxy` 등)에 사용하면 된다.

## Cloudflare DNS challenge를 통해 자동 갱신 하기
---

DNS challenge를 통해 Let’s encrypt의 인증서를 자동으로 갱신하여서 사용하는 방법이다.  
DNS challenge를 지원하는 곳에서만 가능하며, 이런 서비스를 이용 하는 곳이라면 도메인은 구매를 해야한다.  
위의 글에서 이어 진행하는 것이여서, 아래의 글에선 중복되는 것들이나 디렉토리가 이미 생성 되어 있을 수 있다.

### 1. 관련 패키지 설치

```shell
dor1@lxc-haproxy:~$ sudo apt install certbot python3-certbot-dns-cloudflare
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
certbot is already the newest version (2.9.0-1).
The following additional packages will be installed:
  libxslt1.1 python3-bs4 python3-cloudflare python3-cssselect python3-html5lib python3-lxml python3-soupsieve
  python3-webencodings
Suggested packages:
  python3-genshi python-lxml-doc
The following NEW packages will be installed:
  libxslt1.1 python3-bs4 python3-certbot-dns-cloudflare python3-cloudflare python3-cssselect python3-html5lib python3-lxml
  python3-soupsieve python3-webencodings
0 upgraded, 9 newly installed, 0 to remove and 67 not upgraded.
Need to get 1722 kB of archives.
After this operation, 7190 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://mirror.kakao.com/ubuntu noble/main amd64 libxslt1.1 amd64 1.1.39-0exp1build1 [167 kB]
Get:2 http://mirror.kakao.com/ubuntu noble/main amd64 python3-soupsieve all 2.5-1 [33.0 kB]
Get:3 http://mirror.kakao.com/ubuntu noble/main amd64 python3-bs4 all 4.12.3-1 [109 kB]
Get:4 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-cloudflare all 2.11.1-1ubuntu1 [41.9 kB]
Get:5 http://mirror.kakao.com/ubuntu noble/universe amd64 python3-certbot-dns-cloudflare all 2.0.0-1 [9000 B]
Get:6 http://mirror.kakao.com/ubuntu noble/main amd64 python3-webencodings all 0.5.1-5 [11.5 kB]
Get:7 http://mirror.kakao.com/ubuntu noble/main amd64 python3-html5lib all 1.1-6 [88.8 kB]
Get:8 http://mirror.kakao.com/ubuntu noble/main amd64 python3-lxml amd64 5.2.1-1 [1243 kB]
Get:9 http://mirror.kakao.com/ubuntu noble/main amd64 python3-cssselect all 1.2.0-2 [18.5 kB]
Fetched 1722 kB in 0s (6564 kB/s)
Selecting previously unselected package libxslt1.1:amd64.
(Reading database ... 62400 files and directories currently installed.)
Preparing to unpack .../0-libxslt1.1_1.1.39-0exp1build1_amd64.deb ...
Unpacking libxslt1.1:amd64 (1.1.39-0exp1build1) ...
Selecting previously unselected package python3-soupsieve.
Preparing to unpack .../1-python3-soupsieve_2.5-1_all.deb ...
Unpacking python3-soupsieve (2.5-1) ...
Selecting previously unselected package python3-bs4.
Preparing to unpack .../2-python3-bs4_4.12.3-1_all.deb ...
Unpacking python3-bs4 (4.12.3-1) ...
Selecting previously unselected package python3-cloudflare.
Preparing to unpack .../3-python3-cloudflare_2.11.1-1ubuntu1_all.deb ...
Unpacking python3-cloudflare (2.11.1-1ubuntu1) ...
Selecting previously unselected package python3-certbot-dns-cloudflare.
Preparing to unpack .../4-python3-certbot-dns-cloudflare_2.0.0-1_all.deb ...
Unpacking python3-certbot-dns-cloudflare (2.0.0-1) ...
Selecting previously unselected package python3-webencodings.
Preparing to unpack .../5-python3-webencodings_0.5.1-5_all.deb ...
Unpacking python3-webencodings (0.5.1-5) ...
Selecting previously unselected package python3-html5lib.
Preparing to unpack .../6-python3-html5lib_1.1-6_all.deb ...
Unpacking python3-html5lib (1.1-6) ...
Selecting previously unselected package python3-lxml:amd64.
Preparing to unpack .../7-python3-lxml_5.2.1-1_amd64.deb ...
Unpacking python3-lxml:amd64 (5.2.1-1) ...
Selecting previously unselected package python3-cssselect.
Preparing to unpack .../8-python3-cssselect_1.2.0-2_all.deb ...
Unpacking python3-cssselect (1.2.0-2) ...
Setting up python3-webencodings (0.5.1-5) ...
Setting up python3-html5lib (1.1-6) ...
Setting up libxslt1.1:amd64 (1.1.39-0exp1build1) ...
Setting up python3-cssselect (1.2.0-2) ...
Setting up python3-soupsieve (2.5-1) ...
Setting up python3-bs4 (4.12.3-1) ...
Setting up python3-cloudflare (2.11.1-1ubuntu1) ...
/usr/lib/python3/dist-packages/CloudFlare/api_decode_from_openapi.py:10: SyntaxWarning: invalid escape sequence '\{'
  match_identifier = re.compile('\{[A-Za-z0-9_]*\}')
Setting up python3-lxml:amd64 (5.2.1-1) ...
Setting up python3-certbot-dns-cloudflare (2.0.0-1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.3) ...
```

### 2.  CF API Token추가

Cloudflare에서 API Token은 `My profile → Appearance → API Tokens`에 있다.  
이 곳에서 `Create Token → Edit zone DNS → Use template → Zone Resources 선택 → Continue to Summary → Create Token`으로 생성하면 된다.  
생성 하면 API Token을 아래와 같이 설정파일에 넣어준다.

```shell
# API Token 추가
dor1@lxc-haproxy:~$ sudo vi /etc/letsencrypt/cloudflare.ini
dns_cloudflare_api_token=Hl5r9GWzELMedsbUxNexg8phqSIpcKwgjJ1uKle0
```

`certbot`에서 읽을 수 있도록 권한도 변경한다.

```shell
dor1@lxc-haproxy:~$ sudo chmod 600 /etc/letsencrypt/cloudflare.ini
```

### 3. \*(Wildcard) 인증서 발급

위와 같이 준비 되었으면 한 줄로 간단히 인증서 발급이 가능하다.

```shell
# URL에 대해 변경 하여 진행
dor1@lxc-haproxy:~$ sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini -d "*.nexsyslab.com" -d nexsyslab.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for *.nexsyslab.com and nexsyslab.com
Waiting 10 seconds for DNS changes to propagate

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/nexsyslab.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/nexsyslab.com/privkey.pem
This certificate expires on 2024-11-24.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Expire date가 **2024-11-24** 인 것을 확인 할 수 있다.  
이후 자동 갱신 시 expire date에 최대한 가깝게 설정하면 된다.  
보통 만료 2주 정도 전이 적절하다.

### 4. 자동 갱신

`crontab`을 사용하여  갱신 할 수 있다.  
갱신은 만료 14일 전에 자동으로 작동할 수 있도록 해본다.  
방법은 여러개가 있지만 사용하기 쉽게 shell 파일을 만들어서 `crontab`으로 등록해서 사용하면 된다.  
아래의 script는 `haproxy`에 대해 구성이 되어있는 상태에서 사용하는 것을 가정으로 한다.

```shell
# /usr/local/bin/certbot-renew-check.sh
dor1@lxc-haproxy:~$ sudo vi /usr/local/bin/certbot-renew-check.sh
#!/bin/bash

DOMAIN="nexsyslab.com"
CERT_DIR="/etc/letsencrypt/live/$DOMAIN"
PEM_FILE="$CERT_DIR/nexsyslabhaproxy.pem"

# 1. SSL 인증서의 만료일까지 남은 기간 계산
EXPIRY_DATE=$(date -d "$(openssl x509 -enddate -noout -in $CERT_DIR/fullchain.pem | cut -d= -f2)" +%s)
CURRENT_DATE=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_DATE - $CURRENT_DATE) / 86400 ))

# 2. 인증서 만료일까지 14일 이하 남았는지 확인
if [ "$DAYS_LEFT" -le 14 ]; then
    echo "Certificate will expire in $DAYS_LEFT days. Attempting to renew..."
    
    # 3. 인증서 갱신 시도
    if /usr/bin/certbot renew --quiet --no-self-upgrade; then
        echo "Certificate successfully renewed."

        # 4. 갱신된 인증서와 개인 키를 결합하여 haproxy.pem 파일 생성
        cat "$CERT_DIR/fullchain.pem" "$CERT_DIR/privkey.pem" > "$PEM_FILE"
        echo "Combined certificate and key into $PEM_FILE."

        # 5. HAProxy 재시작
        systemctl reload haproxy
        echo "HAProxy reloaded with the new certificate."
    else
        echo "Failed to renew the certificate."
    fi
else
    echo "Certificate is valid for more than 14 days ($DAYS_LEFT days left). No renewal needed."
fi
```

위와 같이 shell을 짜준 뒤 사용할 수 있도록 권한을 바꿔준다.

```shell
dor1@lxc-haproxy:~$ sudo chmod 755 /usr/local/bin/certbot-renew-check.sh
```

이후 `crontab`에 대한 shell을 등록 해줘서 갱신하면 된다.

```shell
dor1@lxc-haproxy:~$ sudo crontab -e
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command

0 3 * * * /usr/local/bin/certbot-renew-check.sh
```

이렇게 하면 매일 새벽 3시마다 해당 script를 실행 시키고, 인증서의 만료일이 14일 전이면 갱신이 된 후 `haproxy`에서도 바로 인식이 된다.