---
title: Linux - LVM(Logical Volume Manager) 명령어 정리
date: 2022-08-08 10:40:00 +0900
categories: [Linux]
tags: [linux, lvm]
description: Linux에서 많이 사용하는 Disk volume manager인 LVM의 명령어에 대해 정리해 보았다.
---

>CentOS 7.9
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* LVM 사용 시 자주 사용하는 명령어 총 정리한다.
* LVM은 Redhat 계열에서 많이 사용하기에 CentOS로 진행 해 보았다.
* LVM은 기본적으로 `PV(Physical Volumes)`, `VG(Volume Group)`, `LV(Logical Volumes)`, `FS(File System)`으로 구성되어 있다.

## 명령어 정리
---

### LVM(Logical Volume Manager) 명령어

| 명령어           | 내용                    |
| ---------------- | ----------------------- |
| `lvm dumpconfig` | LVM 구성 정보 출력      |
| `lvmdump`        | LVM 구성 덤프 생성      |
| `lvm formats`    | 메타데이터 초기화       |
| `lvm diskscan`   | 모든 장치 검색하여 출력 |

### PV(Physical Volumes) 명령어

| 명령어      | 내용                                                               | 예시                                                                                  |
| ----------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| `pvcreate`  | PV 생성 (Partition을 PV단위 로 생성)                               | `pvcreate /dev/sda1`                                                                  |
| `pvremove`  | PV 삭제                                                            | `pvremove /dev/sda1`                                                                  |
| `pvchange`  | PV 상태 변경 <br> Flags <br> -u : UUID 부여 <br> -x : PV 사용 설정 | `pvchange -u /dev/sda1` <br> `pvchange -x y /dev/sda1` <br> `pvchange -x n /dev/sda1` |
| `pvs`       | PV 정보 출력                                                       | `pvs`                                                                                 |
| `pvdisplay` | PV 상태 출력                                                       | `pvdisplay`                                                                           |
| `pvscan`    | PV에 사용되는 모든 디바이스 스캔                                   | `pvscan`                                                                              |

### VG(Volume Group) 명령어

| 명령어      | 내용                                                                                                | 예시                                                                                    |
| ----------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `vgcreate`  | VG 생성 (PV를 생성한 다음 진행)                                                                     | `vgcreate group_name /dev/sda1 /dev/sdb1`                                               |
| `vgremove`  | VG 삭제                                                                                             | `vgremove group_name`                                                                   |
| `vgreduce`  | VG에 속해있는 PV를 삭제                                                                             | `vgreduce group_name /dev/sdb1`                                                         |
| `vgextend`  | VG에 PV를 추가시켜 확장                                                                             | `vgextend group_name /dev/sdb1`                                                         |
| `vgchange`  | VG 상태 변경 <br> Flags <br> -a : PV 사용 설정 <br> -l : VG 안에 생성할 수 있는 최대 LV 개수의 지정 | `vgchange -a y group_name`<br>`vgchange -a n group_name`<br>`vgchange -l 10 group_name` |
| `vgs`       | VG 정보 출력                                                                                        | `vgs`                                                                                   |
| `vgdisplay` | VG 상태 출력 <br> Flags <br> -v : 상세히                                                            | `vgdisplay`                                                                             |
| `vgscan`    | VG에 속해있는 모든 디바이스 스캔 및 LVM cache file 재생성                                           | `vgscan`                                                                                |

### LV(Logical Volume) 명령어

| 명령어      | 내용                                                                                                                                                                                                                        | 예시                                                                                                                               |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `lvcreate`  | LV 생성 (VG를 생성한 다음 진행) <br> Flags <br> -L : 사이즈 지정, 단위는 K(kilobytes), M(megabytes), G(gigabytes), T(terabytes) 사용 가능 <br> -l : 사이즈 지정, pe의 개수로 설정이 가능 (1pe = 4MB) <br> -n : LV 이름 지정 | `lvcreate -L 100G -n data1 group_name` <br> `lvcreate -l 500 -n backup1 group_name` <br> `lvcreate -l 100%FREE -n etc1 group_name` |
| `lvremove`  | LV 삭제 <br> Flags <br> -f : 강제(force)                                                                                                                                                                                    | `lvremove etc1`                                                                                                                    |
| `lvreduce`  | LV 용량 축소 <br> Flags <br> -L : 지정한 사이즈로 축소, 단위는 `lvcreate`와 같음 <br> -l : 지정한 pe의 개수 만큼 축소                                                                                                       | `lvreduce -L 50G /dev/group_name/data1` <br> `lvreduce -L -30G /dev/group_name/data1`                                              |
| `lvextend`  | LV 용량 확장 <br> Flags <br> -L : 지정한 사이즈만큼 확장, 단위는 `lvcreate`와 같음 <br> -l : 지정한 pe의 개수 만큼 확장                                                                                                     | `lvextend -L +30G /dev/group_name/data1` <br> `lvextend -l +100%FREE /dev/group_name/data1`                                        |
| `lvchange`  | LV 상태 변경 <br> Flags <br> -a : LV 사용 설정                                                                                                                                                                              | `lvchange -a y /dev/group_name/data1` <br> `lvchange -a n /dev/group_name/data1`                                                   |
| `lvs`       | LV 정보 출력                                                                                                                                                                                                                | `lvs`                                                                                                                              |
| `lvdisplay` | LV 상태 출력                                                                                                                                                                                                                | `lvdisplay`                                                                                                                        |
| `lvscan`    | LV에 사용되는 모든 디바이스 스캔                                                                                                                                                                                            | `lvscan`                                                                                                                           |

### FS(File System) 명령어

| 명령어       | 내용                            | 예시                                                                      |
| ------------ | ------------------------------- | ------------------------------------------------------------------------- |
| `mkfs`       | 파일 시스템 생성 및 포맷        | `mkfs.ext4 /dev/group_name/data1` <br> `mkfs.xfs /dev/group_name/backup1` |
| `resize2fs`  | ext4 파일시스템의 사이즈 재설정 | `resize2fs /dev/group_name/data1`                                         |
| `xfs_growfs` | xfs 파일시스템의 사이즈 재설정  | `xfs_growfs /dev/group_name/backup1`                                      |

