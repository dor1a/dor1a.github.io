---
title: Linux(Ubuntu) - User로 systemd 사용하기
date: 2024-07-31 11:04:00 +0900
categories: [Linux]
tags: [linux, ubuntu, systemd]
description: sudo 권한이 없는 상태로 systemctl 명령어를 사용하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `sudo` 권한이 없는 상태로 `systemctl` 명령어를 통해 service를 제어해야하는 상황을 고려하여 생각해보게 되었다.
* binary(/usr/bin 등) 파일에 대한 권한이 필요할 시에는 그룹 권한을 줘서 처리하면 될 것으로 생각이 든다.

> ref.
> - <https://blog.victormendonca.com/2018/05/14/creating-a-simple-systemd-user-service/>

## python의 http.server로 systemd 구성해보기
---

`python`의 module 중 `http.server`가 있는데 이것으로 간단하게 구성해보기로 한다.  
`http.server`는 local 파일에 대해서 http로 띄어주는 module이며, Ubuntu에서는 기본적으로 설치 되어있다.  
`systemd`에 대해서 작성 후 start 해보기로 한다.  
기본적으로 user에서 바라보는 `systemd`의 설정 파일은 `~/.config/systemd/user` 이하에 있다.

```shell
# ~/.config/systmed/user 디렉토리 생성
admin@ubuntu-2404-test:~$ mkdir -p ~/.config/systemd/user

# service config 작성
admin@ubuntu-2404-test:~$ vi ~/.config/systemd/user/http_server.service
[Unit]
Description=Simple Python HTTP Server

[Service]
ExecStart=/home/admin/http_server/start_server.sh
Restart=always

[Install]
WantedBy=default.target

# config를 systemd에 인식(daemon reload)
admin@ubuntu-2404-test:~$ systemctl --user daemon-reload
```

`systemd`의 config는 root(sudo)권한으로 작동하는 `systemd` config와 양식이 동일하다.  
보통 root에서 동작하는 config의 위치는 `/lib/systemd/system` 이하에 있으니, 비슷하게 만들어도 된다.  
다음의 링크에서 작성에 대해서 참고할 수 있긴 하지만, 가독성이 좋지는 않아 그다지 추천하지는 않는다.  
<https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html>  
`python` 또한 작성 한다.

```shell
# 위의 [Service] 항목의 경로대로 디렉토리 생성
admin@ubuntu-2404-test:~$ mkdir -p ~/http_server

# http.server의 python 작성
admin@ubuntu-2404-test:~$ vi ~/http_server/start_server.sh
#!/bin/bash
python3 -m http.server 8080
```

이후 `systemctl`의 명령어를 통해 진행한다.

```shell
admin@ubuntu-2404-test:~$ systemctl --user start http_server.service
admin@ubuntu-2404-test:~$ systemctl --user status http_server.service
● http_server.service - Simple Python HTTP Server
     Loaded: loaded (/home/admin/.config/systemd/user/http_server.service; disabled; preset: enabled)
     Active: active (running) since Wed 2024-07-31 08:20:29 UTC; 13s ago
   Main PID: 21538 (start_server.sh)
      Tasks: 2 (limit: 19062)
     Memory: 9.6M (peak: 9.9M)
        CPU: 67ms
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/http_server.service
             ├─21538 /bin/bash /home/admin/http_server/start_server.sh
             └─21539 python3 -m http.server 8080

Jul 31 08:20:29 ubuntu-2404-test systemd[1244]: Started http_server.service - Simple Python HTTP Server.
```

잘 작동하는지 `curl`로 확인해본다.

```shell
admin@ubuntu-2404-test:~$ curl localhost:8080
<!DOCTYPE HTML>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".bash_history">.bash_history</a></li>
<li><a href=".bash_logout">.bash_logout</a></li>
<li><a href=".bashrc">.bashrc</a></li>
<li><a href=".cache/">.cache/</a></li>
<li><a href=".config/">.config/</a></li>
<li><a href=".lesshst">.lesshst</a></li>
<li><a href=".local/">.local/</a></li>
<li><a href=".profile">.profile</a></li>
<li><a href=".ssh/">.ssh/</a></li>
<li><a href=".sudo_as_admin_successful">.sudo_as_admin_successful</a></li>
<li><a href=".viminfo">.viminfo</a></li>
<li><a href=".Xauthority">.Xauthority</a></li>
<li><a href="http_server/">http_server/</a></li>
</ul>
<hr>
</body>
</html>
```