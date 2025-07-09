---
title: Keycloak - Google OAuth 연동
date: 2025-05-01 16:16:00 +0900
categories: [Service]
tags: [service, keycloak, google, docker]
description: Docker에서 구동 중인 Keycloak에서 Google OAuth를 연동 해봤다.
---

>keycloak/keycloak:26.2.2
{: .prompt-info}

>Container
{: .prompt-warning}

>Web
{: .prompt-tip}

## 개요
---

* SSO(Single Sign On)를 목표로 사용하는 Keycloak에서 좀 더 유연하고 편한 사용을 위해 Google OAuth를 연동 해봤다.

## Keycloak에서 Identity providers를 이용하여 Google provider 생성
---

Keycloak에 접속하여 좌측 메뉴 중 `Configure → Identity providers`를 선택한다.  
그 이후 Google을 누르면 provider에 대해 생성할 수 있다.

![Google provider](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/1.png)
_Google provider_

여기서 `Redirct URI`는 하단에서 OAuth Client 생성 할 때 사용 하는 메뉴다.  
그리고 `Client ID`와 `Client secret` 또한 생성이 완료 되면 나오는 값을 입력 해주면 된다.

### Google Cloud Console에서 OAuth Client 만들기

우선 [Google Cloud Console](https://console.cloud.google.com/apis/credentials)에 접속한다.  
프로젝트가 없다면 상단에 Google Cloud 오른쪽에서 생성하면 된다.

![Console 상단 부분](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/2.png)
_Console 상단 부분_

생성을 완료 하였으면 `Create credentials → OAuth client iD`를 누르면 `Configure consent screen` 화면이 출력된다.  
이것은 사용 전 Google’s OAuth servers에 접근하는 것에 대한 동의를 의미한다.  
이후 좌측의 `Clients`를 탭을 눌러 생성을 하려해도 `Google Auth Platform not configured yet`이라는 메세지가 뜨며 API와 Sigin-in에 대한 확인을 요청한다.

![Clients 탭](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/3.png)
_Clients 탭_

간단하게 `Get started`를 하여 brand을 만들 Information들을 채우고, `Audience`부분은 `External`로 맞춰주면 된다.  
그 다음 죄측 탭에서 다시 `Clients`을 눌러 `Create client`한다.  
`Application type`은 `Web application`으로 한다.  
하단의 `Authroized redirect URIs`에는 keycloak의 redirect URI를 넣어준다.

![Web application 확인](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/4.png)
_Web application 확인_

이후 만들게 되면 팝업이 뜨며 `Client ID`와 `Client secret`을 이 생성된다.

![Client iD & Client secret 생성 완료](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/5.png)
_Client iD & Client secret 생성 완료_

팝업을 무시한다 해도 ID 칸에 제대로 생성 되어있어 거기서 확인하면 된다.  
여기서 생성 된 `Client ID`와 `Client secret`을 상단에서 진행하던 `Google provider`에서 채워주면 된다.  
채워 준 뒤 Keycloak을 다시 접속 해보면 Google auth가 생긴 것을 확인 할 수 있다.

![Keycloak Sigin in 창](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/6.png)
_Keycloak Sigin in 창_

## 특정 ID에 다른 Google 메일을 갖고 있는 대상을 link 하기
---

다음과 같은 상황으로 예시로 한다.

관리자가 admin이라는 계정을 사용하는데 Google 메일은 admin@gmail.com이 아니고, abc@gmail.com의 메일을 사용한다고 가정 해 보았을 경우 admin 계정에 abc@gmail.com을 link하는 것이다.

이것을 처리하기 위해서는 조금 복잡하지만 JWT(JSON Web Token)을 찾아 진행해야 한다.

우선 위와 같이 `Identity providers → Google provider`가 연결 되어있어야 한다.  
`Linked identity providers`의 메뉴는 Keycloak 접속 후 좌측에 `Users → 사용자 계정 → Identity provider links`에 위치해 있다.

![Linked identity providers](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/7.png)
_Linked identity providers_

여기에 필요한 정보는 `User ID`와 `Username`이다.  
Username이야 gmail 계정을 써줘도 상관이 없는데 `User ID`가 문제다.

### 1. Google Cloud Console에서 localhost 추가하기

우선 JWT을 받아 `User ID`를 찾으려면 위에서 작업했던 Google Cloud Console에 들어가서 다음과 같이 항목을 추가한다.  
여기서 주의해야 할 것은 `http://localhost:8080`은 작동이 안된다는 것이다.  
Google에서 `localhost`에 대해서는 보안적으로 막아 놓은 듯한데, 왜 127.0.0.1은 되는 것인가 싶다.

![http://127.0.0.1:8080으로만 작성할 것](/assets/img/post/service/2025-05-01-keycloak-integrate_google_oauth/8.png)
_http://127.0.0.1:8080으로만 작성할 것_

이후 다음과 같은 URL에 `Client ID`(Google OAuth client ID)와 함께 웹 브라우저에서 접속한다.

<https://accounts.google.com/o/oauth2/v2/auth?client_id=클라이언트ID&redirect_uri=http://127.0.0.1:8080&response_type=code&scope=openid%20email%20profile&access_type=offline&prompt=consent>

접속 후 로그인을 마치면 마지막 브라우저 URL에 `code=`이후의 값이 적혀서 나올 것이다.  
여기서 얻게 된 code값은 `id_token`을 받는데 결정적인 역할을 한다.

### 2. code를 가지고 JWT 받아 User ID 찾기

code를 얻었다면 `Postman`과 같은 곳에서 다음과 같이 작성해서 `id_token`을 얻는다.

>`User ID`의 값은 고유 값이다 보니 보안상 스크린샷이 따로 없다.  
`code=`이후의 값이 `4%2F0Ab_`와 같이 나왔을 경우 %2F는 `/` 이기 때문에 아래에 적을 시 유의해서 적는다.  
ex. `4%2F0Ab_` → `4/0Ab_`
{: .prompt-tip}

**URL:**

```plaintext
https://oauth2.googleapis.com/token
```

**Body (x-www-form-urlencoded):**

```plaintext
code=복사한_code_값
client_id=OAuth_클라이언트_ID
client_secret=OAuth_클라이언트_시크릿
redirect_uri=http://127.0.0.1:8080
grant_type=authorization_code
```

이렇게 해서 받게 된 `id_token`(JWT)는 다음과 같은 사이트에서 정보를 받을 수 있다.  
<https://jwt.io>  
사이트에서 복사해서 넣으면 오른쪽에서 `sub`항목을 찾으면 된다.  
여기서 `sub`항목이 `User ID`이다.  
`User ID`를 가지고 Keycloak에 등록 해주면 된다.

작업이 다 끝나면 꼭 **Google Cloud Console**의 `Authorized redirect URIs`에서 `http://127.0.0.1:8080` 항목을 지워준다.  
또한 `User ID`와 같은 값이 적혀 있는 곳(`Postman` 등)도 지워주면 좋다.