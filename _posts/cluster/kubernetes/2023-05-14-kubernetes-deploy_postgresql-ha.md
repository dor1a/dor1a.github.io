---
title: Kubernetes - postgresql-ha 배포해 보기
date: 2023-05-14 18:01:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, postgres]
description: Kubernetes에서 Postgres DB를 HA 구성 해주는 postgresql-ha를 배포해 보았다.
---

>Kubernetes v1.24.7 / Helm 3.9.0 / postgresql-ha(bitnami) 11.1.5 / Ubuntu 22.04 LTS
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* 기존에 [postgres-operator](/posts/kubernetes-deploy_postgres-operator/)로 진행하려 했지만 기능도 너무 많고 다루기가 어려워, 간단하게 HA(High Availability)로 구성되어있는 항목 중 [Bitnami](https://github.com/bitnami/charts) 의 chart를 발견하여 진행하게 되었다.  
  (사실 넘어오게 된 가장 큰 이유 중 하나는 구체적인 postgres의 DB설정에 직접적으로 접근하기가 힘들기 때문)
* [postgres-operator](/posts/kubernetes-deploy_postgres-operator)에 비해서 postgresql-ha는 [patroni](https://github.com/zalando/patroni)와 같은 기능은 없고, 단순하게 [pgpool](https://pgpool.net/mediawiki/index.php/Main_Page)을 이용한다.

![Bitnami의 PostgreSQL HA Architecture](/assets/img/post/cluster/kubernetes/2023-05-14-kubernetes-deploy_postgresql-ha/1.png)
_Bitnami의 PostgreSQL HA Architecture_

## Deploy via helm
---

`TL;DR` 항목으로 보면 간단하게 두 줄로 이뤄져 있지만 막상 진행해보면 `pending` 상태로 진행이 안된다.  
관리자 입장에 맞춰 설정 변경을 하여 진행 하는 것이 좋다.  
<https://github.com/bitnami/charts/tree/main/bitnami/postgresql-ha>

### 1. Add helm repo

```shell
# add helm repo
dor1@is-master ~ > helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

# update helm repo (recommend)
dor1@is-master ~ > helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈

# check helm repo
dor1@is-master ~ > helm repo ls
NAME    URL
bitnami https://charts.bitnami.com/bitnami
```

### 2. vaules.yaml를 통한 설정 변경

`helm chart`는 기본적으로 `values.yaml`을 바라보고 그 안에는 parameters가 존재한다.  
parameters들은 `pod`의 env에 맞춰져 있으며 대다수 official manual에 요구하는 정의가 적혀 있다.

```shell
# vaules.yaml
dor1@is-master ~ > mkdir postgresql-ha
mkdir: created directory 'postgresql-ha'

dor1@is-master ~ > cd postgresql-ha
dor1@is-master ~/postgresql-ha ❯ helm show values bitnami/postgresql-ha > values.yaml
```

`values.yaml`에는 엄청 많은 parameters와 함께 주석이 들어가 있기 때문에 주석을 제거 해준 뒤 필요한 parameter만 정의해준다.  
`vim`으로 연 뒤에 [vim 명령 정리](/posts/linux-vim_command/) 항목을 따라 주석 제거를 해준다.  
`vim`으로 주석 제거 후 아래 항목에서 필요한 설정을 진행한다. (parameter가 너무 많다 보니 필요 부분 외 모두 주석 처리)  
[witness](https://repmgr.org/docs/5.3/repmgrd-witness-server.html)항목은 필요 시에 `true`로 설정하여 만들어준다.  
또한 `rook-ceph`을 사용한다면 `persistence` 부분의 `storageClass`를 변경해준다.

```yaml
# values.yaml
dor1@is-master ~/postgresql-ha ❯ vi ./values.yaml
#global:
#  imageRegistry: ""
#  imagePullSecrets: []
#  storageClass: ""
#  postgresql:
#    username: ""
#    password: ""
#    database: ""
#    repmgrUsername: ""
#    repmgrPassword: ""
#    repmgrDatabase: ""
#    existingSecret: ""
#  ldap:
#    bindpw: ""
#    existingSecret: ""
#  pgpool:
#    adminUsername: ""
#    adminPassword: ""
#    existingSecret: ""

#kubeVersion: ""
#nameOverride: ""
#fullnameOverride: ""
#namespaceOverride: ""
#commonLabels: {}
#commonAnnotations: {}
clusterDomain: cluster.local
#extraDeploy: []
diagnosticMode:
  enabled: false
  command:
    - sleep
  args:
    - infinity

postgresql:
  image:
    registry: docker.io
    repository: bitnami/postgresql-repmgr
    tag: 15.2.0-debian-11-r12
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    # change that false -> true for looging
    debug: true
#  labels: {}
#  podLabels: {}
#  serviceAnnotations: {}
  replicaCount: 3
  updateStrategy:
    type: RollingUpdate
  containerPorts:
    postgresql: 5432
#  hostAliases: []
  hostNetwork: false
  hostIPC: false
#  podAnnotations: {}
#  podAffinityPreset: ""
  podAntiAffinityPreset: soft
#  nodeAffinityPreset:
#    type: ""
#    key: ""
#    values: []
#  affinity: {}
#  nodeSelector: {}
#  tolerations: []
#  topologySpreadConstraints: []
#  priorityClassName: ""
#  schedulerName: ""
#  terminationGracePeriodSeconds: ""
  podSecurityContext:
    enabled: true
    fsGroup: 1001
  containerSecurityContext:
    enabled: true
    runAsUser: 1001
    runAsNonRoot: true
    readOnlyRootFilesystem: false
#  command: []
#  args: []
#  lifecycleHooks: {}
#  extraEnvVars: []
#  extraEnvVarsCM: ""
#  extraEnvVarsSecret: ""
#  extraVolumes: []
#  extraVolumeMounts: []
#  initContainers: []
#  sidecars: []
#  resources:
#    limits: {}
#    requests: {}
  livenessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 6
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 6
  startupProbe:
    enabled: false
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 10
#  customLivenessProbe: {}
#  customReadinessProbe: {}
#  customStartupProbe: {}
  pdb:
    create: false
    minAvailable: 1
    maxUnavailable: ""
  ## change userid (postgres -> ???)
  username: deepadmin
  password: ""
  ## add DB name match userid
  database: deepadmin
  ## add secrets and then create secrets file
  existingSecret: postgresql-secrets
  postgresPassword: ""
#  usePasswordFile: ""
#  repmgrUsePassfile: ""
#  repmgrPassfilePath: ""
  upgradeRepmgrExtension: false
  pgHbaTrustAll: false
  syncReplication: false
  repmgrUsername: repmgr
  repmgrPassword: ""
  repmgrDatabase: repmgr
  repmgrLogLevel: NOTICE
  repmgrConnectTimeout: 5
  repmgrReconnectAttempts: 2
  repmgrReconnectInterval: 3
  repmgrFenceOldPrimary: false
  repmgrChildNodesCheckInterval: 5
  repmgrChildNodesConnectedMinCount: 1
  repmgrChildNodesDisconnectTimeout: 30
  usePgRewind: false
  audit:
    logHostname: true
    logConnections: false
    logDisconnections: false
    pgAuditLog: ""
    pgAuditLogCatalog: "off"
    clientMinMessages: error
    logLinePrefix: ""
    ## setting TZ
    logTimezone: Asia/Seoul
  sharedPreloadLibraries: "pgaudit, repmgr"
#  maxConnections: ""
#  postgresConnectionLimit: ""
#  dbUserConnectionLimit: ""
#  tcpKeepalivesInterval: ""
#  tcpKeepalivesIdle: ""
#  tcpKeepalivesCount: ""
#  statementTimeout: ""
#  pghbaRemoveFilters: ""
#  extraInitContainers: []
#  repmgrConfiguration: ""
#  configuration: ""
#  pgHbaConfiguration: ""
#  configurationCM: ""
#  extendedConf: ""
#  extendedConfCM: ""
#  initdbScripts: {}
#  initdbScriptsCM: ""
#  initdbScriptsSecret: ""
  tls:
    enabled: false
    preferServerCiphers: true
    certificatesSecret: ""
    certFilename: ""
    certKeyFilename: ""

witness:
  ## if use change that false -> true
  create: true
#  labels: {}
#  podLabels: {}
  replicaCount: 1
  updateStrategy:
    type: RollingUpdate
  containerPorts:
    postgresql: 5432
#  hostAliases: []
  hostNetwork: false
  hostIPC: false
#  podAnnotations: {}
#  podAffinityPreset: ""
  podAntiAffinityPreset: soft
#  nodeAffinityPreset:
#    type: ""
#    key: ""
#    values: []
#  affinity: {}
#  nodeSelector: {}
#  tolerations: []
#  topologySpreadConstraints: []
#  priorityClassName: ""
#  schedulerName: ""
#  terminationGracePeriodSeconds: ""
  podSecurityContext:
    enabled: true
    fsGroup: 1001
  containerSecurityContext:
    enabled: true
    runAsUser: 1001
    runAsNonRoot: true
    readOnlyRootFilesystem: false
#  command: []
#  args: []
#  lifecycleHooks: {}
#  extraEnvVars: []
#  extraEnvVarsCM: ""
#  extraEnvVarsSecret: ""
#  extraVolumes: []
#  extraVolumeMounts: []
#  initContainers: []
#  sidecars: []
#  resources:
#    limits: {}
#    requests: {}
  livenessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 6
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 6
  startupProbe:
    enabled: false
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 10
#  customLivenessProbe: {}
#  customReadinessProbe: {}
#  customStartupProbe: {}
  pdb:
    create: false
    minAvailable: 1
    maxUnavailable: ""
  upgradeRepmgrExtension: false
  pgHbaTrustAll: false
  repmgrLogLevel: NOTICE
  repmgrConnectTimeout: 5
  repmgrReconnectAttempts: 2
  repmgrReconnectInterval: 3
  audit:
    logHostname: true
    logConnections: false
    logDisconnections: false
    pgAuditLog: ""
    pgAuditLogCatalog: "off"
    clientMinMessages: error
    logLinePrefix: ""
    ## setting TZ
    logTimezone: Asia/Seoul
#  maxConnections: ""
#  postgresConnectionLimit: ""
#  dbUserConnectionLimit: ""
#  tcpKeepalivesInterval: ""
#  tcpKeepalivesIdle: ""
#  tcpKeepalivesCount: ""
#  statementTimeout: ""
#  pghbaRemoveFilters: ""
#  extraInitContainers: []
#  repmgrConfiguration: ""
#  configuration: ""
#  pgHbaConfiguration: ""
#  configurationCM: ""
#  extendedConf: ""
#  extendedConfCM: ""
#  initdbScripts: {}
#  initdbScriptsCM: ""
#  initdbScriptsSecret: ""

pgpool:
  image:
    registry: docker.io
    repository: bitnami/pgpool
    tag: 4.4.2-debian-11-r15
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    ## change that false -> true for looging
    debug: false
  customUsers: {}
  usernames: ""
  passwords: ""
#  hostAliases: []
  ## add secrets and then create secrets file
  customUsersSecret: pgpool-secrets
#  existingSecret: ""
  srCheckDatabase: postgres
#  labels: {}
#  podLabels: {}
#  serviceLabels: {}
#  serviceAnnotations: {}
#  customLivenessProbe: {}
#  customReadinessProbe: {}
#  customStartupProbe: {}
#  command: []
#  args: []
#  lifecycleHooks: {}
#  extraEnvVars: []
#  extraEnvVarsCM: ""
#  extraEnvVarsSecret: ""
#  extraVolumes: []
#  extraVolumeMounts: []
#  initContainers: []
#  sidecars: []
  replicaCount: 1
#  podAnnotations: {}
#  priorityClassName: ""
#  schedulerName: ""
#  terminationGracePeriodSeconds: ""
#  topologySpreadConstraints: []
#  podAffinityPreset: ""
  podAntiAffinityPreset: soft
#  nodeAffinityPreset:
#    type: ""
#    key: ""
#    values: []
#  affinity: {}
#  nodeSelector: {}
#  tolerations: []
  podSecurityContext:
    enabled: true
    fsGroup: 1001
  containerSecurityContext:
    enabled: true
    runAsUser: 1001
    runAsNonRoot: true
    readOnlyRootFilesystem: false
#  resources:
#    limits: {}
#    requests: {}
  livenessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  startupProbe:
    enabled: false
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 10
  pdb:
    create: false
    minAvailable: 1
    maxUnavailable: ""
#  updateStrategy: {}
  containerPorts:
    postgresql: 5432
#  minReadySeconds: ""
  adminUsername: admin
  adminPassword: ""
#  usePasswordFile: ""
  authenticationMethod: scram-sha-256
  logConnections: false
  logHostname: true
  logPerNodeStatement: false
  logLinePrefix: ""
  clientMinMessages: error
#  numInitChildren: ""
  reservedConnections: 1
#  maxPool: ""
#  childMaxConnections: ""
#  childLifeTime: ""
#  clientIdleLimit: ""
#  connectionLifeTime: ""
  useLoadBalancing: true
  loadBalancingOnWrite: transaction
#  configuration: ""
#  configurationCM: ""
#  initdbScripts: {}
#  initdbScriptsCM: ""
#  initdbScriptsSecret: ""
  tls:
    enabled: false
    autoGenerated: false
    preferServerCiphers: true
    certificatesSecret: ""
    certFilename: ""
    certKeyFilename: ""
    certCAFilename: ""

ldap:
  enabled: false
  existingSecret: ""
  uri: ""
  basedn: ""
  binddn: ""
  bindpw: ""
  bslookup: ""
  scope: ""
  tlsReqcert: ""
  nssInitgroupsIgnoreusers: root,nslcd

rbac:
  create: false
  rules: []

serviceAccount:
  create: false
  name: ""
  annotations: {}
  automountServiceAccountToken: true

psp:
  create: false

metrics:
  enabled: false
  image:
    registry: docker.io
    repository: bitnami/postgres-exporter
    tag: 0.11.1-debian-11-r69
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    debug: false
  podSecurityContext:
    enabled: true
    runAsUser: 1001
#  resources:
#    limits: {}
#    requests: {}
  containerPorts:
    http: 9187
  livenessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 6
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 6
  startupProbe:
    enabled: false
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 10
#  customLivenessProbe: {}
#  customReadinessProbe: {}
#  customStartupProbe: {}
  service:
    type: ClusterIP
    ports:
      metrics: 9187
    nodePorts:
      metrics: ""
    clusterIP: ""
    loadBalancerIP: ""
    loadBalancerSourceRanges: []
    externalTrafficPolicy: Cluster
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9187"
#  customMetrics: {}
#  extraEnvVars: []
#  extraEnvVarsCM: ""
#  extraEnvVarsSecret: ""
  serviceMonitor:
    enabled: false
    namespace: ""
    interval: ""
    scrapeTimeout: ""
    annotations: {}
    labels: {}
    selector:
      prometheus: kube-prometheus
    relabelings: []
    metricRelabelings: []
    honorLabels: false
    jobLabel: ""

volumePermissions:
  enabled: false
  image:
    registry: docker.io
    repository: bitnami/bitnami-shell
    tag: 11-debian-11-r96
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
  podSecurityContext:
    runAsUser: 0
#  resources:
#    limits: {}
#    requests: {}

persistence:
  enabled: true
  existingClaim: ""
  ## if use ceph storge, input storageclass
  storageClass: rook-cephfs
  mountPath: /bitnami/postgresql
  accessModes:
    - ReadWriteMany
  ## 8Gi -> 300Gi
  size: 300Gi
  annotations: {}
  labels: {}
  selector: {}

service:
  type: ClusterIP
  ports:
    postgresql: 5432
  portName: postgresql
  nodePorts:
    postgresql: ""
  loadBalancerIP: ""
  loadBalancerSourceRanges: []
  clusterIP: ""
  externalTrafficPolicy: Cluster
  extraPorts: []
  sessionAffinity: "None"
  sessionAffinityConfig: {}
  annotations: {}
  serviceLabels: {}

networkPolicy:
  enabled: false
  allowExternal: true
  egressRules:
    denyConnectionsToExternal: false
    customRules: []
```

위와 같이 설정하였다면 중간에 `secret` 들어가는 항목에 맞춰 password들을 secret.yaml 파일을 만들어 넣어준다.

```yaml
dor1@is-master ~/postgresql-ha ❯ vi postgresql-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgresql-secrets
  namespace: postgresql-ha
type: Opaque
data:
  # passwrod for postgres "postgres" passsword
  postgres-password: "VGNHVlNSdDRKQA=="
  # password for postgres "user" password
  password: "YW9vbmkzNjUhQA=="
  # repmgr password
  repmgr-password: "VGNHVlNSdDRKQA=="
  # pgpool admin password
  admin-password: "VGNHVlNSdDRKQA=="
  # pgpool custom username & password
  usernames: "ZGVlcGFkbWlu"
  passwords: "YW9vbmkzNjUhQA=="
```

```yaml
dor1@is-master ~/postgresql-ha ❯ vi pgpool-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: pgpool-secrets
  namespace: postgresql-ha
type: Opaque
data:
  # pgpool admin password
  admin-password: "VGNHVlNSdDRKQA=="
  # pgpool custom username & password
  usernames: "ZGVlcGFkbWlu"
  passwords: "YW9vbmkzNjUhQA=="
```

여기까지 다 되었다면 deployment 준비가 다 됐다.

### 3. Deployment

순차적으로 진행한다. (namespace → secrets → helm install)

```shell
# namespace
dor1@is-master ~/postgresql-ha ❯ kubectl create ns postgresql-ha
namespace/postgresql-ha created

# secrets
dor1@is-master ~/postgresql-ha ❯ kubectl apply -f ./postgresql-secrets.yaml -f ./pgpool-secrets.yaml
secret/postgresql-secrets created
secret/pgpool-secrets created

# helm install
dor1@is-master ~/postgresql-ha ❯ helm install postgresql-ha bitnami/postgresql-ha -n postgresql-ha -f ./values.yaml
NAME: postgresql-ha
LAST DEPLOYED: Wed Mar 15 16:06:02 2023
NAMESPACE: postgresql-ha
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql-ha
CHART VERSION: 11.1.5
APP VERSION: 15.2.0
** Please be patient while the chart is being deployed **
PostgreSQL can be accessed through Pgpool via port 5432 on the following DNS name from within your cluster:

    postgresql-ha-pgpool.postgresql-ha.svc.cluster.local

Pgpool acts as a load balancer for PostgreSQL and forward read/write connections to the primary node while read-only connections are forwarded to standby nodes.

To get the password for "deepadmin" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace postgresql-ha postgresql-secrets -o jsonpath="{.data.password}" | base64 -d)

To get the password for "repmgr" run:

    export REPMGR_PASSWORD=$(kubectl get secret --namespace postgresql-ha postgresql-secrets -o jsonpath="{.data.repmgr-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgresql-ha-client --rm --tty -i --restart='Never' --namespace postgresql-ha --image docker.io/bitnami/postgresql-repmgr:15.2.0-debian-11-r12 --env="PGPASSWORD=$POSTGRES_PASSWORD"  \
        --command -- psql -h postgresql-ha-pgpool -p 5432 -U deepadmin -d deepadmin

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace postgresql-ha svc/postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U deepadmin -d deepadmin

# 확인
dor1@is-master ~/postgresql-ha ❯ helm ls -A
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
postgresql-ha   postgresql-ha   1               2023-03-15 16:06:02.432037109 +0900 KST deployed        postgresql-ha-11.1.5    15.2.0

dor1@is-master ~/postgresql-ha ❯ kubectl get po,deployments,svc,cm,secrets,pv,pvc -n postgresql-ha
NAME                                       READY   STATUS    RESTARTS      AGE
pod/postgresql-ha-pgpool-779444dbf-9b5w5   1/1     Running   0             3m41s
pod/postgresql-ha-postgresql-0             1/1     Running   0             3m41s
pod/postgresql-ha-postgresql-1             1/1     Running   0             3m40s
pod/postgresql-ha-postgresql-2             1/1     Running   2 (60s ago)   3m40s
pod/postgresql-ha-postgresql-witness-0     1/1     Running   0             3m41s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgresql-ha-pgpool   1/1     1            1           3m41s

NAME                                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/postgresql-ha-pgpool                ClusterIP   10.233.41.96    <none>        5432/TCP   3m41s
service/postgresql-ha-postgresql            ClusterIP   10.233.15.213   <none>        5432/TCP   3m41s
service/postgresql-ha-postgresql-headless   ClusterIP   None            <none>        5432/TCP   3m41s
service/postgresql-ha-postgresql-witness    ClusterIP   None            <none>        5432/TCP   3m41s

NAME                                               DATA   AGE
configmap/kube-root-ca.crt                         1      3m50s
configmap/postgresql-ha-postgresql-hooks-scripts   1      3m41s

NAME                                         TYPE                 DATA   AGE
secret/pgpool-secrets                        Opaque               3      3m49s
secret/postgresql-ha-pgpool                  Opaque               1      3m41s
secret/postgresql-secrets                    Opaque               6      3m49s
secret/sh.helm.release.v1.postgresql-ha.v1   helm.sh/release.v1   1      3m41s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                   STORAGECLASS   REASON   AGE
persistentvolume/pvc-7edb7e28-7504-4dac-9e7d-27456d5392e7   300Gi      RWO            Retain           Bound    postgresql-ha/data-postgresql-ha-postgresql-2           rook-cephfs             3m39s
persistentvolume/pvc-a69ecfa5-db7a-4f10-94c6-f946f98a9295   300Gi      RWO            Retain           Bound    postgresql-ha/data-postgresql-ha-postgresql-0           rook-cephfs             3m40s
persistentvolume/pvc-ae656754-99eb-4249-83b0-9961f95540d9   300Gi      RWO            Retain           Bound    postgresql-ha/data-postgresql-ha-postgresql-1           rook-cephfs             3m39s
persistentvolume/pvc-ea605028-15e4-46d1-9f7a-d47714d94887   300Gi      RWO            Retain           Bound    postgresql-ha/data-postgresql-ha-postgresql-witness-0   rook-cephfs             3m40s

NAME                                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data-postgresql-ha-postgresql-0           Bound    pvc-a69ecfa5-db7a-4f10-94c6-f946f98a9295   300Gi      RWO            rook-cephfs    3m41s
persistentvolumeclaim/data-postgresql-ha-postgresql-1           Bound    pvc-ae656754-99eb-4249-83b0-9961f95540d9   300Gi      RWO            rook-cephfs    3m40s
persistentvolumeclaim/data-postgresql-ha-postgresql-2           Bound    pvc-7edb7e28-7504-4dac-9e7d-27456d5392e7   300Gi      RWO            rook-cephfs    3m40s
persistentvolumeclaim/data-postgresql-ha-postgresql-witness-0   Bound    pvc-ea605028-15e4-46d1-9f7a-d47714d94887   300Gi      RWO            rook-cephfs    3m41s
```

## postgresql-ha-client
---

HA로 구성되어있는 `postgres`의 접근을 위해서는 `pgpool`에 직접 접근이 되지는 않아 postgresql-ha-client를 따로 만들어서 진행해준다.  
위의 `Deployment` 항목을 본다면 다음과 같은 출력이 되어있는 것을 확인할 수 있다.

```shell
Pgpool acts as a load balancer for PostgreSQL and forward read/write connections to the primary node while read-only connections are forwarded to standby nodes.

To get the password for "deepadmin" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace postgresql-ha postgresql-secrets -o jsonpath="{.data.password}" | base64 -d)

To get the password for "repmgr" run:

    export REPMGR_PASSWORD=$(kubectl get secret --namespace postgresql-ha postgresql-secrets -o jsonpath="{.data.repmgr-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgresql-ha-client --rm --tty -i --restart='Never' --namespace postgresql-ha --image docker.io/bitnami/postgresql-repmgr:15.2.0-debian-11-r12 --env="PGPASSWORD=$POSTGRES_PASSWORD"  \
        --command -- psql -h postgresql-ha-pgpool -p 5432 -U deepadmin -d deepadmin

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace postgresql-ha svc/postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U deepadmin -d deepadmin
```

그러나 관리의 편리함에 있어서는 위의 명령어를 매일 치기보다 `pod`를 하나 만들어서 접근하여 관리 하는 것이 편하다.  
`pod`의 접근을 빨리 하기 위해서는 `namespace`를 `default`로 관리 해주는 것이 편하다.

```yaml
# postgresql-ha-client
dor1@is-master ~/postgresql-ha ❯ vi postgresql-ha-client.yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgresql-ha-client
  namespace: default
spec:
  containers:
  - name: postgresql-ha-client
    image: docker.io/bitnami/postgresql-repmgr:15.2.0-debian-11-r5
    command: ["bash"]
    args: ["-c", "while true; do sleep 10; done"]
    stdin: true
    tty: true
    terminationMessagePolicy: "FallbackToLogsOnError"
```

```shell
dor1@is-master ~/postgresql-ha ❯ kubectl apply -f ./postgresql-ha-client.yaml
pod/postgresql-ha-client created

# 확인
dor1@is-master ~/postgresql-ha ❯ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
postgresql-ha-client       1/1     Running   0          93s
```

접근은 쉽게 exec를 사용하여 해 주면 된다.  
이미지 특성 상 `username`이 존재하지는 않아 history가 남지는 않는다.  
이걸 생각해보면 어쩌면 다른 이미지로 `psql` 사용이 가능하게 끔 하여 만들 수도 있을 것으로 생각이 든다.

```shell
# exec로 접근
dor1@is-master ~/postgresql-ha ❯ kubectl exec -it postgresql-ha-client -- bash
I have no name!@postgresql-ha-client:/$

# psql로 DB 접속
I have no name!@postgresql-ha-client:/$ psql -h postgresql-ha-pgpool.postgresql-ha.svc.cluster.local -p 5432 -U deepadmin
Password for user deepadmin:
psql (15.2)
Type "help" for help.

deepadmin=> \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 deepadmin | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 repmgr    | Superuser, Replication                                     | {}

deepadmin=>
```

## Superuser 권한 주기
---

`postgres`의 기본 유저가 아닌 parameter를 통하여 유저를 따로 사용하게 되면 SUPERUSER 권한이 따로 없다.  
만약 `pgadmin4` 같은 곳에서 session에 대해 stop(kill) 시켜야 되는 상황이라면 권한의 문제로 인하여 안된다.  
권한 수정을 위해 `postgresql-ha-client`를 통하여 Superuser 권한이 있는 `postgres`계정을 접근하고 싶어도 다음과 같은 error가 발생하여 접근이 안된다.

```shell
I have no name!@postgresql-ha-client:/$ psql -h postgresql-ha-pgpool.postgresql-ha.svc.cluster.local -p 5432 -U postgres
psql: error: connection to server at "postgresql-ha-pgpool.postgresql-ha.svc.cluster.local" (10.233.41.96), port 5432 failed: FATAL:  SCRAM authentication failed
DETAIL:  pool_passwd file does not contain an entry for "postgres"
```

이럴 경우엔 HA 구성에 따라 1번 pod의 DB를 접근하여 계정에 대하여 role 권한을 수정해주면 된다.

```shell
dor1@is-master ~/postgresql-ha ❯ kubectl exec -it -n postgresql-ha postgresql-ha-postgresql-0 -- bash

I have no name!@postgresql-ha-postgresql-0:/$ psql -U postgres
Password for user postgres:
psql (15.2)
Type "help" for help.

postgres=# ALTER ROLE deepadmin SUPERUSER;
ALTER ROLE
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 deepadmin | Superuser, Create DB                                       | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 repmgr    | Superuser, Replication                                     | {}

postgres=
```

권한이 수정 되면 HA 구성에 따라 자동으로 다른 postgres pod에도 설정이 된다.