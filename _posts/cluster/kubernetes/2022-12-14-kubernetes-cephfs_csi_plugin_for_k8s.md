---
title: Kubernetes - CephFS CSI plugin for k8s
date: 2022-12-14 15:48:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, cephfs, csi]
description: Kubernetes에서 CephFS의 CSI를 사용해서 StorageClass를 사용해보았다.
---

>Kubernetes v1.22.16 / v17.2.5 (quincy) / Ubuntu 20.04 LTS
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요 및 요구 사항
---

* 기존에는 RBD(RADOS Block Device)를 사용하여 k8s에 사용을 많이 했었지만, CSI(Container Storage Interface)가 보편화 되면서 CSI로 넘어가는 추세이다.
* CSI는 대다수의 플랫폼들을 지원하고 있다.
  <https://kubernetes-csi.github.io/docs/drivers.html>
* 아직까지 [Ceph official page](https://docs.ceph.com/en/latest/rbd/rbd-kubernetes/#configure-ceph-csi)에는 k8s에 deployment 시 RBD를 이용한 방법이 나와있지만 Github에는 Only cephfs-csi plugin으로만 deployment 하는 방법이 나와있다.
* Kubernetes v1.14+ 이상이 필요하다.

> ref.
> - <https://github.com/ceph/ceph-csi/blob/devel/docs/deploy-cephfs.md>

## CephFS CSI plugin을 이용하여 deployment 하기
---

Github에서 `git clone https://github.com/ceph/ceph-csi.git`으로 먼저 cloning 해 준 뒤에 시작한다.  
네임스페이스(NameSpace)를 만들어 따로 분류 해 준다.

**Testbed**
- is-m1 (Ceph cluster master)
- is-n1 (Ceph cluster worker)

```shell
# Plugin deployment까지는 cephfs에서 진행 함
dor1@is-m1:~$ cd ceph-csi/deploy/cephfs/kubernetes
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ ll
total 18
drwxrwxr-x 2 dor1 dor1    6 Dec 14 15:21 .
drwxrwxr-x 3 dor1 dor1    1 Dec 14 15:21 ..
-rw-rw-r-- 1 dor1 dor1 5955 Dec 14 15:21 csi-cephfsplugin-provisioner.yaml
-rw-rw-r-- 1 dor1 dor1 6591 Dec 14 15:21 csi-cephfsplugin.yaml
-rw-rw-r-- 1 dor1 dor1  100 Dec 14 15:21 csi-config-map.yaml
-rw-rw-r-- 1 dor1 dor1  164 Dec 14 15:21 csidriver.yaml
-rw-rw-r-- 1 dor1 dor1  846 Dec 14 15:21 csi-nodeplugin-rbac.yaml
-rw-rw-r-- 1 dor1 dor1 3000 Dec 14 15:21 csi-provisioner-rbac.yaml

# 순서에 맞춰 정리
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ mv csidriver.yaml 00-csidriver.yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ mv csi-provisioner-rbac.yaml 01-csi-provisioner-rbac.yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ mv csi-nodeplugin-rbac.yaml 02-csi-nodeplugin-rbac.yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ mv csi-config-map.yaml 03-csi-config-map.yaml

dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cp ../../../examples/ceph-conf.yaml 04-ceph-conf.yaml

dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ mv csi-cephfsplugin-provisioner.yaml 05-csi-cephfsplugin-provisioner.yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ mv csi-cephfsplugin.yaml 06-csi-cephfsplugin.yaml

# 네임스페이스 분류하기 위하여 추가
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl create namespace cephfs-csi
namespace/cephfs-csi created
```

### 1. Deploy CSI Driver

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 00-csidriver.yaml
---
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: cephfs.csi.ceph.com
spec:
  attachRequired: false
  podInfoOnMount: false
  fsGroupPolicy: File
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 00-csidriver.yaml
csidriver.storage.k8s.io/cephfs.csi.ceph.com created

# 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get csidrivers.storage.k8s.io
NAME                  ATTACHREQUIRED   PODINFOONMOUNT   STORAGECAPACITY   TOKENREQUESTS   REQUIRESREPUBLISH   MODES        AGE
cephfs.csi.ceph.com   false            false            false             <unset>         false               Persistent   48s
```

### 2. Deploy CSI RBAC Provisioner

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 01-csi-provisioner-rbac.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cephfs-csi-provisioner
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-external-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "update", "delete", "patch"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims/status"]
    verbs: ["update", "patch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshots"]
    verbs: ["get", "list"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshots/status"]
    verbs: ["get", "list", "patch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshotcontents"]
    verbs: ["get", "list", "watch", "update", "patch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshotclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["csinodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["snapshot.storage.k8s.io"]
    resources: ["volumesnapshotcontents/status"]
    verbs: ["update", "patch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["serviceaccounts/token"]
    verbs: ["create"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-csi-provisioner-role
subjects:
  - kind: ServiceAccount
    name: cephfs-csi-provisioner
    # 네임스페이스 변경을 통해 분류
    namespace: cephfs-csi
roleRef:
  kind: ClusterRole
  name: cephfs-external-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # replace with non-default namespace name
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
  name: cephfs-external-provisioner-cfg
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["get", "watch", "list", "delete", "update", "create"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-csi-provisioner-role-cfg
  # replace with non-default namespace name
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
subjects:
  - kind: ServiceAccount
    name: cephfs-csi-provisioner
    # replace with non-default namespace name
    # 네임스페이스 변경을 통해 분류
    namespace: cephfs-csi
roleRef:
  kind: Role
  name: cephfs-external-provisioner-cfg
  apiGroup: rbac.authorization.k8s.io
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 01-csi-provisioner-rbac.yaml
serviceaccount/cephfs-csi-provisioner created
clusterrole.rbac.authorization.k8s.io/cephfs-external-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/cephfs-csi-provisioner-role created
role.rbac.authorization.k8s.io/cephfs-external-provisioner-cfg created
rolebinding.rbac.authorization.k8s.io/cephfs-csi-provisioner-role-cfg created

# 확인 (ClusterRole, ClusterRoleBinding은 네임스페이스가 따로 없음)
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get sa,role,rolebinding -n cephfs-csi
NAME                                    SECRETS   AGE
serviceaccount/cephfs-csi-provisioner   1         55s
serviceaccount/default                  1         111m

NAME                                                             CREATED AT
role.rbac.authorization.k8s.io/cephfs-external-provisioner-cfg   2022-12-14T08:13:32Z

NAME                                                                    ROLE                                   AGE
rolebinding.rbac.authorization.k8s.io/cephfs-csi-provisioner-role-cfg   Role/cephfs-external-provisioner-cfg   55s
```

### 3. Deploy CSI RBAC Node plugin

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 02-csi-nodeplugin-rbac.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cephfs-csi-nodeplugin
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-csi-nodeplugin
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["serviceaccounts/token"]
    verbs: ["create"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-csi-nodeplugin
subjects:
  - kind: ServiceAccount
    name: cephfs-csi-nodeplugin
    # replace with non-default namespace name
    # 네임스페이스 변경을 통해 분류
    namespace: cephfs-csi
roleRef:
  kind: ClusterRole
  name: cephfs-csi-nodeplugin
  apiGroup: rbac.authorization.k8s.io
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 02-csi-nodeplugin-rbac.yaml
serviceaccount/cephfs-csi-nodeplugin created
clusterrole.rbac.authorization.k8s.io/cephfs-csi-nodeplugin created
clusterrolebinding.rbac.authorization.k8s.io/cephfs-csi-nodeplugin created

# 확인 (ClusterRole, ClusterRoleBinding은 네임스페이스가 따로 없음)
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get sa -n cephfs-csi
NAME                     SECRETS   AGE
cephfs-csi-nodeplugin    1         41s
cephfs-csi-provisioner   1         1m12s
default                  1         114m
```

### 4. Deploy CSI config map

json 양식에 맞춰 작성 시 다음의 링크 참조하여 작성 진행  
<https://github.com/ceph/ceph-csi/blob/devel/examples/csi-config-map-sample.yaml>

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 03-csi-config-map.yaml
---
apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    [
      {
        # ceph fsid를 통해 clusterID 조회 
        "clusterID": "b63db2e8-75d9-11ed-9764-f30dafc52de6",
        # ceph mon dump를 통해 Internal IP 조회
        "monitors": [
          "172.16.1.111:6789",
          "172.16.1.112:6789",
          "172.16.1.113:6789",
          "172.16.1.114:6789"
        ],
        # ceph fs ls를 통해 대상 Filesystem 조회
        "cephFS": {
          "subvolumeGroup": "cephfs"
        }
      }
    ]
metadata:
  name: ceph-csi-config
  # 네임스페이스 추가를 통해 분류
  namespace: cephfs-csi
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 03-csi-config-map.yaml
configmap/ceph-csi-config created

# 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get configmap -n cephfs-csi 
NAME               DATA   AGE
ceph-csi-config    1      21s
kube-root-ca.crt   1      7d6h
```

### 5. Deploy Ceph config

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 04-ceph-conf.yaml
---
# This is a sample configmap that helps define a Ceph configuration as required
# by the CSI plugins.

# Sample ceph.conf available at
# https://github.com/ceph/ceph/blob/master/src/sample.ceph.conf Detailed
# documentation is available at
# https://docs.ceph.com/en/latest/rados/configuration/ceph-conf/
apiVersion: v1
kind: ConfigMap
data:
  ceph.conf: |
    [global]
    auth_cluster_required = cephx
    auth_service_required = cephx
    auth_client_required = cephx

  # keyring is a required key and its value should be empty
  keyring: |
metadata:
  name: ceph-config
  # 네임스페이스 추가를 통해 분류
  namespace: cephfs-csi
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 04-ceph-conf.yaml
configmap/ceph-config created

# 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get configmap -n cephfs-csi 
NAME               DATA   AGE
ceph-config        2      18s
ceph-csi-config    1      6m
kube-root-ca.crt   1      7d6h
```

### 6. Deploy CSI CephFS Plugin provisioner

KMS는 사용하지 않으므로 주석 처리 해준다.

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 05-csi-cephfsplugin-provisioner.yaml
---
kind: Service
apiVersion: v1
metadata:
  name: csi-cephfsplugin-provisioner
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
  labels:
    app: csi-metrics
spec:
  selector:
    app: csi-cephfsplugin-provisioner
  ports:
    - name: http-metrics
      port: 8080
      protocol: TCP
      targetPort: 8681
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: csi-cephfsplugin-provisioner
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
spec:
  selector:
    matchLabels:
      app: csi-cephfsplugin-provisioner
  replicas: 1
  template:
    metadata:
      labels:
        app: csi-cephfsplugin-provisioner
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - csi-cephfsplugin-provisioner
              topologyKey: "kubernetes.io/hostname"
      serviceAccountName: cephfs-csi-provisioner
      priorityClassName: system-cluster-critical
      containers:
        - name: csi-provisioner
          image: registry.k8s.io/sig-storage/csi-provisioner:v3.3.0
          args:
            - "--csi-address=$(ADDRESS)"
            - "--v=1"
            - "--timeout=150s"
            - "--leader-election=true"
            - "--retry-interval-start=500ms"
            - "--feature-gates=Topology=false"
            - "--feature-gates=HonorPVReclaimPolicy=true"
            - "--prevent-volume-mode-conversion=true"
            - "--extra-create-metadata=true"
          env:
            - name: ADDRESS
              value: unix:///csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-resizer
          image: registry.k8s.io/sig-storage/csi-resizer:v1.6.0
          args:
            - "--csi-address=$(ADDRESS)"
            - "--v=1"
            - "--timeout=150s"
            - "--leader-election"
            - "--retry-interval-start=500ms"
            - "--handle-volume-inuse-error=false"
            - "--feature-gates=RecoverVolumeExpansionFailure=true"
          env:
            - name: ADDRESS
              value: unix:///csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-snapshotter
          image: registry.k8s.io/sig-storage/csi-snapshotter:v6.1.0
          args:
            - "--csi-address=$(ADDRESS)"
            - "--v=1"
            - "--timeout=150s"
            - "--leader-election=true"
            - "--extra-create-metadata=true"
          env:
            - name: ADDRESS
              value: unix:///csi/csi-provisioner.sock
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
        - name: csi-cephfsplugin
          # for stable functionality replace canary with latest release version
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--nodeid=$(NODE_ID)"
            - "--type=cephfs"
            - "--controllerserver=true"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--v=5"
            - "--drivername=cephfs.csi.ceph.com"
            - "--pidlimit=-1"
            - "--enableprofiling=false"
            - "--setmetadata=true"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: NODE_ID
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: CSI_ENDPOINT
              value: unix:///csi/csi-provisioner.sock
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          # - name: KMS_CONFIGMAP_NAME
          #   value: encryptionConfig
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: host-sys
              mountPath: /sys
            - name: lib-modules
              mountPath: /lib/modules
              readOnly: true
            - name: host-dev
              mountPath: /dev
            - name: ceph-config
              mountPath: /etc/ceph/
            - name: ceph-csi-config
              mountPath: /etc/ceph-csi-config/
            - name: keys-tmp-dir
              mountPath: /tmp/csi/keys
            # KMS 미사용으로 주석처리
            #- name: ceph-csi-encryption-kms-config
            #  mountPath: /etc/ceph-csi-encryption-kms-config/
        - name: liveness-prometheus
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--type=liveness"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--metricsport=8681"
            - "--metricspath=/metrics"
            - "--polltime=60s"
            - "--timeout=3s"
          env:
            - name: CSI_ENDPOINT
              value: unix:///csi/csi-provisioner.sock
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
          imagePullPolicy: "IfNotPresent"
      volumes:
        - name: socket-dir
          emptyDir: {
            medium: "Memory"
          }
        - name: host-sys
          hostPath:
            path: /sys
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: host-dev
          hostPath:
            path: /dev
        - name: ceph-config
          configMap:
            name: ceph-config
        - name: ceph-csi-config
          configMap:
            name: ceph-csi-config
        - name: keys-tmp-dir
          emptyDir: {
            medium: "Memory"
          }
        # KMS 미사용으로 주석처리
        #- name: ceph-csi-encryption-kms-config
        #  configMap:
        #    name: ceph-csi-encryption-kms-config
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 05-csi-cephfsplugin-provisioner.yaml
service/csi-cephfsplugin-provisioner created
deployment.apps/csi-cephfsplugin-provisioner created

# services, deployment 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get services,deployment -n cephfs-csi
NAME                                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/csi-cephfsplugin-provisioner   ClusterIP   10.233.18.106   <none>        8080/TCP   116s
service/csi-metrics-cephfsplugin       ClusterIP   10.233.30.107   <none>        8080/TCP   116s

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/csi-cephfsplugin-provisioner   0/1     1            0           116s

# pod 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get pod -n cephfs-csi
NAME                                           READY   STATUS    RESTARTS   AGE
csi-cephfsplugin-provisioner-5f4bf6fd4-vtznd   5/5     Running   0          1m20s
```

### 7. Deploy CSI CephFS Plugin

```yaml
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ cat 06-csi-cephfsplugin.yaml
---
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: csi-cephfsplugin
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
spec:
  selector:
    matchLabels:
      app: csi-cephfsplugin
  template:
    metadata:
      labels:
        app: csi-cephfsplugin
    spec:
      serviceAccountName: cephfs-csi-nodeplugin
      priorityClassName: system-node-critical
      hostNetwork: true
      hostPID: true
      # to use e.g. Rook orchestrated cluster, and mons' FQDN is
      # resolved through k8s service, set dns policy to cluster first
      dnsPolicy: ClusterFirstWithHostNet
      containers:
        - name: driver-registrar
          # This is necessary only for systems with SELinux, where
          # non-privileged sidecar containers cannot access unix domain socket
          # created by privileged CSI driver container.
          securityContext:
            privileged: true
            allowPrivilegeEscalation: true
          image: registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.6.2
          args:
            - "--v=1"
            - "--csi-address=/csi/csi.sock"
            - "--kubelet-registration-path=/var/lib/kubelet/plugins/cephfs.csi.ceph.com/csi.sock"
          env:
            - name: KUBE_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: registration-dir
              mountPath: /registration
        - name: csi-cephfsplugin
          securityContext:
            privileged: true
            capabilities:
              add: ["SYS_ADMIN"]
            allowPrivilegeEscalation: true
          # for stable functionality replace canary with latest release version
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--nodeid=$(NODE_ID)"
            - "--type=cephfs"
            - "--nodeserver=true"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--v=5"
            - "--drivername=cephfs.csi.ceph.com"
            - "--enableprofiling=false"
            # If topology based provisioning is desired, configure required
            # node labels representing the nodes topology domain
            # and pass the label names below, for CSI to consume and advertise
            # its equivalent topology domain
            # - "--domainlabels=failure-domain/region,failure-domain/zone"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: NODE_ID
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          # - name: KMS_CONFIGMAP_NAME
          #   value: encryptionConfig
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
            - name: mountpoint-dir
              mountPath: /var/lib/kubelet/pods
              mountPropagation: Bidirectional
            - name: plugin-dir
              mountPath: /var/lib/kubelet/plugins
              mountPropagation: "Bidirectional"
            - name: host-sys
              mountPath: /sys
            - name: etc-selinux
              mountPath: /etc/selinux
              readOnly: true
            - name: lib-modules
              mountPath: /lib/modules
              readOnly: true
            - name: host-dev
              mountPath: /dev
            - name: host-mount
              mountPath: /run/mount
            - name: ceph-config
              mountPath: /etc/ceph/
            - name: ceph-csi-config
              mountPath: /etc/ceph-csi-config/
            - name: keys-tmp-dir
              mountPath: /tmp/csi/keys
            - name: ceph-csi-mountinfo
              mountPath: /csi/mountinfod
            # KMS 미사용으로 주석처리
            #- name: ceph-csi-encryption-kms-config
            #  mountPath: /etc/ceph-csi-encryption-kms-config/
        - name: liveness-prometheus
          securityContext:
            privileged: true
            allowPrivilegeEscalation: true
          image: quay.io/cephcsi/cephcsi:canary
          args:
            - "--type=liveness"
            - "--endpoint=$(CSI_ENDPOINT)"
            - "--metricsport=8681"
            - "--metricspath=/metrics"
            - "--polltime=60s"
            - "--timeout=3s"
          env:
            - name: CSI_ENDPOINT
              value: unix:///csi/csi.sock
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: socket-dir
              mountPath: /csi
          imagePullPolicy: "IfNotPresent"
      volumes:
        - name: socket-dir
          hostPath:
            path: /var/lib/kubelet/plugins/cephfs.csi.ceph.com/
            type: DirectoryOrCreate
        - name: registration-dir
          hostPath:
            path: /var/lib/kubelet/plugins_registry/
            type: Directory
        - name: mountpoint-dir
          hostPath:
            path: /var/lib/kubelet/pods
            type: DirectoryOrCreate
        - name: plugin-dir
          hostPath:
            path: /var/lib/kubelet/plugins
            type: Directory
        - name: host-sys
          hostPath:
            path: /sys
        - name: etc-selinux
          hostPath:
            path: /etc/selinux
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: host-dev
          hostPath:
            path: /dev
        - name: host-mount
          hostPath:
            path: /run/mount
        - name: ceph-config
          configMap:
            name: ceph-config
        - name: ceph-csi-config
          configMap:
            name: ceph-csi-config
        - name: keys-tmp-dir
          emptyDir: {
            medium: "Memory"
          }
        - name: ceph-csi-mountinfo
          hostPath:
            path: /var/lib/kubelet/plugins/cephfs.csi.ceph.com/mountinfo
            type: DirectoryOrCreate
        # KMS 미사용으로 주석처리
        #- name: ceph-csi-encryption-kms-config
        #  configMap:
        #    name: ceph-csi-encryption-kms-config
---
# This is a service to expose the liveness metrics
apiVersion: v1
kind: Service
metadata:
  name: csi-metrics-cephfsplugin
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
  labels:
    app: csi-metrics
spec:
  ports:
    - name: http-metrics
      port: 8080
      protocol: TCP
      targetPort: 8681
  selector:
    app: csi-cephfsplugin
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl apply -f 06-csi-cephfsplugin.yaml
daemonset.apps/csi-cephfsplugin created
service/csi-metrics-cephfsplugin created

# daemonset, services 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get daemonset,services -n cephfs-csi
NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/csi-cephfsplugin   1         1         0       1            0           <none>          43s

NAME                                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/csi-cephfsplugin-provisioner   ClusterIP   10.233.31.128   <none>        8080/TCP   9m20s
service/csi-metrics-cephfsplugin       ClusterIP   10.233.45.121   <none>        8080/TCP   43s

# pod 확인
dor1@is-m1:~/ceph-csi/deploy/cephfs/kubernetes$ kubectl get pod -n cephfs-csi
NAMESPACE      NAME                                                              READY   STATUS      RESTARTS       AGE
cephfs-csi     csi-cephfsplugin-provisioner-5f4bf6fd4-vtznd                      5/5     Running     0              7m32s
cephfs-csi     csi-cephfsplugin-wch66                                            3/3     Running     0              39s
```

## StorageClass 생성 후 deployment 하기
---

위의 항목들이 정상적으로 다 적용이 되어있어야 k8s API server에서 접근이 가능하다.  
<https://github.com/ceph/ceph-csi/blob/devel/examples/README.md#deploying-the-storage-class>  
`kubectl create -f pod.yaml` 항목은 `pod`를 deployment하여 진행하는 테스트 파일인데, 제외 후 임의의 `pod`로 테스트 예정이다.

```shell
# StorageClass 관련 내용은 ceph-csi/examples/cephfs 있는 내용으로 진행 함
dor1@is-m1:~$ cd ceph-csi/examples/cephfs

dor1@is-m1:~/ceph-csi/examples/cephfs$ ll
total 13
drwxrwxr-x 2 dor1 dor1   18 Dec 14 15:21 .
drwxrwxr-x 6 dor1 dor1    8 Dec 14 15:21 ..
-rw-rw-r-- 1 dor1 dor1  577 Dec 14 15:21 deployment.yaml
-rwxrwxr-x 1 dor1 dor1  398 Dec 14 15:21 exec-bash.sh
-rwxrwxr-x 1 dor1 dor1  387 Dec 14 15:21 logs.sh
-rwxrwxr-x 1 dor1 dor1  351 Dec 14 15:21 plugin-deploy.sh
-rwxrwxr-x 1 dor1 dor1  351 Dec 14 15:21 plugin-teardown.sh
-rw-rw-r-- 1 dor1 dor1  359 Dec 14 15:21 pod-clone.yaml
-rw-rw-r-- 1 dor1 dor1  502 Dec 14 15:21 pod-ephemeral.yaml
-rw-rw-r-- 1 dor1 dor1  363 Dec 14 15:21 pod-restore.yaml
-rw-rw-r-- 1 dor1 dor1  356 Dec 14 15:21 pod-rwop.yaml
-rw-rw-r-- 1 dor1 dor1  346 Dec 14 15:21 pod.yaml
-rw-rw-r-- 1 dor1 dor1  274 Dec 14 15:21 pvc-clone.yaml
-rw-rw-r-- 1 dor1 dor1  312 Dec 14 15:21 pvc-restore.yaml
-rw-rw-r-- 1 dor1 dor1  209 Dec 14 15:21 pvc-rwop.yaml
-rw-rw-r-- 1 dor1 dor1  201 Dec 14 15:21 pvc.yaml
-rw-rw-r-- 1 dor1 dor1  424 Dec 14 15:21 secret.yaml
-rw-rw-r-- 1 dor1 dor1  916 Dec 14 15:21 snapshotclass.yaml
-rw-rw-r-- 1 dor1 dor1  218 Dec 14 15:21 snapshot.yaml
-rw-rw-r-- 1 dor1 dor1 2652 Dec 14 15:21 storageclass.yaml

# 순서에 맞춰 정리
dor1@is-m1:~/ceph-csi/examples/cephfs$ mv secret.yaml 01-secret.yaml
dor1@is-m1:~/ceph-csi/examples/cephfs$ mv storageclass.yaml 02-storageclass.yaml
dor1@is-m1:~/ceph-csi/examples/cephfs$ mv pvc.yaml 03-pvc.yaml
```

### 1. Deploy CSI CephFS Secret key

```yaml
# 현재 Testbed의 Ceph에는 admin 외에 아무것도 없기 때문에 admin keyring만 입력함
dor1@is-m1:~/ceph-csi/examples/cephfs$ cat 01-secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: csi-cephfs-secret
  # 네임스페이스 변경을 통해 분류
  namespace: cephfs-csi
stringData:
  # Required for statically provisioned volumes
  #userID: <plaintext ID>
  #userKey: <Ceph auth key corresponding to ID above>

  # Required for dynamically provisioned volumes
  adminID: admin
  adminKey: AQBn/49j/E1mOxAAS/D+H1IdcPIVCdhxlSg8Wg==

  # Encryption passphrase
  #encryptionPassphrase: test_passphrase
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl apply -f 01-secret.yaml
secret/csi-cephfs-secret created

# 확인
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl get secret -n cephfs-csi
NAME                                 TYPE                                  DATA   AGE
cephfs-csi-nodeplugin-token-7v6cb    kubernetes.io/service-account-token   3      19h
cephfs-csi-provisioner-token-cnmfl   kubernetes.io/service-account-token   3      19h
csi-cephfs-secret                    Opaque                                2      2m45s
default-token-7thf7                  kubernetes.io/service-account-token   3      20h
```

### 2. Deploy StorageClass

```yaml
dor1@is-m1:~/ceph-csi/examples/cephfs$ cat 02-storageclass.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-cephfs-sc
provisioner: cephfs.csi.ceph.com
parameters:
  # (required) String representing a Ceph cluster to provision storage from.
  # Should be unique across all Ceph clusters in use for provisioning,
  # cannot be greater than 36 bytes in length, and should remain immutable for
  # the lifetime of the StorageClass in use.
  # Ensure to create an entry in the configmap named ceph-csi-config, based on
  # csi-config-map-sample.yaml, to accompany the string chosen to
  # represent the Ceph cluster in clusterID below
  # ceph fsid를 통해 clusterID 조회
  clusterID: b63db2e8-75d9-11ed-9764-f30dafc52de6

  # (required) CephFS filesystem name into which the volume shall be created
  # eg: fsName: myfs
  # ceph fs ls를 통해 대상 Filesystem 조회
  fsName: cephfs

  # (optional) Ceph pool into which volume data shall be stored
  # pool: <cephfs-data-pool>

  # (optional) Comma separated string of Ceph-fuse mount options.
  # For eg:
  # fuseMountOptions: debug

  # (optional) Comma separated string of Cephfs kernel mount options.
  # Check man mount.ceph for mount options. For eg:
  # kernelMountOptions: readdir_max_bytes=1048576,norbytes

  # The secrets have to contain user and/or Ceph admin credentials.
  csi.storage.k8s.io/provisioner-secret-name: csi-cephfs-secret
  # 네임스페이스 변경을 통해 분류
  csi.storage.k8s.io/provisioner-secret-namespace: cephfs-csi
  csi.storage.k8s.io/controller-expand-secret-name: csi-cephfs-secret
  # 네임스페이스 변경을 통해 분류
  csi.storage.k8s.io/controller-expand-secret-namespace: cephfs-csi
  csi.storage.k8s.io/node-stage-secret-name: csi-cephfs-secret
  # 네임스페이스 변경을 통해 분류
  csi.storage.k8s.io/node-stage-secret-namespace: cephfs-csi

  # (optional) The driver can use either ceph-fuse (fuse) or
  # ceph kernelclient (kernel).
  # If omitted, default volume mounter will be used - this is
  # determined by probing for ceph-fuse and mount.ceph
  # mounter: kernel

  # (optional) Prefix to use for naming subvolumes.
  # If omitted, defaults to "csi-vol-".
  # volumeNamePrefix: "foo-bar-"

  # (optional) Boolean value. The PVC shall be backed by the CephFS snapshot
  # specified in its data source. `pool` parameter must not be specified.
  # (defaults to `false`)
  # backingSnapshot: "true"

  # (optional) Instruct the plugin it has to encrypt the volume
  # By default it is disabled. Valid values are "true" or "false".
  # A string is expected here, i.e. "true", not true.
  # encrypted: "true"

  # (optional) Use external key management system for encryption passphrases by
  # specifying a unique ID matching KMS ConfigMap. The ID is only used for
  # correlation to configmap entry.
  # encryptionKMSID: <kms-config-id>

reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  - debug
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl apply -f 02-storageclass.yaml
storageclass.storage.k8s.io/csi-cephfs-sc created

# 확인
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl get storageclass
NAME            PROVISIONER           RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
csi-cephfs-sc   cephfs.csi.ceph.com   Delete          Immediate           true                   7s
```

### 3. Deploy PVC(Persistent Volume Claim)

PVC 생성부터 입맛에 맞춰 쓰면 된다.  
아래에 적용하는 PVC는 Test PVC다.

```yaml
dor1@is-m1:~/ceph-csi/examples/cephfs$ cat 03-pvc.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: csi-cephfs-pvc
  # 네임스페이스 추가를 통해 분류
  namespace: cephfs-csi
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-cephfs-sc
```

```shell
# 적용
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl apply -f 09-pvc.yaml
persistentvolumeclaim/csi-cephfs-pvc created

# 확인
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl get pvc -n cephfs-csi
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
csi-cephfs-pvc   Bound    pvc-cdbf6ae4-98fa-4e56-b13a-ccb16393bef2   1Gi        RWX            csi-cephfs-sc   45s
```

### 4. Deploy Test pod

이미지는 [Docker Hub](https://hub.docker.com/)에 있는 [filebrowser](https://hub.docker.com/r/filebrowser/filebrowser)를 이용했으며, 앞서 만든 PVC를 `/src`에 mount 하였다.

```yaml
dor1@is-m1:~/ceph-csi/examples/cephfs$ cat 04-filebrowser.yaml
---
kind: Deployment
apiVersion: apps/v1
metadata:
  namespace: default
  name: homer
  labels:
    app: homer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: homer
  template:
    metadata:
      labels:
        app: homer
    spec:
      containers:
        - name: browser
          image: filebrowser/filebrowser
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: data1
              mountPath: /srv
      volumes:
        - name: data1
          persistentVolumeClaim:
            claimName: csi-cephfs-pvc
---
apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: browser-service
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
      nodePort: 31100
  selector:
    app: homer
```

```shell
# 적용 
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl apply -f 10-filebrowser.yaml
deployment.apps/homer created
service/browser-service created

# deployment, pod 확인
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl get deployment,pod
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/homer   1/1     1            1           25s

NAME                         READY   STATUS    RESTARTS   AGE
pod/homer-66b557b585-hz2kw   1/1     Running   0          25s

# pod 접속 후 mount 확인
dor1@is-m1:~/ceph-csi/examples/cephfs$ kubectl exec -it homer-66b557b585-hz2kw -- sh
/ $ df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                 502.4G     19.8G    457.0G   4% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                    11.7G         0     11.7G   0% /sys/fs/cgroup
172.16.1.111:6789,172.16.1.112:6789,172.16.1.113:6789,172.16.1.114:6789:/volumes/cephfs/csi-vol-2f22298f-e77f-4cd8-9a8d-4721e8c3a4ff/a78b43ad-fdfa-44b9-8648-c19b8a10c6d3
                          1.0G         0      1.0G   0% /srv
/dev/sda2               502.4G     19.8G    457.0G   4% /etc/hosts
/dev/sda2               502.4G     19.8G    457.0G   4% /dev/termination-log
/dev/sda2               502.4G     19.8G    457.0G   4% /etc/hostname
/dev/sda2               502.4G     19.8G    457.0G   4% /etc/resolv.conf
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                    21.8G     12.0K     21.8G   0% /run/secrets/kubernetes.io/serviceaccount
tmpfs                    11.7G         0     11.7G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                    11.7G         0     11.7G   0% /proc/scsi
tmpfs                    11.7G         0     11.7G   0% /sys/firmware
```