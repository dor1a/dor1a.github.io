---
title: Kubernetes - k3s 설치해보기
date: 2025-07-22 11:23:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, k3s]
description: k3s로 kubernetes를 설치하는 방법이다.
---

>k3s v1.32.6+k3s1
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Kubernetes를 설치 할 수 있는 방법은 여러가지 방법이 존재(`kubespary`,`kubesphere`,`k3s` 등)한다.
* 이 중 `k3s`는 k8s의 경량화(lightweight) 된 버전이며 경량화에는 다음과 같은 이유가 있다.
  1. k8s에서 `etcd`같은 것이 파일화(SQLite과 같은 embedded DB)가 되었다.
  2. 불필요한 내장 기능들과 플러그인들이 제거 되었으며 최소한의 기능만 제공한다.
  3. 여러 컴포넌트(API Server, Scheduler, Controller Manager 등)들이 binary(bin 파일)로 패키징 되어있어서 구성이 되어있다.
* k3s는 설치가 매우 간단하며 기본적으로 CRI(containerd), CNI(flannel), CSI(Local Path Provisioner), Ingress Controller(traefik), DNS(CoreDNS), LB(Klipper-lb)와 같이 가볍고 빠른 구성 요소로 제공 된다.
* 이 글에서는 다소 오래 된 CNI인 `flannel` 대신 on-prem에 유리한 `cilium`을 사용할 예정이며, k3s의 내장 된 `traefik`는 오래된 버전이여서 helm chart로 신규 버전을 올리는 것으로 한다.
* `cilium`을 배포하는 과정은 다음과 같은 링크를 참고하면 된다.  
  [Kubernetes - k3s에 cilium 배포해보기](/posts/kubernetes-deploy_cilium_in_k3s/)

>ref.
> - <https://docs.k3s.io/installation/>

## 설치
---

기본적으로 `ufw`/`firewalld`와 같이 OS 내 방화벽은 disabled 되어있는 것이 좋다.  
만약에 disabled가 어려운 환경일경우, 다음과 같은 포트를 개방해준다.

| Protocol | Port      | Source  | Destination | Description |
| -------- | --------- | ------- | ----------- |
| TCP      | 2379-2380 | Servers | Servers     | Required    | only for HA with embedded etcd                           |
| TCP      | 6443      | Agents  | Servers     | K3s         | supervisor and Kubernetes API Server                     |
| UDP      | 8472      | All     | nodes       | All nodes   | Required only for Flannel VXLAN                          |
| TCP      | 10250     | All     | nodes       | All nodes   | Kubelet metrics                                          |
| UDP      | 51820     | All     | nodes       | All nodes   | Required only for Flannel Wireguard with IPv4            |
| UDP      | 51821     | All     | nodes       | All nodes   | Required only for Flannel Wireguard with IPv6            |
| TCP      | 5001      | All     | nodes       | All nodes   | Required only for embedded distributed registry (Spegel) |
| TCP      | 6443      | All     | nodes       | All nodes   | Required only for embedded distributed registry (Spegel) |

**Test-bed**
- k3s-master1 (10.50.1.221)
- k3s-master2 (10.50.1.222)
- k3s-master3 (10.50.1.223)
- k3s-worker1 (10.50.1.231)
- k3s-worker2 (10.50.1.232)

개요에서 적었듯이 `flannel`과 `traefik`은 disable 한다.  
Control-plane 3대중 첫번째만 우선 설치 후 나머지 2대는 k3s server API를 바라보게 만들어 설치 해줘야 node가 올라온다.  
k3s는 기본적으로 root에서 실행되게 되어있는데 이 것의 권한을 조절 하려면 kubeconfig에 대해서 권한을 부여하면 된다.  
보통 group 권한으로 사용하니 그룹은 Read 권한만 부여하는 방법으로 사용하면 좋다.  
여기서는 `kubeusers`라는 그룹이 생성이 완료된 상태로 진행한다.  
설치 시 변수들은 아래의 링크에서 확인이 가능하다.  
<https://docs.k3s.io/cli/server>

```shell
# First control-plane
deepadmin@k3s-master1:~$ curl -sfL https://get.k3s.io | \
  INSTALL_K3S_EXEC="server \
  --cluster-init \
  --flannel-backend=none \
  --disable-network-policy \
  --disable=traefik,servicelb \
  --write-kubeconfig-mode=640 \
  --write-kubeconfig-group=kubeusers" \
  sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.32.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

# check node-token(first control-plane)
deepadmin@k3s-master1:~$ sudo cat /var/lib/rancher/k3s/server/node-token
K106e2c74157c0af9d7bb0a8b638861a039cca30f4f69836a8c1d0dbeea8e5eee4f::server:eb3d4e75061297a86e58382847fb5ba0

# second control-plane
deepadmin@k3s-master2:~$ curl -sfL https://get.k3s.io | \
  K3S_URL=https://10.50.1.221:6443 \
  K3S_TOKEN=K106e2c74157c0af9d7bb0a8b638861a039cca30f4f69836a8c1d0dbeea8e5eee4f::server:eb3d4e75061297a86e58382847fb5ba0 \
  INSTALL_K3S_EXEC="server \
  --flannel-backend=none \
  --disable-network-policy \
  --disable=traefik,servicelb \
  --write-kubeconfig-mode=640 \
  --write-kubeconfig-group=kubeusers" \
  sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.32.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/sha256sum-amd64.txt
[INFO]  Skipping binary downloaded, installed k3s matches hash
[INFO]  Skipping installation of SELinux RPM
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, already exists
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

# third control-plane
deepadmin@k3s-master3:~$ curl -sfL https://get.k3s.io | \
  K3S_URL=https://10.50.1.221:6443 \
  K3S_TOKEN=K106e2c74157c0af9d7bb0a8b638861a039cca30f4f69836a8c1d0dbeea8e5eee4f::server:eb3d4e75061297a86e58382847fb5ba0 \
  INSTALL_K3S_EXEC="server \
  --flannel-backend=none \
  --disable-network-policy \
  --disable=traefik,servicelb \
  --write-kubeconfig-mode=640 \
  --write-kubeconfig-group=kubeusers" \
  sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.32.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

# 확인(first control-plane)
deepadmin@k3s-master1:~$ kubectl get node
NAME          STATUS     ROLES                       AGE     VERSION
k3s-master1   NotReady   control-plane,etcd,master   11m     v1.32.6+k3s1
k3s-master2   NotReady   control-plane,etcd,master   6m29s   v1.32.6+k3s1
k3s-master3   NotReady   control-plane,etcd,master   2m59s   v1.32.6+k3s1
```

`NotReady` 되어있는 것은 CNI가 없기 때문에 서로 통신을 못해서 pod가 배포되지 않기 때문이다.  
마저 data-plane도 설치해본다.

```shell
# first data-plane
deepadmin@k3s-worker1:~$ curl -sfL https://get.k3s.io | \
  K3S_URL=https://10.50.1.221:6443 \
  K3S_TOKEN=K106e2c74157c0af9d7bb0a8b638861a039cca30f4f69836a8c1d0dbeea8e5eee4f::server:eb3d4e75061297a86e58382847fb5ba0 \
  sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.32.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent

# second data-plane
deepadmin@k3s-worker2:~$ curl -sfL https://get.k3s.io | \
  K3S_URL=https://10.50.1.221:6443 \
  K3S_TOKEN=K106e2c74157c0af9d7bb0a8b638861a039cca30f4f69836a8c1d0dbeea8e5eee4f::server:eb3d4e75061297a86e58382847fb5ba0 \
  sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.32.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.32.6+k3s1/sha256sum-amd64.txt
[INFO]  Skipping binary downloaded, installed k3s matches hash
[INFO]  Skipping installation of SELinux RPM
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, already exists
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent

# 확인(first control-plane)
deepadmin@k3s-master1:~$ kubectl get node
NAME          STATUS     ROLES                       AGE    VERSION
k3s-master1   NotReady   control-plane,etcd,master   20h    v1.32.6+k3s1
k3s-master2   NotReady   control-plane,etcd,master   20h    v1.32.6+k3s1
k3s-master3   NotReady   control-plane,etcd,master   20h    v1.32.6+k3s1
k3s-worker1   NotReady   <none>                      107s   v1.32.6+k3s1
k3s-worker2   NotReady   <none>                      50s    v1.32.6+k3s1

# 확인 pod,svc,deploy,replica
deepadmin@k3s-master1:~$ kubectl get all -A
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-5688667fd4-7fdfg                  0/1     Pending   0          20h
kube-system   pod/local-path-provisioner-774c6665dc-57d8f   0/1     Pending   0          20h
kube-system   pod/metrics-server-6f4c6675d5-ppq2z           0/1     Pending   0          20h

NAMESPACE     NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP                  20h
kube-system   service/kube-dns         ClusterIP   10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP   20h
kube-system   service/metrics-server   ClusterIP   10.43.186.107   <none>        443/TCP                  20h

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns                  0/1     1            0           20h
kube-system   deployment.apps/local-path-provisioner   0/1     1            0           20h
kube-system   deployment.apps/metrics-server           0/1     1            0           20h

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-5688667fd4                  1         1         0       20h
kube-system   replicaset.apps/local-path-provisioner-774c6665dc   1         1         0       20h
kube-system   replicaset.apps/metrics-server-6f4c6675d5           1         1         0       20h
```

이러면 설치는 완료 되었으며 helm은 다음과 같은 링크를 따라 설치하면 된다.  
<https://helm.sh/docs/intro/install/>  
helm 설치 후 `dial tcp 127.0.0.1:8080: connect: connection refused`와 같이 API를 못찾는다면 `kubeconfig`를 찾지못해서 그런 거니 바라보게만 해주면 된다.  
다음과 같은 작업이 필요하다.

```shell
deepadmin@k3s-master1:~$ vi ~/.bashrc
---

# 제일 마지막에 줄 추가
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

deepadmin@k3s-master1:~$ source ~/.bashrc
```

CNI를 추가로 붙여야 나머지 pod들도 정상적으로 올라오고 작동 된다.

## Label 작업
---

data-plane쪽에 label을 붙인다 하면 아래와 같이 하면 된다.  
`worker`를 추가로 붙이는 이유는 특정 helm chart에서는 아직 사용 되기 때문이다.

```shell
deepadmin@k3s-master1:~$ kubectl label node k3s-worker1 \
  node-role.kubernetes.io/data-plane=true \
  node-role.kubernetes.io/worker=true \
  --overwrite
node/k3s-worker1 labeled

deepadmin@k3s-master1:~$ kubectl label node k3s-worker2 \
  node-role.kubernetes.io/data-plane=true \
  node-role.kubernetes.io/worker=true \
  --overwrite
node/k3s-worker2 labeled

# 확인
deepadmin@k3s-master1:~$ kubectl get node
NAME          STATUS      ROLES                       AGE         VERSION
k3s-master1   NotReady    control-plane,etcd,master   20h         v1.32.6+k3s1
k3s-master2   NotReady    control-plane,etcd,master   20h         v1.32.6+k3s1
k3s-master3   NotReady    control-plane,etcd,master   20h         v1.32.6+k3s1
k3s-worker1   NotReady    data-plane,worker           6m43s       v1.32.6+k3s1
k3s-worker2   NotReady    data-plane,worker           6m26s       v1.32.6+k3s1
```