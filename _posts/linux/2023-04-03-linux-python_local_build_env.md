---
title: Linux(Ubuntu) - Python Local(build) env 구성
date: 2023-04-03 15:36:00 +0900
categories: [Linux]
tags: [linux, ubuntu, python]
description: Ubuntu에서 Python에 대해 Local Env(환경)을 구성하는 방법이다.
---

>Ubuntu 22.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Python은 여러가지 방법으로 가상 환경을 만들 수 있다.
* 3rd party에서는 대표적으로 [Anaconda](https://www.anaconda.com/)가 있지만 버그가 많아 추천하지는 않는다.
* `virtualenv`는 `python` 공식 패키지가 아닌 외부의 패키지로 이뤄져 있으며, **local에 설치 된 여러가지 python 버전에 대하여 가상 환경을 제공**해준다.
  > ex) `/usr/bin/python3.9`와 `/usr/bin/python3.8`이 있다면, `virtualenv -p`의 parameter를 통하여 가상 환경을 제공
* `venv`는 `python`에서 공식으로 사용되는 가상 환경 이며, python 3.3 버전 이상부터는 외부 패키지가 아닌 Python의 내장 module로 존재한다.
  보통`pip` 패키지에 대하여 local python과 분리를 위해 사용된다.
* python 3버전 에서는`venv`가 표준이기에 `venv`기준으로 작성한다.

## venv 구성
---

`Ubuntu`에서는 간단한 패키지로 `venv`를 설치 할 수 있다.

```shell
dor1@dor1-lxc-provisioning ~ ❯ sudo apt install python3-venv
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-pip-whl python3-setuptools-whl python3.10-venv
The following NEW packages will be installed:
  python3-pip-whl python3-setuptools-whl python3-venv python3.10-venv
0 upgraded, 4 newly installed, 0 to remove and 5 not upgraded.
Need to get 2474 kB of archives.
After this operation, 2888 kB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 http://mirror.kakao.com/ubuntu jammy-security/universe amd64 python3-pip-whl all 22.0.2+dfsg-1ubuntu0.3 [1679 kB]
Get:2 http://mirror.kakao.com/ubuntu jammy-security/universe amd64 python3-setuptools-whl all 59.6.0-1.2ubuntu0.22.04.1 [788 kB]
Get:3 http://mirror.kakao.com/ubuntu jammy-security/universe amd64 python3.10-venv amd64 3.10.12-1~22.04.2 [5724 B]
Get:4 http://mirror.kakao.com/ubuntu jammy-security/universe amd64 python3-venv amd64 3.10.6-1~22.04 [1038 B]
Fetched 2474 kB in 0s (12.5 MB/s)
Selecting previously unselected package python3-pip-whl.
(Reading database ... 61921 files and directories currently installed.)
Preparing to unpack .../python3-pip-whl_22.0.2+dfsg-1ubuntu0.3_all.deb ...
Unpacking python3-pip-whl (22.0.2+dfsg-1ubuntu0.3) ...
Selecting previously unselected package python3-setuptools-whl.
Preparing to unpack .../python3-setuptools-whl_59.6.0-1.2ubuntu0.22.04.1_all.deb ...
Unpacking python3-setuptools-whl (59.6.0-1.2ubuntu0.22.04.1) ...
Selecting previously unselected package python3.10-venv.
Preparing to unpack .../python3.10-venv_3.10.12-1~22.04.2_amd64.deb ...
Unpacking python3.10-venv (3.10.12-1~22.04.2) ...
Selecting previously unselected package python3-venv.
Preparing to unpack .../python3-venv_3.10.6-1~22.04_amd64.deb ...
Unpacking python3-venv (3.10.6-1~22.04) ...
Setting up python3-setuptools-whl (59.6.0-1.2ubuntu0.22.04.1) ...
Setting up python3-pip-whl (22.0.2+dfsg-1ubuntu0.3) ...
Setting up python3.10-venv (3.10.12-1~22.04.2) ...
Setting up python3-venv (3.10.6-1~22.04) ...
```

설치가 되었으면, 간단한 명령어(`python -m venv <name>)`로 구성한다.  
구성 후 `activate`를 시키려면 rc 파일(`.bashrc` /`.zshrc`)에 등록하여 사용하면 간편하다.

```shell
# venv 구성
dor1@dor1-lxc-provisioning ~ ❯ python -m venv .python_venv
dor1@dor1-lxc-provisioning ~ ❯ ll
total 200
drwxr-x--- 6 dor1 dor1  4096 Sep 13 14:34 .
drwxr-xr-x 3 root root  4096 Sep  7 16:29 ..
-rw------- 1 dor1 dor1    67 Sep 13 10:17 .Xauthority
-rw------- 1 dor1 dor1  3795 Sep 10 17:49 .bash_history
-rw-r--r-- 1 dor1 dor1   220 Sep  7 16:29 .bash_logout
-rw-r--r-- 1 dor1 dor1  3810 Sep  8 17:29 .bashrc
drwx------ 5 dor1 dor1  4096 Sep 13 14:27 .cache
drwxrwxr-x 4 dor1 dor1  4096 Sep 10 18:08 .dotfiles
-rw-rw-r-- 1 dor1 dor1   100 Sep 10 18:10 .gitconfig
-rw------- 1 dor1 dor1    20 Sep 10 18:11 .lesshst
lrwxrwxrwx 1 dor1 dor1    28 Sep 10 17:49 .oh-my-zsh -> .dotfiles/_common/.oh-my-zsh
lrwxrwxrwx 1 dor1 dor1    27 Sep 10 17:49 .p10k.zsh -> .dotfiles/_common/.p10k.zsh
-rw-r--r-- 1 dor1 dor1   807 Sep  7 16:29 .profile
-rw------- 1 dor1 dor1     0 Sep 13 14:29 .python_history
# 확인
drwxrwxr-x 5 dor1 dor1  4096 Sep 13 14:28 .python_venv
drwxr-xr-x 2 dor1 dor1  4096 Sep 13 11:37 .ssh
-rw-r--r-- 1 dor1 dor1     0 Sep  7 16:30 .sudo_as_admin_successful
lrwxrwxrwx 1 dor1 dor1    22 Sep 10 17:49 .vim -> .dotfiles/_common/.vim
-rw------- 1 dor1 dor1 19353 Sep 13 11:41 .viminfo
lrwxrwxrwx 1 dor1 dor1    24 Sep 10 17:49 .vimrc -> .dotfiles/_common/.vimrc
-rw-rw-r-- 1 dor1 dor1 49168 Sep 10 17:46 .zcompdump
-rw-rw-r-- 1 dor1 dor1 50848 Sep 11 16:14 .zcompdump-dor1-lxc-provisioning-5.8.1
-rw------- 1 dor1 dor1 23238 Sep 13 14:34 .zsh_history
lrwxrwxrwx 1 dor1 dor1    24 Sep 10 17:49 .zshrc -> .dotfiles/_common/.zshrc

# venv 적용(rc에 등록하면 간편 함)
dor1@dor1-lxc-provisioning ~ ❯ source ~/.python_venv/bin/activate

# pip version을 통하여 venv 확인(위치가 local 디렉토리로 변경 됨)
dor1@dor1-lxc-provisioning ~ ❯ pip --version
pip 22.0.2 from /home/dor1/.python_venv/lib/python3.10/site-packages/pip (python 3.10)
```