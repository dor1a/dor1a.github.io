---
title: Linux - nvidia-smi 명령어 정리
date: 2022-07-22 10:43:00 +0900
categories: [Linux]
tags: [linux, nvidia]
description: Linux에서 NVIDIA GPU Driver를 설치하면 사용하게 되는 nvidia-smi의 명령어를 정리해 보았다.
---

>Ubuntu 20.04 LTS
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## nvidia-smi 란?
---

* NVIDIA Driver를 설치하면 기본적으로 설치되는 명령어이다.
* GPU의 상태를 확인할 수 있다.

## nvidia-smi 살펴보기
---

![nvidia-smi](/assets/img/post/linux/2022-08-11-linux-nvidia-smi_command/1.png)
_nvidia-smi_

1. NVIDIA-SMI  
  → NVIDIA-SMI binary의 버전이다.
2. Driver Version  
  → GPU Driver의 버전이다.
3. CUDA Version  
  → GPU Driver가 지원하는 CUDA의 버전, 현재 사용하고 있는 CUDA의 버전이 아니다!
4. GPU | FAN
  - GPU  
    → GPU의 INDEX를 표시해준다.  
    → INDEX는 메인보드에서 순차적으로 꽃았을때 나오는 INDEX와는 다르다.
  - FAN  
    → GPU(Only Active GPU)에 장착되어있는 FAN의 사용율을 보여준다.
5. Name | Persistence-M | Temp | Perf | Pwr:Usage/Cap   
  - Name  
    → GPU의 모델이다.
  - Persistence-M  
    → Persistence(파워지속성) Mode를 표시해준다.  
    → Default는 Off이며, On을 해주면 GPU의 전력 제한을 걸수 있다.  
    → On일 경우엔 Persistence 상태이다보니 GPU 사용 시 Load delay time을 아낄 수 있다.
  - Temp  
    → GPU의 온도를 표시해준다.  
    → 각각의 GPU throttling 범위는 `nvidia-smi -q`를 통하여 알아 볼 수 있다.
  - Perf  
    → Performance를 표시해준다.  
    → P0 - P12가 보통의 범위이며, 숫자가 낮을 수록 High-performance를 의미한다.
  - Pwr:Usage/Cap  
    → 전력 사용량을 표시해준다.  
    → Usage는 현재 사용량, Cap은 최대치이다.
6. Bus-ID | Disp.A | Memory-Usage
  - Bus-ID  
    → Mainboard slot의 Bus-ID를 표시해준다.
  - Disp.A  
    → Linux에는 desktop과 server 버전이 있는데, desktop버전에서 화면 출력이 되면 그 GPU 카드에는 On 표시가 되어있다.
  - Memory-Usage  
    → GPU의 memory를 표시해준다.
7. Volatile GPU-Util / Uncorr. ECC / Compute M. / MIG M. 
  - Volatile GPU-Util  
    → GPU의 사용율을 표시해준다.
  - Uncorr. ECC  
    → GPU ECC Memory의 error 카운트를 표기해준다.  
    → ECC를 끄기위해서는 `nvidia-smi -e 0` 해주면 된다.  
    → ECC가 검출된 GPU를 사용시에 system hang이 걸린다.
  - Compute M.  
    → 현재 사용중인 Compute mode가 표시된다.  
    → Mode는 다음과 같이 있다.  
    `0. Default / 1. Exclusive_Thread / 2. Prohibited / 3. Exclusive_Process`  
    → Mode 변경은 `nvidia-smi -c 0`와 같이 가능하다.
  - MIG M.  
    → MIG 사용 여부에 대해 표시해준다.  
    → MIG를 사용할 경우 `nvdia-smi -mig 1` 해주면 된다.
8. Processes
  → 현재 GPU에 대해 사용중인 PID, Process name를 표시해준다.