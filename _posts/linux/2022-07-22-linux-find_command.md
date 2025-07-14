---
title: Linux - find 명령어 정리
date: 2022-07-22 11:03:00 +0900
categories: [Linux]
tags: [linux, find]
description: Linux에서 파일 또는 디렉토리를 찾아볼 수 있는 find에 대해서 정리해 보았다.
---

>대부분의 Linux 배포판
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 명령어
---

**[기본 사용법]**

```shell
find '<search_path>' [options]
```

### 1. 특정 파일명을 찾는 경우

**[기본 사용법]**

```shell
find '<search_path>' -name '<file_name>'
```

**[예시]**

현재 디렉토리에서 txt 파일을 모두 찾으려는 경우

```shell
find . -name *.txt
```

tmp 디렉토리 안에서 이름이 "1" 인 모든 확장자를 찾아 내려하는 경우

```shell
find /tmp -name 1.*
```

### 2. 특정 디렉토리를 찾는 하는 경우

**[기본 사용법]**

```shell
find '<search_path>' -type d -name '<directory_name>'
```

**[예시]**

/ 에서 test 디렉토리를 찾는 경우

```shell
find / -type d -name test
```

현재 디렉토리에서 tar이 포함되는 디렉토리를 찾는 경우

```shell
find . -type d -name *tar*
```

### 3. 특정 권한을 가진 파일을 찾는 경우

**[기본 사용법]**

```shell
find '<search_path>' -perm '<permission_num>' -name '<file_name>'
```

여기에서 `<permission_num>`은 Linux에서 기본적으로 파일이나 디렉토리에 대해 10자의 Permission 기호로 이뤄진 번호다.

예를들어 `-rwx—r-x` 가 있다고 가정했을때 이것을 **네 분류**로 나눠서 보면 된다.

* 처음 `–` **파일을 의미**,`d`**디렉토리를 의미**
* 두번째 `rwx` **소유자(Owner)를 의미**
* 세번째 `—` **그룹(Group) 사용자를 의미**
* 마지막 `r-x` **기타(Other) 사용자를 의미**

`r` Read(읽기)의 약자 / `w` Write(쓰기)의 약자 / `x` Execute(실행)의 약자

그리고 이것들을 숫자로 표현하게 된다면, **r = 4 / w = 2 / x = 1** 로 표현된다. 총합은 7이며 **7은 즉 모든 권한**이다.

**[예시]**

현재 디렉토리에서 777(모든 권한)을 가지며 txt 파일을 모두 찾는 경우

```shell
find . -perm 777 -name *.txt
```

### 4. 특정 소유자의 파일을 찾는 경우

**[기본 사용법]**

```shell
find '<search_path>' -user '<user_name>' -name '<file_name>'
```

**[예시]**

tmp 디렉토리에서 root 소유자인 txt 파일을 모두 찾는 경우

```shell
find /tmp -user root -name *.txt
```

### 5. 특정 그룹의 파일을 찾는 경우

**[기본 사용법]**

```shell
find '<search_path>' -group '<group_name>' -name '<file_name>'
```

**[예시]**

root 디렉토리에서 web 그룹 소유권이 있는 파일을 찾는 경우

```shell
find / -group web
```

현재 디렉토리에서 web 그룹에게 소유권이 있는 test.txt 파일을 찾는 경우

```shell
find . -group web -name test.txt
```

### 6. 특정 용량 기준으로 파일을 찾는 경우

**[기본 사용법]**

```shell
find '<search_path>' -size '<capacity>' -name '<file_name>'
```

`<capacity>`에는 다음과 같은 용량이 사용 가능 함

* `C` byte
* `k` kilobyte
* `M` megabyte
* `G` gigabyte

**[예시]**

현재 디렉토리에서 2M 이상 되는 파일을 찾는 경우

```shell
find . -size +2M
```

tmp 디렉토리에서 용량이 2k 이상 4k 미만이 되는 파일을 찾는 경우

```shell
find ./tmp/ -size +2k -size 4k
```

### 7. 특정 시간 기준으로 파일을 찾는 경우

**[기본 사용법]**

```shell
find '<search_path>' -atime '<time_set>' -name '<file_name>'
find '<search_path>' -mtime '<time_set>' -name '<file_name>'
find '<search_path>' -ctime '<time_set>' -name '<file_name>'
```

`<time_set>`은 Linux 시간 표기법 의해 **세 가지로 구분**이 된다.

* `atime (Access Time)` 마지막으로 파일에 접근한 시간, vi 및 cat 등의 명령어로 열람한 시간들이 이에 해당 됨
* `mtime (Modification Time)` 마지막으로 파일의 내용을 수정한 시간
* `ctime (Change time)` chmod 나 chown 등 파일의 속성 및 권한을 변경한 시간

일반적으로 쓰이는 *ls* 의 명령어에서 보이는 시간은 `mtime`  
`ctime` 은 *ls -lc* 의 명령어로 확인  
`atime` 은 *ls -lu* 의 명령어로 확인

**[예시]**

사용자 홈 디렉토리에서 마지막으로 수정한지 2달 이상 된 모든 파일을 찾는 경우

```shell
find ~/ -mtime +60
```

사용자 홈 디렉토리에서 마지막으로 수정한지 4달 이상 된 txt 파일을 찾는 경우

```shell
find ~/ -mtime +120 -name *.txt
```

etc 디렉토리에서 파일의 속성을 변경한지 1달 이상 된 모든 파일을 찾는 경우

```shell
find /etc -ctime +30
```

## 사용 사례
---

### 1. find로 해당 파일, 디렉토리를 찾은 후 `-exec`를 통한 실행 권한

**[기본 사용법]**

```shell
find '<search_path>' -exec '<exe_command' {} \;
```

`{}`: find로 찾은 파일, 디렉토리를 의미  
`\;`: `-exec`의 옵션의 내용 끝을 의미

**[예시]**

/src에 있는 1234.txt 파일을 찾아서 이 텍스트 파일의 내용을 /src의 위치에 5678.txt 파일로 생성 시킬 때

```shell
find /src -name 1234.txt -exec cat {} > /usr/local/src/5678.txt \;
```