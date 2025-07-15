---
title: Ubuntu - SSH key를 통한 무인증 로그인
date: 2022-07-22 11:14:00 +0900
categories: [Linux, Ubuntu]
tags: [linux, ubuntu, ssh]
description: Ubuntu에서 서버간 쉬운 접속을 위한 SSH 무인증 로그인 방법이다.
---

>OpenSSH_8.2p1 / 대부분의 Linux 배포판
{: .prompt-info}

>Host
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 서버 간 SSH(SCP) 통신이 필요할 경우 사용한다.
* Command based로 이뤄진 Cluster 구축 때는 필수로 사용한다.
* node to node 통신이 가능하기에 외부에서 접속이 되면 보안 상 문제가 생길 수 있다.
* 기본적으로 서버 간 접속 시 SSH 비밀번호 대신 SSH key를 제출하여 접속하는 방식으로 접근된다.

## 무인증 로그인
---

SSH key를 우선적으로 만들고 시작해야 한다.  
`ssh-keygen`이라는 패키지를 통해 진행한다.

### 1. ssh-keygen 사용

```shell
nvadmin@DGX-A100-80G-n1:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/nvadmin/.ssh/id_rsa):
Created directory '/home/nvadmin/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/nvadmin/.ssh/id_rsa
Your public key has been saved in /home/nvadmin/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Z3fkQuch1TtkyNEAVu/jj95MhSRvEjABolXAuUA1by8 nvadmin@DGX-A100-80G-n1
The key's randomart image is:
+---[RSA 3072]----+
|   ..o*+o.++++*. |
|    .oo+  .o +.+.|
|    .. .o   = B..|
|      .. . . @.= |
|        E + + Boo|
|         + . =. o|
|               ..|
|               =.|
|             .o +|
+----[SHA256]-----+
```

key에 대해 `passphrase`을 설정 할 수 있으나 외부에서 사용하는 경우 아니면, 오히려 번거러운 작업이 되기때문에 간단하게 **Enter키**를 통하여 패스 할 수 있다.

```shell
nvadmin@DGX-A100-80G-n1:~$ ll ~/.ssh
total 16
drwx------ 2 nvadmin nvadmin 4096  3월 31 17:06 ./
drwxr-xr-x 6 nvadmin nvadmin 4096  3월 31 17:06 ../
-rw-rw-r-- 1 nvadmin nvadmin 2610  3월 31 17:06 autorized_keys
-rw------- 1 nvadmin nvadmin 2610  3월 31 17:06 id_rsa
-rw-r--r-- 1 nvadmin nvadmin  577  3월 31 17:06 id_rsa.pub
```

`$HOME/.ssh`안에 생성된 키들이 존재한다.  
`autorized_keys` 파일은 첫 생성일 경우에는 없을 수도 있다.  
파일에 대한 설명은 다음과 같다.

| 파일명            | 설명                                                                                                            |
| ----------------- | --------------------------------------------------------------------------------------------------------------- |
| `id_rsa`          | **private key**, 외부에 노출될 경우 보안적으로 문제가 생긴다.                                                   |
| `id_rsa.pub`      | **public key**, 접속하려는 서버에 `authorized_keys` 안에 입력하게 되면 무인증이 가능해진다.                     |
| `authorized_keys` | **키값 저장 파일**, 접속 하려는 서버의 `$HOME/.ssh` 디렉토리 아래에 위치하면서 `id_rsa.pub` 키의 값을 저장한다. |

### 2. SSH key 등록

SSH server의 `authorized_keys`의 내용은 SSH client의 `id_rsa.pub` 내용과 일치해야 한다.  
SSH 접속 시에 SSH server는 `id_rsa` 파일과 `authorized_keys`이 일치해야 하며 일치하지 않으면 접속이 불가능하다.  
Server의 `id_rsa.pub`의 내용을 Client의 `authorized_keys`에 넣어준다.

```shell
# Server
nvadmin@DGX-A100-80G-n1:~$ scp ~/.ssh/id_rsa.pub nvadmin@192.168.11.174:~/id_rsa.pub

# Client
nvadmin@DGX-A100-80G-n2:~$ cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
```

### 3. SSH 접속

접속 시 만약 Host(/etc/hosts)에 등록이 되어있다면 `hostname`으로 접속할 수 있다.

```shell
nvadmin@DGX-A100-80G-n1:~$ ssh nvadmin@DGX-A100-80G-n2
nvadmin@DGX-A100-80G-n2:~$
```

### 4. 권한

각 파일에 대한 권한(파일 생성 시 기본 권한)은 다음과 같이 고정하는 것이 관리에 좋다.

```shell
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys
```

## Script
---

간단하게 Shell script를 만들어봤다.  
해당 Script에는 IP 별로 `SSH key` 및 `hostname`을 자동으로 등록 해준다.  
각각 node들은 계정 및 비밀번호가 다 같아야한다.

```shell
#! /bin/bash

#################################################
#                                               #
#                                               #
#               MADE BY Dor1                    #
#                                               #
#                                               #
#################################################

# Change the node ip
IP=("10.0.0.181" "10.0.0.191" "10.0.0.196")

echo -e "\nInput your password"
read passwd

PASSWORD=$passwd

echo $PASSWORD | sudo -S apt update
echo $PASSWORD | sudo -S apt install sshpass -y

keycopy ()
{
        mkdir -p ~/tmp/{hostname,pubkey}

        for iplist in "${IP[@]}";
        do
                sshpass -p $PASSWORD ssh $iplist -o StrictHostKeyChecking=no 'ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ""; cat /etc/hostname | tee ~/hostname';
                sshpass -p $PASSWORD scp $iplist:~/.ssh/id_rsa.pub ~/tmp/pubkey/$iplist;
                PUBKEY=$(cat ~/tmp/pubkey/$iplist);
                sshpass -p $PASSWORD scp $iplist:~/hostname ~/tmp/hostname/$iplist;
                HOST=$(cat ~/tmp/hostname/$iplist);
                sshpass -p $PASSWORD ssh $iplist -o StrictHostKeyChecking=no "rm ~/hostname";
                echo $PUBKEY >> ~/.ssh/authorized_keys;
                echo "$iplist $HOST" >>  ~/tmp/hosts;
        done

        for iplist in "${IP[@]}";
        do
                AUTHKEY=$(cat ~/.ssh/authorized_keys);
                sshpass -p $PASSWORD ssh $iplist -o StrictHostKeyChecking=no "echo '$AUTHKEY' | tee ~/.ssh/authorized_keys;
                echo 'Public key copied to' $iplist;";
        done

        return
}

hostscopy ()
{
        for iplist in "${IP[@]}";
        do
                HOSTS=$(cat ~/tmp/hosts);
                sshpass -p $PASSWORD ssh $iplist -o StrictHostKeyChecking=no "cat /etc/hosts | tee ~/hosts";
                sshpass -p $PASSWORD ssh $iplist -o StrictHostKeyChecking=no "echo '$HOSTS' >> ~/hosts";
        done

        for iplist in "${IP[@]}";
        do
                sshpass -p $PASSWORD ssh $iplist -o StrictHostKeyChecking=no "echo '$PASSWORD' | sudo -S mv ~/hosts /etc/hosts";
        done

        return
}

keycopy
hostscopy

rm -rf ~/tmp

exit 0
```