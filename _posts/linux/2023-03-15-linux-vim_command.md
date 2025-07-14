---
title: Linux - vim 명령 정리
date: 2023-03-15 14:17:00 +0900
categories: [Linux]
tags: [linux, vim]
description: Linux에서 가장 많이 사용하는 편집기인 vim의 명령에 대해 정리해 보았다.
---

>대부분의 Linux 배포판
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 유용한 명령에 대하여 정리한다.
* 필요 시 지속적으로 추가한다.

## vim 명렁
---

`vim`에서 기본적인 명령은 `esc + :` 상태에서 진행한다.

### 1. 특정 line 주석(#) 처리 및 삭제

visual mode(`esc + v`)에서 특정 line에 대하여 선택 후 주석 처리

```vim
:norm i#
```

visual mode(`esc + v`)에서 특정 line에 대하여 앞쪽 문자열 대하여 삭제

```vim
:norm 1x (앞쪽 1글자 삭제)
```

### 2. 주석(#) 줄 전체 삭제

처음 문자에 `#`이 들어간 줄 삭제

```vim
:g/^#/d
```

`#`이 포함 되어 있는 모든 줄 삭제

```vim
:g/\#/d
```

### 3. 특정 문자열 치환

deep을 admin으로 변경할 경우 (마지막 gc의 c는 confirm의 약자)

```vim
:%s/deep/admin/gc
```