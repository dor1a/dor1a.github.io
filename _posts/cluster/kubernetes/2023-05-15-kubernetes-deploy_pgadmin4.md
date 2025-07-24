---
title: Kubernetes - pgadmin4 배포해보기
date: 2023-05-15 16:12:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, pgadmin4]
description: Kubernetes에서 Postgres DB를 GUI로 관리할 수 있는 pgadmin4를 배포해보았다.
---

>Kubernetes v1.24.7 / pgadmin4 v6 / Ubuntu 22.04 LTS
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `postgres`를 통합 관리 하려고 진행하게 되었다.
* [postgresql-ha](/posts/kubernetes-deploy_postgres-operator) 구축 후 `pgadmin4`에서 연결하는 것을 해보았다.
* `Kubernetes`를 공부하기 위하여 `docker`의 image를 `kompose` 없이 manifest 형식으로 변환 해 보았다.  
  실제로 해보면서 yaml파일 작성에 크게 도움이 되었다.
* GUI라는 것은 데이터의 중요성보다 보기만 하면 되는 것이다 보니 단일 `replicas`로 진행하였다.
* 또한, 관리의 편리성을 위해 `namespace`는 default로 고정하였다.

## Prepare
---

큰 부류로 많이 사용 되는 deployment, service, pvc, secrets으로 나눠봤다. (configMap을 넣기는 애매하다 보니 아쉬움…)  
나열되어있는 순서대로 deployment, service, pvc, secrets으로 작성 하였다.

### 1. pgadmin-deployment.yaml

```yaml
dor1@is-master ~ ❯ mkdir pgadmin
mkdir: created directory 'default'
dor1@is-master ~ ❯ cd pgadmin

# pgadmin-deployment.yaml
dor1@is-master ~/pgadmin ❯ vi pgadmin-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgadmin
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: pgadmin
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: pgadmin
    spec:
      containers:
        - env:
            - name: PGADMIN_DEFAULT_EMAIL
              valueFrom:
                secretKeyRef:
                  key: PGADMIN_DEFAULT_EMAIL
                  name: pgadmin-secret
            - name: PGADMIN_DEFAULT_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: PGADMIN_DEFAULT_PASSWORD
                  name: pgadmin-secret
            - name: PGADMIN_PORT
              value: "80"
            - name: TZ
              value: "Asia/Seoul"
          image: dpage/pgadmin4:6
          imagePullPolicy: IfNotPresent
          name: pgadmin
          ports:
          - containerPort: 80
          resources:
            limits:
              memory: 4096Mi
          volumeMounts:
            - mountPath: /var/lib/pgadmin
              name: pgadmin-data
      restartPolicy: Always
      volumes:
        - name: pgadmin-data
          persistentVolumeClaim:
            claimName: pgadmin-data
      securityContext:
        runAsUser: 0
```

### 2. pgadmin-service.yaml

```yaml
# pgadmin-service.yaml
dor1@is-master ~/pgadmin ❯ vi pgadmin-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: pgadmin
  namespace: default
spec:
  ports:
  - name: http
    # nodePort 설정
    nodePort: 30001
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app.kubernetes.io/name: pgadmin
  type: NodePort
```

### 3. pgadmin-pvc.yaml

`rook-ceph`을 사용하기에 `storageClass`은 변경해준다.  
`pgadmin4`특성 상 설정 값만 들어가기에 size는 최대한 적게 만든다.

```yaml
# pgadmin-pvc.yaml
dor1@is-master ~/pgadmin ❯ vi pgadmin-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pgadmin-data
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: rook-cephfs
```

### 4. pgadmin-secrets.yaml

```yaml
# pgadmin-secret.yaml
dor1@is-master ~/pgadmin ❯ pgadmin-secret.yaml
apiVersion: v1
data:
  PGADMIN_DEFAULT_EMAIL: ZGVlcGFkbWluQGRlZXBub2lkLmNvbQ==
  PGADMIN_DEFAULT_PASSWORD: YW9vbmkzNjUhQA==
kind: Secret
metadata:
  name: pgadmin-secret
  namespace: default
```

## Deploy
---

간단히 적용해본다.

```shell
dor1@is-master ~/pgadmin ❯ kubectl apply -f .
deployment.apps/pgadmin created
persistentvolumeclaim/pgadmin-data created
secret/pgadmin-secret created
service/pgadmin created

# 확인
kubectl get po,svc,pvc,secret
NAME                           READY   STATUS    RESTARTS   AGE
pod/pgadmin-587c79bdcb-klnx8   1/1     Running   0          46s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.233.0.1     <none>        443/TCP        42d
service/pgadmin      NodePort    10.233.21.83   <none>        80:30001/TCP   47s

NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pgadmin-data   Bound    pvc-7daa67d5-e7e2-4a44-8115-3704267a993c   5Gi        RWO            rook-cephfs    47s

NAME                    TYPE     DATA   AGE
secret/pgadmin-secret   Opaque   2      47s
```

이후 접속 해본다.

![pgadmin4](/assets/img/post/cluster/kubernetes/2023-05-15-kubernetes-deploy_pgadmin4/1.png)
_pgadmin4_

## DB Connection
---

서로 다른 `namespace`에 있는 `postgres`를 연결 할 때는 `CoreDNS`사용하면 간단하게 된다.  
`CoreDNS`는 vanila k8s에서 calico(L3)로 설치하게 되면 대부분 포함 되어있다.  
`CoreDNS`의 DNS 형식은 보통 다음과 같다.

>*\<svc>.\<namespace>.cluster.local*

`cluster.local`은 k8s를 설치할 때 보통 parameter로 설정된다.  
우선 `Register` → `Server`를 통하여 등록 창을 띄워준다.

![Register → Server](/assets/img/post/cluster/kubernetes/2023-05-15-kubernetes-deploy_pgadmin4/2.png)
_Register → Server_

적절히 Name에 입력하고 다음 `Connection`탭으로 넘긴다.

![Name 입력 후 → Connection](/assets/img/post/cluster/kubernetes/2023-05-15-kubernetes-deploy_pgadmin4/3.png)
_Name 입력 후 → Connection_

`Host name/address`에 DNS를 넣어주면 된다.

![Host name/address 입력](/assets/img/post/cluster/kubernetes/2023-05-15-kubernetes-deploy_pgadmin4/4.png)
_Host name/address 입력_

알맞게 입력되었다면 정상적으로 확인할 수 있다.

![Dashboard 확인](/assets/img/post/cluster/kubernetes/2023-05-15-kubernetes-deploy_pgadmin4/5.png)
_Dashboard 확인_