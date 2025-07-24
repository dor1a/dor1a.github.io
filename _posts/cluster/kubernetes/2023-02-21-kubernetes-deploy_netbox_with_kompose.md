---
title: Kubernetes - NetBox를 kompose로 배포해보기
date: 2023-02-21 10:44:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, netbox, kompose]
description: Kubernetes에서 IPAM(IP Address Management)와 DCIM(DataCenter Infrastructure Management) 등 자산관리를 하는 오픈소스 솔루션인 NetBox를 배포해보았다.
---

>Kubernetes v1.24.7 / NetBox v3.4.4 / kompose v1.28.0 / Ubuntu 22.04 LTS
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* NetBox는 IPAM(IP Address Management)과 DCIM(DataCenter Infrastructure Management)을 구체적으로 관리하기 위하여 나온 오픈소스이다.
* `helm`으로 차트(<https://github.com/bootc/netbox-chart>)가 존재한다.
* `helm`으로도 deployment가 가능은 하지만 `env`에 대해 전부 수정하기는 힘들고, 구조 상 전부 뜯어 보는 것이 좋다고 생각이 들었다.
* `kompose`를 이용하여 manifest 파일을 직접 변환하게 되면 `env`에 대해 구체적으로 수정이 가능하고, 버전이 고정되어도 좋다 생각하여 진행하게 되었다.
* NetBox 구성에 들어가는 기본 service들은 다음과 같다.
  1. Redis (included cache)
  2. PostgreSQL
  3. NetBox (included worker)
  4. NetBox-Housekeeping

## Manifest 파일 변환 (kompose)
---

NetBox를 사용하려면 기본적으로 `kompose`가 설치 되어있어야 한다.  
`kompose`는 `docker-compose`의 파일을 kubernetes의 manifest로 변환 해주는 플러그인이다.  
`kompose`는 binary 형식으로 쉽게 설치 된다.

```shell
# binary
dor1@is-master:~$ curl -L https://github.com/kubernetes/kompose/releases/download/v1.28.0/kompose-linux-amd64 -o kompose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 19.3M  100 19.3M    0     0  5086k      0  0:00:03  0:00:03 --:--:-- 7024k

# 쓰기 권한 부여
dor1@is-master:~$ chmod +x kompose

# kubectl binary와 같은 위치로 이동
dor1@is-master:~$ sudo mv ./kompose /usr/local/bin

# 확인
dor1@is-master:~$ kompose version
1.28.0 (c4137012e)
```

`kompose`설치가 완료 되었다면, `netbox-docker`의 compose 파일 및 config(python, env) 파일을 k8s manifest 파일로 변경 해준다.

```shell
# source
dor1@is-master:~$ git clone https://github.com/netbox-community/netbox-docker.git
Cloning into 'netbox-docker'...
remote: Enumerating objects: 4070, done.
remote: Counting objects: 100% (4070/4070), done.
remote: Compressing objects: 100% (1658/1658), done.
remote: Total 4070 (delta 2342), reused 3999 (delta 2306), pack-reused 0
Receiving objects: 100% (4070/4070), 1010.18 KiB | 1.53 MiB/s, done.
Resolving deltas: 100% (2342/2342), done.

# move nessesary soruce
dor1@is-master:~$ mkdir netbox
dor1@is-master:~$ cp -r netbox-docker/{configuration,reports,scripts,env,docker-compose.yml} ./netbox
dor1@is-master:~$ cd netbox
dor1@is-master:~/netbox$ ll
total 32
drwxrwxr-x 7 dor1 dor1 4096 Feb 17 17:50 ./
drwxrwxrwx 8 dor1 dor1 4096 Feb 17 17:48 ../
drwxrwxr-x 3 dor1 dor1 4096 Feb 17 17:48 configuration/
-rw-rw-r-- 1 dor1 dor1 2104 Feb 17 17:48 docker-compose.yml
drwxrwxr-x 2 dor1 dor1 4096 Feb 17 17:48 env/
drwxrwxr-x 2 dor1 dor1 4096 Feb 17 17:48 reports/
drwxrwxr-x 2 dor1 dor1 4096 Feb 17 17:48 scripts/

# convert docker-compose to k8s manifest
dor1@is-master:~/netbox$ mkdir kompose
dor1@is-master:~/netbox$ kompose convert -f docker-compose.yml -o kompose
WARN Service "netbox" won't be created because 'ports' is not specified
WARN Volume mount on the host "~/netbox/configuration" isn't supported - ignoring path on the host
WARN Volume mount on the host "~/netbox/reports" isn't supported - ignoring path on the host
WARN Volume mount on the host "~/netbox/scripts" isn't supported - ignoring path on the host
WARN Ignoring user directive. User to be specified as a UID (numeric).
INFO Network netbox-default is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
WARN Service "netbox-housekeeping" won't be created because 'ports' is not specified
WARN Volume mount on the host "~/netbox/configuration" isn't supported - ignoring path on the host
WARN Volume mount on the host "~/netbox/reports" isn't supported - ignoring path on the host
WARN Volume mount on the host "~/netbox/scripts" isn't supported - ignoring path on the host
WARN Ignoring user directive. User to be specified as a UID (numeric).
INFO Network netbox-default is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
WARN Service "netbox-worker" won't be created because 'ports' is not specified
WARN Volume mount on the host "~/netbox/configuration" isn't supported - ignoring path on the host
WARN Volume mount on the host "~/netbox/reports" isn't supported - ignoring path on the host
WARN Volume mount on the host "~/netbox/scripts" isn't supported - ignoring path on the host
WARN Ignoring user directive. User to be specified as a UID (numeric).
INFO Network netbox-default is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
WARN Service "postgres" won't be created because 'ports' is not specified
INFO Network netbox-default is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
WARN Service "redis" won't be created because 'ports' is not specified
INFO Network netbox-default is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
WARN Service "redis-cache" won't be created because 'ports' is not specified
INFO Network netbox-default is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
INFO Kubernetes file "kompose/netbox-deployment.yaml" created
INFO Kubernetes file "kompose/env-netbox-env-configmap.yaml" created
INFO Kubernetes file "kompose/netbox-claim0-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-claim1-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-claim2-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-media-files-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-default-networkpolicy.yaml" created
INFO Kubernetes file "kompose/netbox-housekeeping-deployment.yaml" created
INFO Kubernetes file "kompose/netbox-housekeeping-claim0-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-housekeeping-claim1-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-housekeeping-claim2-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-worker-deployment.yaml" created
INFO Kubernetes file "kompose/netbox-worker-claim0-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-worker-claim1-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/netbox-worker-claim2-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/postgres-deployment.yaml" created
INFO Kubernetes file "kompose/env-postgres-env-configmap.yaml" created
INFO Kubernetes file "kompose/netbox-postgres-data-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/redis-deployment.yaml" created
INFO Kubernetes file "kompose/env-redis-env-configmap.yaml" created
INFO Kubernetes file "kompose/netbox-redis-data-persistentvolumeclaim.yaml" created
INFO Kubernetes file "kompose/redis-cache-deployment.yaml" created
INFO Kubernetes file "kompose/env-redis-cache-env-configmap.yaml" created
INFO Kubernetes file "kompose/netbox-redis-cache-data-persistentvolumeclaim.yaml" created

# 확인
dor1@is-master:~/netbox$ cd kompose
dor1@is-master:~/netbox/kompose$ ll
total 128
drwxrwxr-x 2 dor1 dor1 4096 Feb 21 09:57 ./
drwxrwxr-x 7 dor1 dor1 4096 Feb 17 17:50 ../
-rw-r--r-- 1 dor1 dor1 1271 Feb 17 17:50 env-netbox-env-configmap.yaml
-rw-r--r-- 1 dor1 dor1  242 Feb 17 17:50 env-postgres-env-configmap.yaml
-rw-r--r-- 1 dor1 dor1  206 Feb 17 17:50 env-redis-cache-env-configmap.yaml
-rw-r--r-- 1 dor1 dor1  181 Feb 17 17:50 env-redis-env-configmap.yaml
-rw-r--r-- 1 dor1 dor1  248 Feb 17 17:50 netbox-claim0-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  248 Feb 17 17:50 netbox-claim1-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  248 Feb 17 17:50 netbox-claim2-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  325 Feb 17 17:50 netbox-default-networkpolicy.yaml
-rw-r--r-- 1 dor1 dor1 8373 Feb 17 17:50 netbox-deployment.yaml
-rw-r--r-- 1 dor1 dor1  274 Feb 17 17:50 netbox-housekeeping-claim0-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  274 Feb 17 17:50 netbox-housekeeping-claim1-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  274 Feb 17 17:50 netbox-housekeeping-claim2-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1 8624 Feb 17 17:50 netbox-housekeeping-deployment.yaml
-rw-r--r-- 1 dor1 dor1  259 Feb 17 17:50 netbox-media-files-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  263 Feb 17 17:50 netbox-postgres-data-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  269 Feb 17 17:50 netbox-redis-cache-data-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  309 Feb 17 17:50 netbox-redis-data-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  262 Feb 17 17:50 netbox-worker-claim0-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  262 Feb 17 17:50 netbox-worker-claim1-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1  262 Feb 17 17:50 netbox-worker-claim2-persistentvolumeclaim.yaml
-rw-r--r-- 1 dor1 dor1 8602 Feb 17 17:50 netbox-worker-deployment.yaml
-rw-r--r-- 1 dor1 dor1 1584 Feb 17 17:50 postgres-deployment.yaml
-rw-r--r-- 1 dor1 dor1 1358 Feb 17 17:50 redis-cache-deployment.yaml
-rw-r--r-- 1 dor1 dor1 1321 Feb 17 17:50 redis-deployment.yaml
```

변환된 파일들을 application 별로 정리하자면 다음과 같다.

```plaintext
NetBox(Application)
- netbox-default-networkpolicy.yaml
- netbox-deployment.yaml
- env-netbox-env-configmap.yaml
- netbox-claim0-persistentvolumeclaim.yaml
- netbox-claim1-persistentvolumeclaim.yaml
- netbox-claim2-persistentvolumeclaim.yaml
- netbox-media-files-persistentvolumeclaim.yaml

- netbox-housekeeping-deployment.yaml
- netbox-housekeeping-claim0-persistentvolumeclaim.yaml
- netbox-housekeeping-claim1-persistentvolumeclaim.yaml
- netbox-housekeeping-claim2-persistentvolumeclaim.yaml

- netbox-worker-deployment.yaml
- netbox-worker-claim0-persistentvolumeclaim.yaml
- netbox-worker-claim1-persistentvolumeclaim.yaml
- netbox-worker-claim2-persistentvolumeclaim.yaml

PostgreSQL(DB)
- postgres-deployment.yaml
- env-postgres-env-configmap.yaml
- netbox-postgres-data-persistentvolumeclaim.yaml

Redis(Task queue, Cache)
- redis-deployment.yaml
- redis-cache-deployment.yaml
- env-redis-env-configmap.yaml
- env-redis-cache-env-configmap.yaml
- netbox-redis-data-persistentvolumeclaim.yaml
- netbox-redis-cache-data-persistentvolumeclaim.yaml
```

## rook-ceph을 PVC로 이용하는 manifest 만들기
---

Deployment 전 서비스들에서 필요 없는 내용들을 하나하나 변경 해준다.  
전체적으로 필요 없는 정의는 다음과 같다.
- `annotations`
- `creationTimestamp`
- `status`

Service 접근을 위하여 각각의 `deployment`에서 port expose 해준다.  
PVC가 사용되는 내용은 `rook-ceph`으로 변경 해준다.  
`namespace`를 추가하여 `default`와 분리해준다.  
`configMap`에 들어가 있는 내용 중 Password가 들어가 있는 파트는 보안을 위해 `Secret`으로 변경해준다.  
`Secret`으로 바꾸는 이유는 `configMap`보다는 `Secret`에서는 base1-encoded로 한번 더 암호화 되기 때문이다.

### 1. Redis / RedisCache

#### configMap to Secret

`Redis`와 `RedisCache`의 configMap에 들어가 있는 내용은 Password가 다 이므로 내용과 파일 이름을 바꾼다.

```yaml
# env-redis-env-secret.yaml
dor1@is-master:~/netbox/kompose$ mv env-redis-env-configmap.yaml env-redis-env-secret.yaml
dor1@is-master:~/netbox/kompose$ vi env-redis-env-secret.yaml
apiVersion: v1
data:
  REDIS_PASSWORD: YW9vbmkzNjUhQA==
# ConfigMap -> Secret
kind: Secret
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: redis-env-redis-env
  name: env-redis-env
  # Add namespace
  namespace: netbox

# env-redis-cache-env-secret.yaml
dor1@is-master:~/netbox/kompose$ mv env-redis-cache-env-configmap.yaml env-redis-cache-env-secret.yaml
dor1@is-master:~/netbox/kompose$ vi env-redis-cache-env-secret.yaml
apiVersion: v1
data:
  REDIS_PASSWORD: YW9vbmkzNjUhQA==
# ConfigMap -> Secret
kind: Secret
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: redis-cache-env-redis-cache-env
  name: env-redis-cache-env
  # Add namespace
  namespace: netbox
```

#### PVC

PVC는 `rook-ceph`을 사용한다.  
용량은 넉넉하게 50Gi로 설정 한다.

```yaml
# netbox-redis-data-persistentvolumeclaim.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-redis-data-persistentvolumeclaim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-redis-data
  name: netbox-redis-data
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      # 100Mi -> 50Gi
      storage: 50Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}

# netbox-redis-cache-data-persistentvolumeclaim.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-redis-cache-data-persistentvolumeclaim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-redis-cache-data
  name: netbox-redis-cache-data
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      # 100Mi -> 50Gi
      storage: 50Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}
```

#### Deployment

`configMapkeyRef` → `secretKeyRef`  
`Redis`와 `RedisCache`의 expose port를 설정한다.

```yaml
# redis-deployment.yaml
dor1@is-master:~/netbox/kompose$ vi redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #annotations:
  #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
  #  kompose.version: 1.28.0 (c4137012e)
  #creationTimestamp: null
  labels:
    io.kompose.service: redis
  name: redis
  # Add namespace
  namespace: netbox
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: redis
  strategy:
    type: Recreate
  template:
    metadata:
      #annotations:
      #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
      #  kompose.version: 1.28.0 (c4137012e)
      #creationTimestamp: null
      labels:
        io.kompose.network/netbox-default: "true"
        io.kompose.service: redis
    spec:
      containers:
        - args:
            - sh
            - -c
            - redis-server --appendonly yes --requirepass $REDIS_PASSWORD
          env:
            - name: REDIS_PASSWORD
              valueFrom:
                # configMap -> secret
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-env
          image: redis:7-alpine
          name: redis
          # Expose redis serviceport
          ports:
          - containerPort: 6379
          resources: {}
          volumeMounts:
            - mountPath: /data
              name: netbox-redis-data
      restartPolicy: Always
      volumes:
        - name: netbox-redis-data
          persistentVolumeClaim:
            claimName: netbox-redis-data
#status: {}
```

```yaml
# redis-cache-deployment.yaml
dor1@is-master:~/netbox/kompose$ vi redis-cache-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #annotations:
  #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
  #  kompose.version: 1.28.0 (c4137012e)
  #creationTimestamp: null
  labels:
    io.kompose.service: redis-cache
  name: redis-cache
  # Add namespace
  namespace: netbox
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: redis-cache
  strategy:
    type: Recreate
  template:
    metadata:
      #annotations:
      #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
      #  kompose.version: 1.28.0 (c4137012e)
      #creationTimestamp: null
      labels:
        io.kompose.network/netbox-default: "true"
        io.kompose.service: redis-cache
    spec:
      containers:
        - args:
            - sh
            - -c
            - redis-server --requirepass $REDIS_PASSWORD
          env:
            - name: REDIS_PASSWORD
              valueFrom:
                # configMap -> secret
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-cache-env
          image: redis:7-alpine
          name: redis-cache
          # Expose redis-cache serviceport
          ports:
          - containerPort: 6379
          resources: {}
          volumeMounts:
            - mountPath: /data
              name: netbox-redis-cache-data
      restartPolicy: Always
      volumes:
        - name: netbox-redis-cache-data
          persistentVolumeClaim:
            claimName: netbox-redis-cache-data
#status: {}
```

#### Redis / RedisCache Deployment

설정이 완료 되었으면 이후 `Redis`에 대하여 배포를 해준다.

```shell
# namespace 생성
dor1@is-master:~/netbox/kompose$ kubectl create ns netbox
namespace/netbox created

# configMap
dor1@is-master:~/netbox/kompose$ kubectl apply -f env-redis-env-secret.yaml -f env-redis-cache-env-secret.yaml
secret/env-redis-env created
secret/env-redis-cache-env created

# PVC
dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-redis-data-persistentvolumeclaim.yaml -f netbox-redis-cache-data-persistentvolumeclaim.yaml
persistentvolumeclaim/netbox-redis-data created
persistentvolumeclaim/netbox-redis-cache-data created

# Deployment
dor1@is-master:~/netbox/kompose$ kubectl apply -f redis-deployment.yaml -f redis-cache-deployment.yaml
deployment.apps/redis created
deployment.apps/redis-cache created

# 확인
dor1@is-master:~/netbox/kompose$ kubectl get cm,secret,pvc,pod,deployments -n netbox
NAME                         DATA   AGE
configmap/kube-root-ca.crt   1      25h

NAME                         TYPE     DATA   AGE
secret/env-redis-cache-env   Opaque   1      4m33s
secret/env-redis-env         Opaque   1      4m33s

NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/netbox-redis-cache-data   Bound    pvc-122031da-f54a-4aac-b448-72a4a841781c   50Gi       RWO            rook-cephfs    4m33s
persistentvolumeclaim/netbox-redis-data         Bound    pvc-5ce73c26-bb69-493b-88d0-ee05044a4716   50Gi       RWO            rook-cephfs    4m33s

NAME                             READY   STATUS    RESTARTS   AGE
pod/redis-86677c4cf5-tvtdf       1/1     Running   0          4m33s
pod/redis-cache-9d9b9c48-vgh7r   1/1     Running   0          4m33s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis         1/1     1            1           4m33s
deployment.apps/redis-cache   1/1     1            1           4m33s
```

### 2. PostgreSQL

#### configMap to Secret

`PostgreSQL` 또한 configMap에 들어가 있는 내용은 Password가 다 이므로 내용과 파일 이름을 바꾼다.

```yaml
# env-postgres-env-secret.yaml
dor1@is-master:~/netbox/kompose$ mv env-postgres-env-configmap.yaml env-postgres-env-secret.yaml
dor1@is-master:~/netbox/kompose$ vi env-postgres-env-secret.yaml
apiVersion: v1
data:
  POSTGRES_DB: bmV0Ym94
  POSTGRES_PASSWORD: YW9vbmkzNjUhQA==
  POSTGRES_USER: bmV0Ym94
# ConfigMap -> Secret
kind: Secret
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: postgres-env-postgres-env
  name: env-postgres-env
  # Add namespace
  namespace: netbox
```

#### PVC

PVC는 `rook-ceph`을 사용한다.  
용량은 DB이다 보니 500Gi로 크게 설정한다.

```yaml
# netbox-postgres-data-persistentvolumeclaim.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-postgres-data-persistentvolumeclaim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-postgres-data
  name: netbox-postgres-data
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      # 100Mi -> 500Gi
      storage: 500Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}
```

#### Deployment

`configMapkeyRef` → `secretKeyRef`  
`PostgreSQL`의 expose port를 설정한다.

```yaml
# postgres-deployment.yaml
dor1@is-master:~/netbox/kompose$ vi postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #annotations:
  #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
  #  kompose.version: 1.28.0 (c4137012e)
  #creationTimestamp: null
  labels:
    io.kompose.service: postgres
  name: postgres
  # Add namespace
  namespace: netbox
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: postgres
  strategy:
    type: Recreate
  template:
    metadata:
      #annotations:
      #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
      #  kompose.version: 1.28.0 (c4137012e)
      #creationTimestamp: null
      labels:
        io.kompose.network/netbox-default: "true"
        io.kompose.service: postgres
    spec:
      containers:
        - env:
            - name: POSTGRES_DB
              valueFrom:
                # configMap -> secret
                secretKeyRef:
                  key: POSTGRES_DB
                  name: env-postgres-env
            - name: POSTGRES_PASSWORD
              valueFrom:
                # configMap -> secret
                secretKeyRef:
                  key: POSTGRES_PASSWORD
                  name: env-postgres-env
            - name: POSTGRES_USER
              valueFrom:
                 # configMap -> secret
                secretKeyRef:
                  key: POSTGRES_USER
                  name: env-postgres-env
          image: postgres:15-alpine
          name: postgres
          # Expose postgresql serviceport
          ports:
          - containerPort: 5432
          resources: {}
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: netbox-postgres-data
      restartPolicy: Always
      volumes:
        - name: netbox-postgres-data
          persistentVolumeClaim:
            claimName: netbox-postgres-data
#status: {}
```

#### PostgreSQL Deployment

설정이 완료 되었으면 이후 `PostgreSQL`에 대하여 배포를 해준다.

```shell
# configMap
dor1@is-master:~/netbox/kompose$ kubectl apply -f env-postgres-env-secret.yaml
secret/env-postgres-env created

# PVC
dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-postgres-data-persistentvolumeclaim.yaml
persistentvolumeclaim/netbox-postgres-data created

# Deployment
dor1@is-master:~/netbox/kompose$ kubectl apply -f postgres-deployment.yaml
deployment.apps/postgres created

# 확인
dor1@is-master:~/netbox/kompose$ kubectl get cm,secret,pvc,pod,deployments -n netbox
NAME                         DATA   AGE
configmap/kube-root-ca.crt   1      26h

NAME                         TYPE     DATA   AGE
secret/env-postgres-env      Opaque   3      4m8s
secret/env-redis-cache-env   Opaque   1      20m
secret/env-redis-env         Opaque   1      20m

NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/netbox-postgres-data      Bound    pvc-c9ab32be-8ac8-48f1-bf5e-d99901938edb   500Gi      RWO            rook-cephfs    6m17s
persistentvolumeclaim/netbox-redis-cache-data   Bound    pvc-122031da-f54a-4aac-b448-72a4a841781c   50Gi       RWO            rook-cephfs    20m
persistentvolumeclaim/netbox-redis-data         Bound    pvc-5ce73c26-bb69-493b-88d0-ee05044a4716   50Gi       RWO            rook-cephfs    20m

NAME                             READY   STATUS    RESTARTS   AGE
pod/postgres-856f47bdbf-bts8j    1/1     Running   0          6m17s
pod/redis-86677c4cf5-tvtdf       1/1     Running   0          20m
pod/redis-cache-9d9b9c48-vgh7r   1/1     Running   0          20m

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres      1/1     1            1           6m17s
deployment.apps/redis         1/1     1            1           20m
deployment.apps/redis-cache   1/1     1            1           20m
```

### 3. NetBox

`netbox`와 `netbox-worker`는 하나의 패키지 형태로 같이 작업 해준다.

#### configMap & Secret

`configMap`과 `Secret`은 관리를 위해 분리한다.  
POD 간 통신을 위해 expose 되어있는 port 부분도 변수로 설정한다.

```yaml
# env-netbox-env-configmap.yaml
dor1@is-master:~/netbox/kompose$ vi env-netbox-env-configmap.yaml
apiVersion: v1
data:
  CORS_ORIGIN_ALLOW_ALL: "True"
  DB_HOST: postgres
  # Add postgresql serviceport
  DB_PORT: "5432"
  DB_NAME: netbox
  DB_USER: netbox
  # Unnecessary
  #DB_PASSWORD:
  EMAIL_FROM: netbox@bar.com
  # Secret
  #EMAIL_PASSWORD:
  EMAIL_PORT: "25"
  EMAIL_SERVER: localhost
  EMAIL_SSL_CERTFILE: ""
  EMAIL_SSL_KEYFILE: ""
  EMAIL_TIMEOUT: "5"
  EMAIL_USE_SSL: "false"
  EMAIL_USE_TLS: "false"
  EMAIL_USERNAME: netbox
  GRAPHQL_ENABLED: "true"
  HOUSEKEEPING_INTERVAL: "86400"
  MEDIA_ROOT: /opt/netbox/netbox/media
  METRICS_ENABLED: "false"
  REDIS_CACHE_DATABASE: "1"
  REDIS_CACHE_HOST: redis-cache
  # Add redis-cache serviceport
  REDIS_CACHE_PORT: "6379"
  REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY: "false"
  REDIS_CACHE_SSL: "false"
  REDIS_DATABASE: "0"
  REDIS_HOST: redis
  # Add redis serviceport
  REDIS_PORT: "6379"
  REDIS_INSECURE_SKIP_TLS_VERIFY: "false"
  REDIS_SSL: "false"
  RELEASE_CHECK_URL: https://api.github.com/repos/netbox-community/netbox/releases
  # Secret
  #SECRET_KEY:
  SKIP_SUPERUSER: "false"
  # Secret
  #SUPERUSER_API_TOKEN:
  SUPERUSER_EMAIL: admin@example.com
  SUPERUSER_NAME: admin
  # Secret
  #SUPERUSER_PASSWORD:
  WEBHOOKS_ENABLED: "true"
  # Add for TZ
  TIME_ZONE: "Asia/Seoul"
kind: ConfigMap
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-env-netbox-env
  name: env-netbox-env
  # Add namespace
  namespace: netbox
```

```yaml
# NEW [env-netbox-env-secret.yaml]
dor1@is-master:~/netbox/kompose$ vi env-netbox-env-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  labels:
    io.kompose.service: netbox-env-netbox-env
  name: env-netbox-env
  namespace: netbox
data:
  EMAIL_PASSWORD: IA==
  SECRET_KEY: cjhPd0R6bmohIWRjaSNQOWdobVJmZHUxWXN4bTBBaVBlRENRaEtFK05fckNsZldOag==
  SUPERUSER_API_TOKEN: MDEyMzQ1Njc4OWFiY2RlZjAxMjM0NTY3ODlhYmNkZWYwMTIzNDU2Nw==
  SUPERUSER_PASSWORD: YWRtaW4=
```

#### networkPolicy

`networkPolicy`는 `policyType`을 따라 기본적으로 `egress`와 `ingress`에만 적용되기 때문에 `Labels`에 적용된 대상만 접근이 가능하다.  
Cloud와 같이 public망을 사용하지 않는다면, on-premises(legacy) 환경에서 NAT처리로 인하여 사용 할 일이 없기 때문에`NetworkPolicy`는 지워줘도 된다.

```shell
# Delete networkPolicy yaml
dor1@is-master:~/netbox/kompose$ rm netbox-default-networkpolicy.yaml
```

#### PVC

`docker-compose`에서 변환한 `PVC`가 3개로 되어있는데, `Deployment`에서 보게 되면 `volumeMount` 항목으로 `ReadOnly` 되어있다.  
PVC의 항목들은 하나의 파일로 합친 후 `netbox-worker`와 `netbox-housekeeping` 같이 사용하면 config가 공유된다. (claim에 대한 volumeMount 항목이 같음)  
`netbox-media-files-persistentvolumeclaim.yaml`은 media 파일 특성 상 별도로 관리한다.  
PVC는 `rook-ceph`을 사용한다.
`netbox`의 용량은 설정 값을 고려하여 50Gi로 설정한다.  
`netbox-media-files`는 media 파일 특성 상 500Gi 이상 크게 잡는다.  
`rook-ceph`으로 바꾸는 과정에서 `rook-ceph.cephfs.CSI` 드라이버는 ROX(ReadOnlyMany)를 지원하지 않기에 RWX(ReadWriteMany)로 변경한다. (`netbox-media-files`는 공통적으로 사용하기에 RXO 상태로 유지)  
<https://rook.io/docs/rook/latest/Getting-Started/quickstart/#storage>

```yaml
# Merge contents
dor1@is-master:~/netbox/kompose$ cat netbox-claim[0-2]-persistentvolumeclaim.yaml > netbox-persistentvolumeclaim.yaml
dor1@is-master:~/netbox/kompose$ rm netbox-claim[0-2]-persistentvolumeclaim.yaml netbox-worker-claim[0-2]-persistentvolumeclaim.yaml

# netbox-persistentvolumeclaim.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-persistentvolumeclaim.yaml
# [netbox-claim0-persistentvolumeclaim.yaml]
# for /etc/netbox/config
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-claim0
  name: netbox-claim0
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    # ROX -> RWX
    - ReadWriteMany
  resources:
    requests:
      # 100Mi -> 50Gi
      storage: 50Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}

# Add divider [netbox-claim1-persistentvolumeclaim.yaml]
# for /etc/netbox/reports
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-claim1
  name: netbox-claim1
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    # ROX -> RWX
    - ReadWriteMany
  resources:
    requests:
      # 100Mi -> 50Gi
      storage: 50Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}

# Add divider [netbox-claim2-persistentvolumeclaim.yaml]
# for /etc/netbox/scripts
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-claim2
  name: netbox-claim2
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    # ROX -> RWX
    - ReadWriteMany
  resources:
    requests:
      # 100Mi -> 50Gi
      storage: 50Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}
```

```yaml
# netbox-media-files-persistentvolumeclaim.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-media-files-persistentvolumeclaim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-media-files
  name: netbox-media-files
  # Add namespace
  namespace: netbox
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      # 100Mi -> 500Gi
      storage: 500Gi
  # Add storageClass
  storageClassName: rook-cephfs
#status: {}
```

#### Deployment

PVC의 통합을 진행하기 때문에 `netbox-worker`의 PVC는 `netbox-claim`으로 변경한다.  
`NetBox` 항목은 접근을 위하여 expose port 설정한다.  
이미지의 버전을 고정한다.  
`configMapkeyRef` → `secretKeyRef` (Secret 분리 사항만)  
`Redis` 및 `PostgreSQL`에 대한 항목은 expose 되어있는 port가 있으므로 접근을 위해 항목 추가한다.  
k8s 특성 상 `liveness`(docker의 helathcheck와 유사한 기능)은 새로 path를 설정 해줘야 하며, 필요 없을 시에는 사용할 필요가 없다. (기본 이미지로 진행하게 되면 기본적으로 설정 되어있어`Events`의 항목에 error가 지속적으로 발생함)

```yaml
# netbox-deployment.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #annotations:
  #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
  #  kompose.version: 1.28.0 (c4137012e)
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox
  name: netbox
  # Add namespace
  namespace: netbox
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: netbox
  strategy:
    type: Recreate
  template:
    metadata:
      #annotations:
      #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
      #  kompose.version: 1.28.0 (c4137012e)
      #creationTimestamp: null
      labels:
        io.kompose.network/netbox-default: "true"
        io.kompose.service: netbox
    spec:
      containers:
        - env:
            - name: CORS_ORIGIN_ALLOW_ALL
              valueFrom:
                configMapKeyRef:
                  key: CORS_ORIGIN_ALLOW_ALL
                  name: env-netbox-env
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  key: DB_HOST
                  name: env-netbox-env
            # Added for DB_PORT
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  key: DB_PORT
                  name: env-netbox-env
            - name: DB_NAME
              valueFrom:
                configMapKeyRef:
                  key: DB_NAME
                  name: env-netbox-env
            - name: DB_USER
              valueFrom:
                configMapKeyRef:
                  key: DB_USER
                  name: env-netbox-env
            # configMap -> secret
            # Used env-postgres-env
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: POSTGRES_PASSWORD
                  name: env-postgres-env
            - name: EMAIL_FROM
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_FROM
                  name: env-netbox-env
            # configMap -> secret
            - name: EMAIL_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: EMAIL_PASSWORD
                  name: env-netbox-env
            - name: EMAIL_PORT
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_PORT
                  name: env-netbox-env
            - name: EMAIL_SERVER
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SERVER
                  name: env-netbox-env
            - name: EMAIL_SSL_CERTFILE
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SSL_CERTFILE
                  name: env-netbox-env
            - name: EMAIL_SSL_KEYFILE
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SSL_KEYFILE
                  name: env-netbox-env
            - name: EMAIL_TIMEOUT
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_TIMEOUT
                  name: env-netbox-env
            - name: EMAIL_USERNAME
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USERNAME
                  name: env-netbox-env
            - name: EMAIL_USE_SSL
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USE_SSL
                  name: env-netbox-env
            - name: EMAIL_USE_TLS
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USE_TLS
                  name: env-netbox-env
            - name: GRAPHQL_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: GRAPHQL_ENABLED
                  name: env-netbox-env
            - name: HOUSEKEEPING_INTERVAL
              valueFrom:
                configMapKeyRef:
                  key: HOUSEKEEPING_INTERVAL
                  name: env-netbox-env
            - name: MEDIA_ROOT
              valueFrom:
                configMapKeyRef:
                  key: MEDIA_ROOT
                  name: env-netbox-env
            - name: METRICS_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: METRICS_ENABLED
                  name: env-netbox-env
            - name: REDIS_CACHE_DATABASE
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_DATABASE
                  name: env-netbox-env
            - name: REDIS_CACHE_HOST
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_HOST
                  name: env-netbox-env
            # Added for REDIS_CACHE_PORT
            - name: REDIS_CACHE_PORT
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_PORT
                  name: env-netbox-env
            - name: REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY
                  name: env-netbox-env
            # configMap -> secret
            # REDIS_CACHE_PASSWROD -> REDIS_PASSWORD
            # Used env-redis-cache-env
            - name: REDIS_CACHE_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-cache-env
            - name: REDIS_CACHE_SSL
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_SSL
                  name: env-netbox-env
            - name: REDIS_DATABASE
              valueFrom:
                configMapKeyRef:
                  key: REDIS_DATABASE
                  name: env-netbox-env
            - name: REDIS_HOST
              valueFrom:
                configMapKeyRef:
                  key: REDIS_HOST
                  name: env-netbox-env
            # Added for REDIS_PORT
            - name: REDIS_PORT
              valueFrom:
                configMapKeyRef:
                  key: REDIS_PORT
                  name: env-netbox-env
            - name: REDIS_INSECURE_SKIP_TLS_VERIFY
              valueFrom:
                configMapKeyRef:
                  key: REDIS_INSECURE_SKIP_TLS_VERIFY
                  name: env-netbox-env
            # configMap -> secret
            # Used env-redis-env
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-env
            - name: REDIS_SSL
              valueFrom:
                configMapKeyRef:
                  key: REDIS_SSL
                  name: env-netbox-env
            - name: RELEASE_CHECK_URL
              valueFrom:
                configMapKeyRef:
                  key: RELEASE_CHECK_URL
                  name: env-netbox-env
            # configMap -> secret
            - name: SECRET_KEY
              valueFrom:
                secretKeyRef:
                  key: SECRET_KEY
                  name: env-netbox-env
            - name: SKIP_SUPERUSER
              valueFrom:
                configMapKeyRef:
                  key: SKIP_SUPERUSER
                  name: env-netbox-env
            # configMap -> secret
            - name: SUPERUSER_API_TOKEN
              valueFrom:
                secretKeyRef:
                  key: SUPERUSER_API_TOKEN
                  name: env-netbox-env
            - name: SUPERUSER_EMAIL
              valueFrom:
                configMapKeyRef:
                  key: SUPERUSER_EMAIL
                  name: env-netbox-env
            - name: SUPERUSER_NAME
              valueFrom:
                configMapKeyRef:
                  key: SUPERUSER_NAME
                  name: env-netbox-env
            # configMap -> secret
            - name: SUPERUSER_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: SUPERUSER_PASSWORD
                  name: env-netbox-env
            - name: WEBHOOKS_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: WEBHOOKS_ENABLED
                  name: env-netbox-env
            # Added for TZ
            - name: TIME_ZONE
              valueFrom:
                configMapKeyRef:
                  key: TIME_ZONE
                  name: env-netbox-env
          # Fixed version
          image: netboxcommunity/netbox:v3.4-2.4.0
          #livenessProbe:
          #  exec:
          #    command:
          #      - curl -f http://localhost:8080/api/ || exit 1
          #  initialDelaySeconds: 60
          #  periodSeconds: 15
          #  timeoutSeconds: 3
          name: netbox
          # Expose netbox serviceport
          ports:
          - containerPort: 8080
          resources: {}
          volumeMounts:
            - mountPath: /etc/netbox/config
              name: netbox-claim0
              readOnly: true
            - mountPath: /etc/netbox/reports
              name: netbox-claim1
              readOnly: true
            - mountPath: /etc/netbox/scripts
              name: netbox-claim2
              readOnly: true
            - mountPath: /opt/netbox/netbox/media
              name: netbox-media-files
      restartPolicy: Always
      volumes:
        - name: netbox-claim0
          persistentVolumeClaim:
            claimName: netbox-claim0
            readOnly: true
        - name: netbox-claim1
          persistentVolumeClaim:
            claimName: netbox-claim1
            readOnly: true
        - name: netbox-claim2
          persistentVolumeClaim:
            claimName: netbox-claim2
            readOnly: true
        - name: netbox-media-files
          persistentVolumeClaim:
            claimName: netbox-media-files
#status: {}
```

```yaml
# netbox-worker-deployment.yaml
dor1@is-master:~/netbox/kompose$ vi netbox-worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #annotations:
  #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
  #  kompose.version: 1.28.0 (c4137012e)
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-worker
  name: netbox-worker
  # Add namespace
  namespace: netbox
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: netbox-worker
  strategy:
    type: Recreate
  template:
    metadata:
      #annotations:
      #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
      #  kompose.version: 1.28.0 (c4137012e)
      #creationTimestamp: null
      labels:
        io.kompose.network/netbox-default: "true"
        io.kompose.service: netbox-worker
    spec:
      containers:
        - args:
            - /opt/netbox/venv/bin/python
            - /opt/netbox/netbox/manage.py
            - rqworker
          env:
            - name: CORS_ORIGIN_ALLOW_ALL
              valueFrom:
                configMapKeyRef:
                  key: CORS_ORIGIN_ALLOW_ALL
                  name: env-netbox-env
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  key: DB_HOST
                  name: env-netbox-env
            # Added for DB_PORT
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  key: DB_PORT
                  name: env-netbox-env
            - name: DB_NAME
              valueFrom:
                configMapKeyRef:
                  key: DB_NAME
                  name: env-netbox-env
            - name: DB_USER
              valueFrom:
                configMapKeyRef:
                  key: DB_USER
                  name: env-netbox-env
            # configMap -> secret
            # Used env-postgres-env
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: POSTGRES_PASSWORD
                  name: env-postgres-env
            - name: EMAIL_FROM
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_FROM
                  name: env-netbox-env
            # configMap -> secret
            - name: EMAIL_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: EMAIL_PASSWORD
                  name: env-netbox-env
            - name: EMAIL_PORT
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_PORT
                  name: env-netbox-env
            - name: EMAIL_SERVER
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SERVER
                  name: env-netbox-env
            - name: EMAIL_SSL_CERTFILE
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SSL_CERTFILE
                  name: env-netbox-env
            - name: EMAIL_SSL_KEYFILE
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SSL_KEYFILE
                  name: env-netbox-env
            - name: EMAIL_TIMEOUT
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_TIMEOUT
                  name: env-netbox-env
            - name: EMAIL_USERNAME
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USERNAME
                  name: env-netbox-env
            - name: EMAIL_USE_SSL
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USE_SSL
                  name: env-netbox-env
            - name: EMAIL_USE_TLS
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USE_TLS
                  name: env-netbox-env
            - name: GRAPHQL_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: GRAPHQL_ENABLED
                  name: env-netbox-env
            - name: HOUSEKEEPING_INTERVAL
              valueFrom:
                configMapKeyRef:
                  key: HOUSEKEEPING_INTERVAL
                  name: env-netbox-env
            - name: MEDIA_ROOT
              valueFrom:
                configMapKeyRef:
                  key: MEDIA_ROOT
                  name: env-netbox-env
            - name: METRICS_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: METRICS_ENABLED
                  name: env-netbox-env
            - name: REDIS_CACHE_DATABASE
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_DATABASE
                  name: env-netbox-env
            - name: REDIS_CACHE_HOST
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_HOST
                  name: env-netbox-env
            # Added for REDIS_CACHE_PORT
            - name: REDIS_CACHE_PORT
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_PORT
                  name: env-netbox-env
            - name: REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY
                  name: env-netbox-env
            # configMap -> secret
            # REDIS_CACHE_PASSWROD -> REDIS_PASSWORD
            # Used env-redis-cache-env
            - name: REDIS_CACHE_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-cache-env
            - name: REDIS_CACHE_SSL
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_SSL
                  name: env-netbox-env
            - name: REDIS_DATABASE
              valueFrom:
                configMapKeyRef:
                  key: REDIS_DATABASE
                  name: env-netbox-env
            - name: REDIS_HOST
              valueFrom:
                configMapKeyRef:
                  key: REDIS_HOST
                  name: env-netbox-env
            # Added for REDIS_PORT
            - name: REDIS_PORT
              valueFrom:
                configMapKeyRef:
                  key: REDIS_PORT
                  name: env-netbox-env
            - name: REDIS_INSECURE_SKIP_TLS_VERIFY
              valueFrom:
                configMapKeyRef:
                  key: REDIS_INSECURE_SKIP_TLS_VERIFY
                  name: env-netbox-env
            # configMap -> secret
            # Used env-redis-env
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-env
            - name: REDIS_SSL
              valueFrom:
                configMapKeyRef:
                  key: REDIS_SSL
                  name: env-netbox-env
            - name: RELEASE_CHECK_URL
              valueFrom:
                configMapKeyRef:
                  key: RELEASE_CHECK_URL
                  name: env-netbox-env
            # configMap -> secret
            - name: SECRET_KEY
              valueFrom:
                secretKeyRef:
                  key: SECRET_KEY
                  name: env-netbox-env
            - name: SKIP_SUPERUSER
              valueFrom:
                configMapKeyRef:
                  key: SKIP_SUPERUSER
                  name: env-netbox-env
            # configMap -> secret
            - name: SUPERUSER_API_TOKEN
              valueFrom:
                secretKeyRef:
                  key: SUPERUSER_API_TOKEN
                  name: env-netbox-env
            - name: SUPERUSER_EMAIL
              valueFrom:
                configMapKeyRef:
                  key: SUPERUSER_EMAIL
                  name: env-netbox-env
            - name: SUPERUSER_NAME
              valueFrom:
                configMapKeyRef:
                  key: SUPERUSER_NAME
                  name: env-netbox-env
            # configMap -> secret
            - name: SUPERUSER_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: SUPERUSER_PASSWORD
                  name: env-netbox-env
            - name: WEBHOOKS_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: WEBHOOKS_ENABLED
                  name: env-netbox-env
            # Added for TZ
            - name: TIME_ZONE
              valueFrom:
                configMapKeyRef:
                  key: TIME_ZONE
                  name: env-netbox-env
          # Fixed version
          image: netboxcommunity/netbox:v3.4-2.4.0
          #livenessProbe:
          #  exec:
          #    command:
          #      - ps -aux | grep -v grep | grep -q rqworker || exit 1
          #  initialDelaySeconds: 20
          #  periodSeconds: 15
          #  timeoutSeconds: 3
          name: netbox-worker
          resources: {}
          volumeMounts:
            - mountPath: /etc/netbox/config
              name: netbox-worker-claim0
              readOnly: true
            - mountPath: /etc/netbox/reports
              name: netbox-worker-claim1
              readOnly: true
            - mountPath: /etc/netbox/scripts
              name: netbox-worker-claim2
              readOnly: true
            - mountPath: /opt/netbox/netbox/media
              name: netbox-media-files
      restartPolicy: Always
      volumes:
        - name: netbox-worker-claim0
          persistentVolumeClaim:
            # Integrated netbox-claim
            claimName: netbox-claim0
            readOnly: true
        - name: netbox-worker-claim1
          persistentVolumeClaim:
            # Integrated netbox-claim
            claimName: netbox-claim1
            readOnly: true
        - name: netbox-worker-claim2
          persistentVolumeClaim:
            # Integrated netbox-claim
            claimName: netbox-claim2
            readOnly: true
        - name: netbox-media-files
          persistentVolumeClaim:
            claimName: netbox-media-files
#status: {}
```

#### POD

PVC 특성 상 권한의 문제도 있고, config가 따로 없다 보니 config에 대해 initializing이 필요하여 `busybox` 이미지를 통해 config를 옮겨주는 작업이 필요하다.

```yaml
# New [busybox.yaml]
dor1@is-master:~/netbox/kompose$ vi busybox.yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: netbox
spec:
  containers:
    - name: config
      image: busybox:latest
      command: ["sleep", "3600" ]
      volumeMounts:
        - mountPath: /mnt
          name: netbox-claim0
    - name: reports
      image: busybox:latest
      command: ["sleep", "3600" ]
      volumeMounts:
        - mountPath: /mnt
          name: netbox-claim1
    - name: scripts
      image: busybox:latest
      command: ["sleep", "3600" ]
      volumeMounts:
        - mountPath: /mnt
          name: netbox-claim2
  volumes:
    - name: netbox-claim0
      persistentVolumeClaim:
        claimName: netbox-claim0
    - name: netbox-claim1
      persistentVolumeClaim:
        claimName: netbox-claim1
    - name: netbox-claim2
      persistentVolumeClaim:
        claimName: netbox-claim2
```

설정이 완료되었으면, config 값에 대해 `busybox`의 이미지를 이용하여 옮겨준다.

```shell
# configMap
dor1@is-master:~/netbox/kompose$ kubectl apply -f env-netbox-env-configmap.yaml
configmap/env-netbox-env created

# secret
dor1@is-master:~/netbox/kompose$ kubectl apply -f env-netbox-env-secret.yaml
secret/env-netbox-env created

# networkPolicy
dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-default-networkpolicy.yaml
networkpolicy.networking.k8s.io/netbox-default created

# PVC
dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-persistentvolumeclaim.yaml -f netbox-media-files-persistentvolumeclaim.yaml
persistentvolumeclaim/netbox-claim0 created
persistentvolumeclaim/netbox-claim1 created
persistentvolumeclaim/netbox-claim2 created
persistentvolumeclaim/netbox-media-files created

# busybox POD
dor1@is-master:~/netbox/kompose$ kubectl apply -f busybox.yaml
pod/busybox created

dor1@is-master:~/netbox/kompose$ kubectl get pod -n netbox busybox
NAME      READY   STATUS    RESTARTS   AGE
busybox   3/3     Running   0          2m30s

## config 이동
### config
dor1@is-master:~/netbox/kompose$ cd .. 
dor1@is-master:~/netbox$ kubectl cp -n netbox ./configuration busybox:/mnt -c config
dor1@is-master:~/netbox$ kubectl exec -n netbox busybox -c config -- sh -c "mv /mnt/configuration/* /mnt && rm -rf /mnt/configuration"
dor1@is-master:~/netbox$ kubectl exec -n netbox busybox -c config -- sh -c "ls /mnt"
configuration.py
extra.py
ldap
logging.py
plugins.py

### reports
dor1@is-master:~/netbox$ kubectl cp -n netbox ./reports busybox:/mnt -c reports
dor1@is-master:~/netbox$ kubectl exec -n netbox busybox -c reports -- sh -c "mv /mnt/reports/* /mnt && rm -rf /mnt/reports"
dor1@is-master:~/netbox$ kubectl exec -n netbox busybox -c reports -- sh -c "ls /mnt"
devices.py.example

### scripts
dor1@is-master:~/netbox$ kubectl cp -n netbox ./scripts busybox:/mnt -c scripts
dor1@is-master:~/netbox$ kubectl exec -n netbox busybox -c scripts -- sh -c "mv /mnt/scripts/* /mnt && rm -rf /mnt/scripts"
dor1@is-master:~/netbox$ kubectl exec -n netbox busybox -c scripts -- sh -c "ls /mnt"
__init__.py

## Delete busybox POD
dor1@is-master:~/netbox$ cd kompose
dor1@is-master:~/netbox/kompose$ kubectl delete -f busybox.yaml
pod "busybox" deleted
```

`Deployment`항목은 service가 올라가야 서로 host를 찾을 수 있기 때문에 뒤쪽의 service 항목을 먼저 올린 뒤에 작업 해준다.

### 4. NetBox-Housekeeping

`netbox-housekeeping`은 다음과 같이 지원 해준다.

- 인증 세션 만료된 DB 정리
- 오래된 changelog 삭제
- 오래된 job result에 대한 정리
- NetBox release(신버전) 체크
- 
<https://github.com/netbox-community/netbox/blob/develop/docs/administration/housekeeping.md>


#### PVC

설정 값이 저장되어있는 `NetBox`에서 만든 PVC를 그대로 이용하므로 필요가 없다.

```shell
# Delete PVC yaml
dor1@is-master:~/netbox/kompose$ rm netbox-housekeeping-claim[0-2]-persistentvolumeclaim.yaml
```

#### Deployment

`NetBox`에서 만들어진 PVC를 이용하기 위하여 PVC는 `netbox-claim`으로 변경해준다.  
이미지의 버전은 fix 시킨다.  
k8s 특성 상 `liveness`(docker의 helathcheck와 유사한 기능)은 새로 path를 설정 해줘야 하며, 필요 없을 시에는 사용할 필요가 없다. (기본 이미지로 진행하게 되면 기본적으로 설정 되어있어`Events`의 항목에 error가 지속적으로 발생함)

```yaml
# netbox-housekeeping-deployment.yaml
dor1@is-master:~/netbox/kompose$ netbox-housekeeping-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  #annotations:
  #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
  #  kompose.version: 1.28.0 (c4137012e)
  #creationTimestamp: null
  labels:
    io.kompose.service: netbox-housekeeping
  name: netbox-housekeeping
  # Add namespace
  namespace: netbox
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: netbox-housekeeping
  strategy:
    type: Recreate
  template:
    metadata:
      #annotations:
      #  kompose.cmd: kompose convert -f docker-compose.yml -o kompose
      #  kompose.version: 1.28.0 (c4137012e)
      #creationTimestamp: null
      labels:
        io.kompose.network/netbox-default: "true"
        io.kompose.service: netbox-housekeeping
    spec:
      containers:
        - args:
            - /opt/netbox/housekeeping.sh
          env:
            - name: CORS_ORIGIN_ALLOW_ALL
              valueFrom:
                configMapKeyRef:
                  key: CORS_ORIGIN_ALLOW_ALL
                  name: env-netbox-env
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  key: DB_HOST
                  name: env-netbox-env
            # Added for DB_PORT
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  key: DB_PORT
                  name: env-netbox-env
            - name: DB_NAME
              valueFrom:
                configMapKeyRef:
                  key: DB_NAME
                  name: env-netbox-env
            - name: DB_USER
              valueFrom:
                configMapKeyRef:
                  key: DB_USER
                  name: env-netbox-env
            # configMap -> secret
            # Used env-postgres-env
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: POSTGRES_PASSWORD
                  name: env-postgres-env
            - name: EMAIL_FROM
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_FROM
                  name: env-netbox-env
            # configMap -> secret
            - name: EMAIL_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: EMAIL_PASSWORD
                  name: env-netbox-env
            - name: EMAIL_PORT
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_PORT
                  name: env-netbox-env
            - name: EMAIL_SERVER
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SERVER
                  name: env-netbox-env
            - name: EMAIL_SSL_CERTFILE
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SSL_CERTFILE
                  name: env-netbox-env
            - name: EMAIL_SSL_KEYFILE
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_SSL_KEYFILE
                  name: env-netbox-env
            - name: EMAIL_TIMEOUT
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_TIMEOUT
                  name: env-netbox-env
            - name: EMAIL_USERNAME
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USERNAME
                  name: env-netbox-env
            - name: EMAIL_USE_SSL
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USE_SSL
                  name: env-netbox-env
            - name: EMAIL_USE_TLS
              valueFrom:
                configMapKeyRef:
                  key: EMAIL_USE_TLS
                  name: env-netbox-env
            - name: GRAPHQL_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: GRAPHQL_ENABLED
                  name: env-netbox-env
            - name: HOUSEKEEPING_INTERVAL
              valueFrom:
                configMapKeyRef:
                  key: HOUSEKEEPING_INTERVAL
                  name: env-netbox-env
            - name: MEDIA_ROOT
              valueFrom:
                configMapKeyRef:
                  key: MEDIA_ROOT
                  name: env-netbox-env
            - name: METRICS_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: METRICS_ENABLED
                  name: env-netbox-env
            - name: REDIS_CACHE_DATABASE
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_DATABASE
                  name: env-netbox-env
            - name: REDIS_CACHE_HOST
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_HOST
                  name: env-netbox-env
            # Added for REDIS_CACHE_PORT
            - name: REDIS_CACHE_PORT
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_PORT
                  name: env-netbox-env
            - name: REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_INSECURE_SKIP_TLS_VERIFY
                  name: env-netbox-env
            # configMap -> secret
            # REDIS_CACHE_PASSWROD -> REDIS_PASSWORD
            # Used env-redis-cache-env
            - name: REDIS_CACHE_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-cache-env
            - name: REDIS_CACHE_SSL
              valueFrom:
                configMapKeyRef:
                  key: REDIS_CACHE_SSL
                  name: env-netbox-env
            - name: REDIS_DATABASE
              valueFrom:
                configMapKeyRef:
                  key: REDIS_DATABASE
                  name: env-netbox-env
            - name: REDIS_HOST
              valueFrom:
                configMapKeyRef:
                  key: REDIS_HOST
                  name: env-netbox-env
            # Added for REDIS_PORT
            - name: REDIS_PORT
              valueFrom:
                configMapKeyRef:
                  key: REDIS_PORT
                  name: env-netbox-env
            - name: REDIS_INSECURE_SKIP_TLS_VERIFY
              valueFrom:
                configMapKeyRef:
                  key: REDIS_INSECURE_SKIP_TLS_VERIFY
                  name: env-netbox-env
            # configMap -> secret
            # Used env-redis-env
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_PASSWORD
                  name: env-redis-env
            - name: REDIS_SSL
              valueFrom:
                configMapKeyRef:
                  key: REDIS_SSL
                  name: env-netbox-env
            - name: RELEASE_CHECK_URL
              valueFrom:
                configMapKeyRef:
                  key: RELEASE_CHECK_URL
                  name: env-netbox-env
            # configMap -> secret
            - name: SECRET_KEY
              valueFrom:
                secretKeyRef:
                  key: SECRET_KEY
                  name: env-netbox-env
            - name: SKIP_SUPERUSER
              valueFrom:
                configMapKeyRef:
                  key: SKIP_SUPERUSER
                  name: env-netbox-env
            # configMap -> secret
            - name: SUPERUSER_API_TOKEN
              valueFrom:
                secretKeyRef:
                  key: SUPERUSER_API_TOKEN
                  name: env-netbox-env
            - name: SUPERUSER_EMAIL
              valueFrom:
                configMapKeyRef:
                  key: SUPERUSER_EMAIL
                  name: env-netbox-env
            - name: SUPERUSER_NAME
              valueFrom:
                configMapKeyRef:
                  key: SUPERUSER_NAME
                  name: env-netbox-env
            # configMap -> secret
            - name: SUPERUSER_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: SUPERUSER_PASSWORD
                  name: env-netbox-env
            - name: WEBHOOKS_ENABLED
              valueFrom:
                configMapKeyRef:
                  key: WEBHOOKS_ENABLED
                  name: env-netbox-env
            # Added for TZ
            - name: TIME_ZONE
              valueFrom:
                configMapKeyRef:
                  key: TIME_ZONE
                  name: env-netbox-env
          # Fixed version
          image: netboxcommunity/netbox:v3.4-2.4.0
          #livenessProbe:
          #  exec:
          #    command:
          #      - ps -aux | grep -v grep | grep -q housekeeping || exit 1
          #  initialDelaySeconds: 20
          #  periodSeconds: 15
          #  timeoutSeconds: 3
          name: netbox-housekeeping
          resources: {}
          volumeMounts:
            - mountPath: /etc/netbox/config
              name: netbox-housekeeping-claim0
              readOnly: true
            - mountPath: /etc/netbox/reports
              name: netbox-housekeeping-claim1
              readOnly: true
            - mountPath: /etc/netbox/scripts
              name: netbox-housekeeping-claim2
              readOnly: true
            - mountPath: /opt/netbox/netbox/media
              name: netbox-media-files
      restartPolicy: Always
      volumes:
        - name: netbox-housekeeping-claim0
          persistentVolumeClaim:
            # Integrated netbox-claim
            claimName: netbox-claim0
            readOnly: true
        - name: netbox-housekeeping-claim1
          persistentVolumeClaim:
            # Integrated netbox-claim
            claimName: netbox-claim1
            readOnly: true
        - name: netbox-housekeeping-claim2
          persistentVolumeClaim:
            # Integrated netbox-claim
            claimName: netbox-claim2
            readOnly: true
        - name: netbox-media-files
          persistentVolumeClaim:
            claimName: netbox-media-files
#status: {}
```

`Deployment`항목은 service가 올라가야 서로 host를 찾을 수 있기 때문에 뒤쪽의 service 항목을 먼저 올린 뒤에 작업 해준다.

## rook-ceph을 이용한 NetBox Deployment
---

### 1. Service

각각의 host를 찾기 위해서 service 항목을 새로 만들어준다.

```yaml
dor1@is-master:~/netbox/kompose$ vi netbox-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: netbox
spec:
  selector:
    io.kompose.service: redis
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis-cache
  namespace: netbox
spec:
  selector:
    io.kompose.service: redis-cache
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: netbox
spec:
  selector:
    io.kompose.service: postgres
  ports:
  - protocol: TCP
    port: 5432
    targetPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: netbox
  namespace: netbox
spec:
  ports:
  - name: http
    nodePort: 30200
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    io.kompose.service: netbox
  type: NodePort
```


### 2. NetBox Deployment

```shell
dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-service.yaml
service/redis created
service/redis-cache created
service/postgres created
service/netbox created

dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-worker-deployment.yaml
deployment.apps/netbox-worker created

dor1@is-master:~/netbox/kompose$ kubectl get pod -n netbox
NAME                                  READY   STATUS    RESTARTS   AGE
netbox-worker-744d87586c-qlkpd        1/1     Running   0          22m
postgres-964c6f654-8c75l              1/1     Running   0          35m
redis-6dbdf5685f-7gc72                1/1     Running   0          35m
redis-cache-79864b7b6-5th8v           1/1     Running   0          35m

dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-deployment.yaml
deployment.apps/netbox created

dor1@is-master:~/netbox/kompose$ kubectl get pod -n netbox
NAME                                  READY   STATUS    RESTARTS   AGE
netbox-59895cb669-znmpl               1/1     Running   0          12m
netbox-worker-744d87586c-qlkpd        1/1     Running   0          34m
postgres-964c6f654-8c75l              1/1     Running   0          47m
redis-6dbdf5685f-7gc72                1/1     Running   0          47m
redis-cache-79864b7b6-5th8v           1/1     Running   0          47m

dor1@is-master:~/netbox/kompose$ kubectl apply -f netbox-housekeeping-deployment.yaml
deployment.apps/netbox-housekeeping created

dor1@is-master:~/netbox/kompose$ kubectl get pod -n netbox
NAME                                  READY   STATUS    RESTARTS   AGE
netbox-59895cb669-znmpl               1/1     Running   0          22m
netbox-housekeeping-74d8d75d9-lzmnm   1/1     Running   0          10m
netbox-worker-744d87586c-qlkpd        1/1     Running   0          44m
postgres-964c6f654-8c75l              1/1     Running   0          57m
redis-6dbdf5685f-7gc72                1/1     Running   0          57m
redis-cache-79864b7b6-5th8v           1/1     Running   0          57m

dor1@is-master:~/netbox/kompose$ kubectl get cm,secret,pvc,pod,deployments -n netbox
NAME                         DATA   AGE
configmap/env-netbox-env     33     74m
configmap/kube-root-ca.crt   1      2d22h

NAME                         TYPE     DATA   AGE
secret/env-netbox-env        Opaque   4      74m
secret/env-postgres-env      Opaque   5      3h19m
secret/env-redis-cache-env   Opaque   1      76m
secret/env-redis-env         Opaque   1      76m

NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/netbox-claim0             Bound    pvc-918aaf9f-db84-435e-8e41-fb0e7e53cebd   50Gi       RWX            rook-cephfs    74m
persistentvolumeclaim/netbox-claim1             Bound    pvc-4d8b8b08-03e3-4047-92ea-4eb7ef9a3738   50Gi       RWX            rook-cephfs    74m
persistentvolumeclaim/netbox-claim2             Bound    pvc-767ad8f6-ca7f-4a9d-b44f-863e4a9f70b9   50Gi       RWX            rook-cephfs    74m
persistentvolumeclaim/netbox-media-files        Bound    pvc-5e17f6aa-a6b7-4d5f-bec6-3f345bd6a4b7   500Gi      RWO            rook-cephfs    74m
persistentvolumeclaim/netbox-postgres-data      Bound    pvc-ecce50a4-a54c-48e3-ad88-54b5fefeecfd   500Gi      RWO            rook-cephfs    76m
persistentvolumeclaim/netbox-redis-cache-data   Bound    pvc-8c4bb301-5b88-48b3-9eb2-3f7c51629481   50Gi       RWO            rook-cephfs    76m
persistentvolumeclaim/netbox-redis-data         Bound    pvc-e2d0560d-d9e6-4eda-a3c8-3541497f7a72   50Gi       RWO            rook-cephfs    76m

NAME                                      READY   STATUS    RESTARTS   AGE
pod/netbox-59895cb669-znmpl               1/1     Running   0          52m
pod/netbox-housekeeping-74d8d75d9-lzmnm   1/1     Running   0          20m
pod/netbox-worker-744d87586c-qlkpd        1/1     Running   0          53m
pod/postgres-964c6f654-8c75l              1/1     Running   0          76m
pod/redis-6dbdf5685f-7gc72                1/1     Running   0          76m
pod/redis-cache-79864b7b6-5th8v           1/1     Running   0          76m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/netbox                1/1     1            1           52m
deployment.apps/netbox-housekeeping   1/1     1            1           20m
deployment.apps/netbox-worker         1/1     1            1           53m
deployment.apps/postgres              1/1     1            1           76m
deployment.apps/redis                 1/1     1            1           76m
deployment.apps/redis-cache           1/1     1            1           76m
```

Deployment가 다 끝나면 접속해보면 정상적으로 접속이 된다.

![NetBox 초기 화면](assets/img/post/cluster/kubernetes/2023-02-21-kubernetes-deploy_netbox_with_kompose/1.png)
_NetBox 초기 화면_

초기 계정은 `admin`/`admin` 이다.

## 추가 사항
---

### 1. Secret 사용 시 필수 사항

`Secret` 사용 시에는 기본적으로 base64의 암호화 되어있는 값(vaule)만 들어가야 한다.  
즉, 다음과 같이 base64로 encoding 하여서 입력한다.  
추가로 line break(\\n) 항목이 들어가는 것을 피하기 위하여 반드시 `-n` 옵션을 넣어서 입력한다.

```shell
dor1@is-master:~/netbox/kompose$ echo -n 'admin' | base64
YWRtaW4=
```