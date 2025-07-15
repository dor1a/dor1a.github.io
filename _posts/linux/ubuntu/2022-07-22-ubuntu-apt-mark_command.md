---
title: Ubuntu - apt-mark 명령어 사용해보기
date: 2022-07-22 10:23:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, apt-mark]
description: Ubuntu에서 apt 패키지를 관리하는 apt-mark를 사용 해본다.
---

>Ubuntu 20.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `Debian`에서도 동일하다.
* apt-mark는 설치 시 `package` 및 `dependency`에 대해 자동으로 또는 수동으로 설치한 것을 표시(관리자가 의도적으로 직접 설치) 또는 `hold`, `install`, `deinstall`, `purge` 등 수행 된 것들에 대해 표시가 가능하다.

## 사용법
---

```shell
SYNOPSIS
       apt-mark {-f=filename | {auto | manual} pkg...  |
                {showauto | showmanual} [pkg...] } | {-v | --version} |
                {-h | --help}

       apt-mark {hold | unhold | install | remove | purge} pkg...  |
                {showhold | showinstall | showremove | showpurge} [pkg...]
```

### 1. auto

**auto**는 `package` 및 `dependency`에 대하여 자동으로 설치 되는 것을 표시한다.  

**[기본 사용법 및 예시]**

```shell
sudo apt-mark auto curl
curl set to automatically installed.
```

### 2. manual

**manual**은 `package`를 수동으로 설치한 것에 대해 표시한다. **auto** 표시를 하였는데도 불구하고 `apt install` 등으로 설치하게 된다면 **manual**로 고정 된다.

**[기본 사용법 및 예시]**

```shell
sudo apt-mark manual curl
curl set to manually installed.
```

### 3. minimize-manual

**minimize-manual**은 `metapackage dependency`에 대하여 자동으로 설치 되는 것을 표시한다. 간단하게 설치된 `package`들을 최소화 시키는데 사용한다. 보통 `package` 환경 구성이나 `metapackage` 관리에 사용된다.

**[기본 사용법 및 예시]**

```shell
sudo apt-mark minimize-manual
The following packages will be marked as automatically installed:
  ca-certificates vim
Do you want to continue? [y/N]
```

### 4. showauto

**showauto**는 **auto**로 표시된 항목을 보여준다.

**[기본 사용법]**

```shell
sudo apt-mark showauto <package_name>
```

### 5. showmanual

**showmanual**는 **manual**로 표시된 항목을 보여준다.

**[기본 사용법]**

```shell
sudo apt-mark showmanual <package_name>
```

## 옵션
---

`man apt-mark`에서도 출력 되는 내용들이다.

### 1. package 변경 방지

**hold**는 `package`의 버전을 고정 시킨다. 이 작업을 했을 시 `package`들은 자동으로 설치 및 업그레이드가 되지 않는다.

**[기본 사용법]**

```shell
# 고정
sudo apt-mark hold <package_name>

# 고정 취소
sudo apt-mark unhold <package_name>

# 고정 된 항목 나열
sudo apt-mark showhold <package_name>
```

### 2. install, remove, purge 등 일정을 위한 옵션

`dpkg -l` 명령시에 `package` 앞부분에 나오는 항목에 대해 표시 수정을 통해 관리가 가능하다.  
이러한 것들은 위의 항목과 같은 형식으로 쉽게 지정이 가능하다.

**[기본 사용법]**

```shell
# install 표시
sudo apt-mark install <package_name>

# remove 표시
sudo apt-mark remove <package_name>

# purge 표시
sudo apt-mark purge <package_name>

# install 표시 된 항목 나열
sudo apt-mark showinstall <package_name>

# remove 표시 된 항목 나열
sudo apt-mark showremove <package_name>

# purge 표시 된 항목 나열
sudo apt-mark showpurge <package_name>
```