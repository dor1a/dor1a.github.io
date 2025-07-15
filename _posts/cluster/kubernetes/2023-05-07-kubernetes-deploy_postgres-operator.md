---
title: Kubernetes - postgres-operator 배포해 보기
date: 2023-05-07 21:55:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, postgres]
description: Kubernetes에서 Postgres DB를 HA 구성 해주는 postgres-operator를 배포해 보았다.
---

>Kubernetes v1.24.7 / postgres-operator(Crunchy Data) v5.3.0 / Ubuntu 22.04 LTS
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 최근 동향을 보면 `MaraiaDB` → `PostgreSQL`으로 많이 넘어가는 추세이다.  
  추세의 변환에 따라 자연스럽게 기술 stack들은 `PostgreSQL`로 많이 넘어가고 있다.
* k8s를 사용을 하다 보면 각각의 기술 stack들이 NS(Namespace)마다 `Postgres` DB가 있는 것으로 확인되며, 운영 상 DB들을 하나로 통합 시켜 운영하는 것이 관리 목적 상 좋다고 생각이 들어 진행하게 되었다.
* `postgresql-operator`는 operator와 같이 CRD 기반으로 동작하며, 여러가지의 operator 중 Crunchy Data PostgreSQL Operator을 사용하게 되었다.  
  가장 큰 이유는 RDB를 제대로 잘 못 다루는 나에게 친숙한 `pgadmin4`의 GUI 대한 요구가 필요하였고, functional & simple 한 것들이 좋다고 판단하였다.  
  *cf. Comparing Kubernetes operators for PostgreSQL 2021 (<https://blog.palark.com/comparing-kubernetes-operators-for-postgresql/>)
* 현재 Crunchy Data PostgreSQL의 최신 버전은 v5.3.0이며 `PostgreSQL`의 버전은 14가 default이다.
* Crunchy Data PostgreSQL Operator의 Official URL이 제대로 보이지 않아 googling으로 찾아야 버전 별로 나온다.  
  <https://access.crunchydata.com/documentation/postgres-operator/5.3.0/>

![Crunchy Data PostgreSQL Operator HA Structure](/assets/img/post/cluster/kubernetes/2023-05-07-kubernetes-deploy_postgres-operator/1.png)
_Crunchy Data PostgreSQL Operator HA Structure_

## Install via kustomize (with. rook-ceph)
---

Crunchy Data PostgreSQL의 `postgres-operator`는 기본적으로 2가지 방법을 제공한다.
- kustomize
- helm
Docs에 있는 2가지 방법에서 추천하는 방법은 `kustomize`의 방법이며, `kustomize`는 앞서 awx-operator에서도 사용했던 것과 같다.

>*`kustomize`는 file로 이뤄진 config 값(`deployment`, `service`등)들을 하나의 file에서 관리할 수 있게 만들어진 configuration management solution이다.*  
*<https://kubectl.docs.kubernetes.io/installation/kustomize/>*

`kustomize`의 설치 방법은 다음의 링크에서 제공한다.  
<https://access.crunchydata.com/documentation/postgres-operator/5.3.0/installation/kustomize/>  
`Prerequisites`항목을 보면 `postgres-operator-examples`를 fork 한 후 하라고 하지만, 굳이 그럴 필요 없이 clone하여 사용해도 된다.

```shell
# postgres-operator
dor1@is-master:~$ git clone https://github.com/CrunchyData/postgres-operator-examples.git
Cloning into 'postgres-operator-examples'...
remote: Enumerating objects: 1452, done.
remote: Counting objects: 100% (32/32), done.
remote: Compressing objects: 100% (28/28), done.
remote: Total 1452 (delta 9), reused 16 (delta 4), pack-reused 1420
Receiving objects: 100% (1452/1452), 487.48 KiB | 3.29 MiB/s, done.
Resolving deltas: 100% (784/784), done.

dor1@is-master:~$ mv postgres-operator-examples postgres-operator
dor1@is-master:~/postgres-operator$ ll
total 16
drwxrwxr-x 1 deepadmin deepadmin    86 Mar  7 11:27 ./
drwxr-x--- 1 deepadmin deepadmin   312 Mar  7 11:29 ../
drwxrwxr-x 1 deepadmin deepadmin   138 Mar  7 11:27 .git/
drwxrwxr-x 1 deepadmin deepadmin    28 Mar  7 11:27 .github/
drwxrwxr-x 1 deepadmin deepadmin    30 Mar  7 11:27 helm/
drwxrwxr-x 1 deepadmin deepadmin   176 Mar  7 11:27 kustomize/
-rw-rw-r-- 1 deepadmin deepadmin 10780 Mar  7 11:27 LICENSE.md
-rw-rw-r-- 1 deepadmin deepadmin  1145 Mar  7 11:27 README.md
dor1@is-master:~/postgres-operator$ ll kustomize
total 0
drwxrwxr-x 1 deepadmin deepadmin 176 Mar  7 11:27 ./
drwxrwxr-x 1 deepadmin deepadmin  86 Mar  7 11:27 ../
drwxrwxr-x 1 deepadmin deepadmin 118 Mar  7 11:27 azure/
drwxrwxr-x 1 deepadmin deepadmin  48 Mar  7 11:27 certmanager/
drwxrwxr-x 1 deepadmin deepadmin  98 Mar  7 11:27 gcs/
drwxrwxr-x 1 deepadmin deepadmin  68 Mar  7 11:27 high-availability/
drwxrwxr-x 1 deepadmin deepadmin  90 Mar  7 11:27 install/
drwxrwxr-x 1 deepadmin deepadmin  88 Mar  7 11:27 keycloak/
drwxrwxr-x 1 deepadmin deepadmin 618 Mar  7 11:27 monitoring/
drwxrwxr-x 1 deepadmin deepadmin 164 Mar  7 11:27 multi-backup-repo/
drwxrwxr-x 1 deepadmin deepadmin  62 Mar  7 11:27 postgres/
drwxrwxr-x 1 deepadmin deepadmin 112 Mar  7 11:27 s3/
```

operator를 쓰는 목적 중 가장 큰 것은 HA(High Availability)이다 보니 위에서 `high-availability`와 `install` 을 사용하게 된다.  
우선 `install`항목을 통해 operator부터 deployment 한다.

```shell
# namespace
dor1@is-master:~/postgres-operator$ kubectl apply -k kustomize/install/namespace
namespace/postgres-operator created

# crd, clusterrole, clusterrolebindding, deployment
dor1@is-master:~/postgres-operator$ kubectl apply --server-side -k kustomize/install/default
customresourcedefinition.apiextensions.k8s.io/pgupgrades.postgres-operator.crunchydata.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/postgresclusters.postgres-operator.crunchydata.com serverside-applied
serviceaccount/pgo serverside-applied
serviceaccount/postgres-operator-upgrade serverside-applied
clusterrole.rbac.authorization.k8s.io/postgres-operator serverside-applied
clusterrole.rbac.authorization.k8s.io/postgres-operator-upgrade serverside-applied
clusterrolebinding.rbac.authorization.k8s.io/postgres-operator serverside-applied
clusterrolebinding.rbac.authorization.k8s.io/postgres-operator-upgrade serverside-applied
deployment.apps/pgo serverside-applied
deployment.apps/pgo-upgrade serverside-applied

# operator 확인
dor1@is-master:~/postgres-operator$ kubectl get deployments,pod -n postgres-operator
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pgo           1/1     1            1           3m57s
deployment.apps/pgo-upgrade   1/1     1            1           3m57s

AME                               READY   STATUS    RESTARTS   AGE
pod/pgo-79dd9d544d-q95h9           1/1     Running   0          3m57s
pod/pgo-upgrade-6bdd468d8d-s4dzx   1/1     Running   0          3m57s
```

operator가 다 deployment 되었다면, Postgre HA deployment를 해준다.  
DB 통합을 위해서 진행하는 만큼 용량은 최대한 크게 가져가고 안정성을 위해 node만큼 replica(용량은 배로 증가)를 맞춰준다.  
현재 시스템에는 `rook-ceph `CSI를 사용 중이라서 SC(StorageClass)를 `rook-ceph`에 맞춰 진행한다.  
`pgbackrest`는 `postgres`의 data가 다른 replica에 대해 parallel backup & restore 된다.  
<https://access.crunchydata.com/documentation/pgbackrest/latest/>  
`pgBouncer`은 connection pooler 기능(proxing)을 해준다.  
<https://github.com/pgbouncer/pgbouncer>

```yaml
# ha-postgres.yaml
dor1@is-master:~/postgres-operator$ vi ./kustomize/high-availability/ha-postgres.yaml
apiVersion: postgres-operator.crunchydata.com/v1beta1
kind: PostgresCluster
metadata:
  # hippo-ha -> postgres 변경
  name: postgres
spec:
  image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-14.6-2
  postgresVersion: 14
  instances:
    # pgha1 -> instance 변경 
    - name: instance
      # node에 맞춰 변경(2 -> 3)
      replicas: 3
      dataVolumeClaimSpec:
        accessModes:
        - "ReadWriteOnce"
        resources:
          requests:
            # 1Gi -> 500Gi (Total 1.5Ti)
            storage: 500Gi
        # rook-ceph 사용
        storageClassName: rook-cephfs
      # 같은 node에 대한 replica 배포 금지(AntiAffinity)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  # meatadata에 맞춰 변경
                  postgres-operator.crunchydata.com/cluster: postgres
                  postgres-operator.crunchydata.com/instance-set: instance
  backups:
    pgbackrest:
      image: registry.developers.crunchydata.com/crunchydata/crunchy-pgbackrest:ubi8-2.41-2
      repos:
      - name: repo1
        volume:
          volumeClaimSpec:
            accessModes:
            - "ReadWriteOnce"
            resources:
              requests:
                # instance에 맞춰 변경 1Gi -> 500Gi
                storage: 500Gi
            storageClassName: rook-cephfs
  proxy:
    pgBouncer:
      image: registry.developers.crunchydata.com/crunchydata/crunchy-pgbouncer:ubi8-1.17-5
      # instance의 replica에 맞춰 늘려줌
      replicas: 3
      # 같은 node에 대한 replica 배포 금지(AntiAffinity)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  # meatadata에 맞춰 변경
                  postgres-operator.crunchydata.com/cluster: postgres
                  postgres-operator.crunchydata.com/role: pgbouncer
```

```shell
# 적용
dor1@is-master:~/postgres-operator$ kubectl apply -k ./kustomize/high-availability
postgrescluster.postgres-operator.crunchydata.com/postgres created

# 확인
dor1@is-master:~/postgres-operator$ kubectl get deployments,pod,pvc,cm,secrets,svc -n postgres-operator
NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pgo                  1/1     1            1           4h3m
deployment.apps/pgo-upgrade          1/1     1            1           4h3m
deployment.apps/postgres-pgbouncer   3/3     3            3           98s

NAME                                      READY   STATUS    RESTARTS   AGE
pod/pgo-79dd9d544d-q95h9                  1/1     Running   0          4h3m
pod/pgo-upgrade-6bdd468d8d-s4dzx          1/1     Running   0          4h3m
pod/postgres-backup-lbnx-tfjkw            1/1     Running   0          65s
pod/postgres-instance-94v7-0              4/4     Running   0          98s
pod/postgres-instance-t7xx-0              4/4     Running   0          98s
pod/postgres-instance-zskm-0              4/4     Running   0          99s
pod/postgres-pgbouncer-58b849fc58-9v2c9   2/2     Running   0          98s
pod/postgres-pgbouncer-58b849fc58-qvjtf   2/2     Running   0          98s
pod/postgres-pgbouncer-58b849fc58-rt64x   2/2     Running   0          98s
pod/postgres-repo-host-0                  2/2     Running   0          98s

NAME                                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/postgres-instance-94v7-pgdata   Bound    pvc-d7ff9e0d-90d3-4611-a754-982a27810dd8   500Gi      RWO            rook-cephfs    99s
persistentvolumeclaim/postgres-instance-t7xx-pgdata   Bound    pvc-3ec7ad36-77f5-4f64-a452-a2367e637f8e   500Gi      RWO            rook-cephfs    98s
persistentvolumeclaim/postgres-instance-zskm-pgdata   Bound    pvc-6b3e83ec-570d-41ae-bae2-705ad1cc87c3   500Gi      RWO            rook-cephfs    99s
persistentvolumeclaim/postgres-repo1                  Bound    pvc-634024fe-9bc5-4ab5-984b-ad97022c9494   500Gi      RWO            rook-cephfs    98s

NAME                                      DATA   AGE
configmap/kube-root-ca.crt                1      4h16m
configmap/pgo-upgrade-check               1      4h14m
configmap/postgres-config                 1      99s
configmap/postgres-instance-94v7-config   1      99s
configmap/postgres-instance-t7xx-config   1      98s
configmap/postgres-instance-zskm-config   1      99s
configmap/postgres-pgbackrest-config      4      98s
configmap/postgres-pgbouncer              2      98s

NAME                                  TYPE     DATA   AGE
secret/pgo-root-cacert                Opaque   2      99s
secret/postgres-cluster-cert          Opaque   3      99s
secret/postgres-instance-94v7-certs   Opaque   6      99s
secret/postgres-instance-t7xx-certs   Opaque   6      98s
secret/postgres-instance-zskm-certs   Opaque   6      99s
secret/postgres-pgbackrest            Opaque   5      98s
secret/postgres-pgbouncer             Opaque   6      98s
secret/postgres-pguser-postgres       Opaque   12     98s
secret/postgres-replication-cert      Opaque   3      99s

NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/postgres-ha          ClusterIP   10.233.2.65     <none>        5432/TCP   99s
service/postgres-ha-config   ClusterIP   None            <none>        <none>     99s
service/postgres-pgbouncer   ClusterIP   10.233.8.73     <none>        5432/TCP   98s
service/postgres-pods        ClusterIP   None            <none>        <none>     99s
service/postgres-primary     ClusterIP   None            <none>        5432/TCP   99s
service/postgres-replicas    ClusterIP   10.233.62.128   <none>        5432/TCP   99s
```

### pgadmin4 deployment

앞서 개요에서 언급했던 것과 같이 GUI가 필요하여 `pgadmin4`을 사용하기 위해서는 몇몇 항목을 추가해준다.  
<https://access.crunchydata.com/documentation/postgres-operator/5.3.0/architecture/pgadmin4/>  
단, URL에 명시 되어있는 버전을 쓰면 버그로 인하여 아래와 같은 error가 발생하게 되는데 버전을 올려주면 해결된다.

>'psycopg2.extensions.Column' object has no attribute '_asdict'
><https://github.com/CrunchyData/postgres-operator/issues/3375>

```yaml
# ha-postgres.yaml(pgadmin4 추가)
dor1@is-master:~/postgres-operator$ vi ./kustomize/high-availability/ha-postgres.yaml
apiVersion: postgres-operator.crunchydata.com/v1beta1
kind: PostgresCluster
metadata:
  # hippo-ha -> postgres 변경
  name: postgres
spec:
  image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-14.6-2
  postgresVersion: 14
  instances:
    # pgha1 -> instance 변경 
    - name: instance
      # node에 맞춰 변경(2 -> 3)
      replicas: 3
      dataVolumeClaimSpec:
        accessModes:
        - "ReadWriteOnce"
        resources:
          requests:
            # 1Gi -> 500Gi (Total 1.5Ti)
            storage: 500Gi
        # rook-ceph 사용
        storageClassName: rook-cephfs
      # 같은 node에 대한 replica 배포 금지(AntiAffinity)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  # meatadata에 맞춰 변경
                  postgres-operator.crunchydata.com/cluster: postgres
                  postgres-operator.crunchydata.com/instance-set: instance
  backups:
    pgbackrest:
      image: registry.developers.crunchydata.com/crunchydata/crunchy-pgbackrest:ubi8-2.41-2
      repos:
      - name: repo1
        volume:
          volumeClaimSpec:
            accessModes:
            - "ReadWriteOnce"
            resources:
              requests:
                # instance에 맞춰 변경 1Gi -> 500Gi
                storage: 500Gi
            storageClassName: rook-cephfs
  proxy:
    pgBouncer:
      image: registry.developers.crunchydata.com/crunchydata/crunchy-pgbouncer:ubi8-1.17-5
      # instance의 replica에 맞춰 늘려줌
      replicas: 3
      # 같은 node에 대한 replica 배포 금지(AntiAffinity)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  # meatadata에 맞춰 변경
                  postgres-operator.crunchydata.com/cluster: postgres
                  postgres-operator.crunchydata.com/role: pgbouncer
  # pgadmin4 항목추가
  userInterface:
    pgAdmin:
      # 버그로 인하여 버전 변경 4.30-4 -> 4.30-8
      image: registry.developers.crunchydata.com/crunchydata/crunchy-pgadmin4:ubi8-4.30-8
      dataVolumeClaimSpec:
        accessModes:
        - "ReadWriteOnce"
        resources:
          requests:
            storage: 1Gi
        storageClassName: rook-cephfs
      # nodeport 설정 
      service:
        type: NodePort
        nodePort: 30001
```

```shell
# 적용
dor1@is-master:~/postgres-operator$ kubectl apply -k ./kustomize/high-availability
postgrescluster.postgres-operator.crunchydata.com/postgres configured

# 확인
dor1@is-master:~/postgres-operator$ kubectl get deployments,pod,pvc,cm,secrets,svc -n postgres-operator
NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pgo                  1/1     1            1           4h7m
deployment.apps/pgo-upgrade          1/1     1            1           4h7m
deployment.apps/postgres-pgbouncer   3/3     3            3           5m57s

NAME                                      READY   STATUS      RESTARTS   AGE
pod/pgo-79dd9d544d-q95h9                  1/1     Running     0          4h7m
pod/pgo-upgrade-6bdd468d8d-s4dzx          1/1     Running     0          4h7m
pod/postgres-backup-lbnx-tfjkw            0/1     Completed   0          5m24s
pod/postgres-instance-94v7-0              4/4     Running     0          5m57s
pod/postgres-instance-t7xx-0              4/4     Running     0          5m57s
pod/postgres-instance-zskm-0              4/4     Running     0          5m58s
pod/postgres-pgadmin-0                    1/1     Running     0          78s
pod/postgres-pgbouncer-58b849fc58-9v2c9   2/2     Running     0          5m57s
pod/postgres-pgbouncer-58b849fc58-qvjtf   2/2     Running     0          5m57s
pod/postgres-pgbouncer-58b849fc58-rt64x   2/2     Running     0          5m57s
pod/postgres-repo-host-0                  2/2     Running     0          5m57s

NAME                                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/postgres-instance-94v7-pgdata   Bound    pvc-d7ff9e0d-90d3-4611-a754-982a27810dd8   500Gi      RWO            rook-cephfs    5m58s
persistentvolumeclaim/postgres-instance-t7xx-pgdata   Bound    pvc-3ec7ad36-77f5-4f64-a452-a2367e637f8e   500Gi      RWO            rook-cephfs    5m57s
persistentvolumeclaim/postgres-instance-zskm-pgdata   Bound    pvc-6b3e83ec-570d-41ae-bae2-705ad1cc87c3   500Gi      RWO            rook-cephfs    5m58s
persistentvolumeclaim/postgres-pgadmin                Bound    pvc-fac26508-a0b5-4951-8f40-9fd779abad9e   1Gi        RWO            rook-cephfs    79s
persistentvolumeclaim/postgres-repo1                  Bound    pvc-634024fe-9bc5-4ab5-984b-ad97022c9494   500Gi      RWO            rook-cephfs    5m57s

NAME                                      DATA   AGE
configmap/kube-root-ca.crt                1      4h20m
configmap/pgo-upgrade-check               1      4h19m
configmap/postgres-config                 1      5m58s
configmap/postgres-instance-94v7-config   1      5m58s
configmap/postgres-instance-t7xx-config   1      5m57s
configmap/postgres-instance-zskm-config   1      5m58s
configmap/postgres-pgadmin                1      79s
configmap/postgres-pgbackrest-config      4      5m57s
configmap/postgres-pgbouncer              2      5m57s

NAME                                  TYPE     DATA   AGE
secret/pgo-root-cacert                Opaque   2      5m58s
secret/postgres-cluster-cert          Opaque   3      5m58s
secret/postgres-instance-94v7-certs   Opaque   6      5m58s
secret/postgres-instance-t7xx-certs   Opaque   6      5m57s
secret/postgres-instance-zskm-certs   Opaque   6      5m58s
secret/postgres-pgbackrest            Opaque   5      5m57s
secret/postgres-pgbouncer             Opaque   6      5m57s
secret/postgres-pguser-postgres       Opaque   12     5m57s
secret/postgres-replication-cert      Opaque   3      5m58s

NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/postgres-ha          ClusterIP   10.233.2.65     <none>        5432/TCP         5m58s
service/postgres-ha-config   ClusterIP   None            <none>        <none>           5m58s
service/postgres-pgadmin     NodePort    10.233.24.18    <none>        5050:30001/TCP   79s
service/postgres-pgbouncer   ClusterIP   10.233.8.73     <none>        5432/TCP         5m57s
service/postgres-pods        ClusterIP   None            <none>        <none>           5m58s
service/postgres-primary     ClusterIP   None            <none>        5432/TCP         5m58s
service/postgres-replicas    ClusterIP   10.233.62.128   <none>        5432/TCP         5m58s
```

정상적으로 deployment가 되었다면, 접속 해본다.

![pgadmin4](/assets/img/post/cluster/kubernetes/2023-05-07-kubernetes-deploy_postgres-operator/2.png)
_pgadmin4_

기본 계정은 다음과 같이 확인이 가능하다.  
Username은 출력 되는 ID에 @pgo를 붙여서 진행(postgres@pgo)한다.

<pre>
{% raw %}
# user
kubectl get secrets -n postgres-operator "postgres-pguser-postgres" -o go-template='{{.data.user | base64decode}}'
postgres

# password
kubectl get secrets -n postgres-operator "postgres-pguser-postgres" -o go-template='{{.data.password | base64decode}}'
-z3,C75}Tdv4dsK83dwa+A)5
{% endraw %}
</pre>