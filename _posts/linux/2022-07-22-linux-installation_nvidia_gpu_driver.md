---
title: Linux - NVIDIA GPU 드라이버 설치
date: 2022-07-22 09:25:00 +0900
categories: [Linux]
tags: [linux, nvidia]
description: Linux에서 NVIDIA GPU Driver를 설치하는 방법이다.
---

>Ubuntu 20.04 LTS / CentOS 7.9
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 설치 방법
---

* NVIDIA GPU 드라이버 설치 방법은 대표적으로 두 가지가 있다.
  1. Runfile 설치하는 방법
  2. Package 설치하는 방법 (Data Center/Tesla 모델만 가능)
* `nvidia-smi`의 명령어에 대해서는 다음의 글로 이동해서 살펴본다.  
  [Linux - nvidia-smi 명령어 정리](/posts/linux-nvidia-smi_command/)

## Ubuntu (Runfile & Package)
---

### Runfile 설치하는 방법

#### 1. Blacklist nouveau 처리

Linux에서의 `nouveau`는 Windows의 `표준 VGA 그래픽 어댑터`와 같은 드라이버라고 생각하면 쉽다.  
`nouveau`는 오픈 소스 그래픽 장치 드라이버다.  
과거에 NVIDIA에서 Tegra family를 사용하기 위하여 독립적으로 지원을 해주었다.  
`nouveau` 드라이버를 사용하게 된다면 NVIDIA GPU 드라이버 설치가 정상적으로 작동하지 않아 **Module blacklist** 처리를 해주어야 된다.

- `tee`를 통해 새로운 파일을 만들어 준다.

```shell
echo "
blacklist nouveau
options nouveau modeset=0" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
```

- conf 파일을 Kernel에 적용한다.

```shell
sudo update-initramfs -u
```

`update-initramfs`는 RAM 파일 시스템에 update를 해주는 것이다.  
Linux는 프로세스가 **rootfs에 kernel이 마운트** 되는데, 이곳에 위에 적은 `blacklist-nouveau.conf`를 **kernel에 update 해준다.**

#### 2. NVIDIA GPU 드라이버 설치 및 확인

1. [NVIDIA Linux 전용 드라이버](https://www.nvidia.com/Download/index.aspx?lang=en-us)를 받아준다.  
   선택 과정에서 OS는 `Linux 64-bit`로 설정 한다.
2. Driver는 `.run` 확장자를 갖고 있으며 파일에 쓰기권한(`chmod +x`)를 주어 설치하면 된다.  
   Ubuntu 20.04 LTS 부터는 x86 라이브러리를 설치하는 과정에서 선택 할 수 있다.
3. 설치 과정에 `dkms`를 넣어 설치해주면 향후 Kernel Upgrade 시 안정적으로 사용이 가능하다.

```shell
sudo ./NVIDIA-Linux-x86_64-510.54.run --dkms
```

드라이버 확인

```shell
nvidia-smi
```

## Package 설치하는 방법

>해당 방법은 Data Center/Tesla 모델만 가능 하다.  
Ubuntu packge는 `apt`와 `dpkg`로 관리하게 된다.  
`apt`와 `dpkg` 사용에 익숙하지 않다면 위의 방법이 훨씬 더 관리 및 업데이트 하기에 쉽다.
{: .prompt-danger}

### NVIDIA GPU 드라이버 설치 및 확인

1. [NVIDIA Linux 전용 드라이버](https://www.nvidia.com/Download/index.aspx?lang=en-us)를 받아준다.
   선택 과정에서 OS는 `Linux 64-bit Ubuntu 20.04`로 설정 한다.
2. 받은 파일을 설치 해준다.  
   <https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#driver-installation>

```shell
sudo dpkg -i nvidia-driver-local-repo-ubuntu2004-510.47.03_1.0-1_amd64.deb
sudo apt-key add /var/nvidia-driver-local-repo-ubuntu2004-510.47.03/7fa2af80.pub
sudo apt update
sudo apt install nvidia-driver-510
```

드라이버 확인

```shell
nvidia-smi
```

## CentOS (Runfile)
---

### 1. Blacklist nouveau 처리

Linux에서의 `nouveau`는 Windows의 `표준 VGA 그래픽 어댑터`와 같은 드라이버라고 생각하면 쉽다.  
`nouveau`는 오픈 소스 그래픽 장치 드라이버다.  
`nouveau` 드라이버를 사용하게 된다면 NVIDIA GPU 드라이버 설치가 정상적으로 작동하지 않아 **Module blacklist** 처리를 해주어야 된다.

- `tee`를 통해 새로운 파일을 만들어 준다.

```shell
sudo vim /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
```

- conf 파일을 Kernel에 적용한다.

```shell
sudo dracut -f
```

### 2. NVIDIA GPU 드라이버 설치 및 확인

1. [NVIDIA Linux 전용 드라이버](https://www.nvidia.com/Download/index.aspx?lang=en-us)를 받아준다.  
   선택 과정에서 OS는 `Linux 64-bit`로 설정 한다.
2. Driver는 `.run` 확장자를 갖고 있으며 파일에 쓰기권한(`chmod +x`)를 주어 설치하면 된다.
3. 설치 과정에 `dkms`를 넣어 설치해주면 향후 Kernel Upgrade 시 안정적으로 사용이 가능하다.  
   (CentOS 또는 RHEL에서는 `epel-release` [Repository](https://docs.fedoraproject.org/en-US/epel/#How_can_I_use_these_extra_packages.3F)에서 `dkms`설치 할 수 있음)

```shell
sudo ./NVIDIA-Linux-x86_64-510.54.run --dkms
```

드라이버 확인

```shell
nvidia-smi
```

## Troubleshooting
---

>최신 버전에서는 해결되어 있을 수 있음
{: .prompt-danger}

### Ubuntu

#### 1. IGNORE_CC_MISMATCH

![IGNORE_CC_MISMATCH](/assets/img/post/linux/2022-07-22-linux-installation_nvidia_gpu_driver/1.png)
_IGNORE_CC_MISMATCH_

보통 Ubuntu의 `gcc` 버전이 높은 상태에서 `dkms`와 함께 사용 시에 나타난다.  
해결은 두 가지의 방법이 있다.

**1\. gcc 버전을 낮추기**

`gcc`의 버전을 `apt` 패키지 관리자를 통하여 낮춰 주는 방법이다.

```shell
# gcc 버전 확인
sudo apt-cache madison gcc-9

# 의존성과 함께 다운그레이드
sudo apt install gcc-9=9.3.0-10ubuntu2 cpp-9=9.3.0-10ubuntu2 gcc-9-base=9.3.0-10ubuntu2 libgcc-9-dev=9.3.0-10ubuntu2 libasan5=9.3.0-10ubuntu2
```

**2\. Runfile에서 처리하기**

위의 방법보다는 간단하지만 간혹 `gcc`가 너무 높아져 버리면 downgrade가 필요 할 수 있다.

```shell
sudo ./NVIDIA-Linux-x86_64-510.54.run --dkms --no-cc-version-check
```