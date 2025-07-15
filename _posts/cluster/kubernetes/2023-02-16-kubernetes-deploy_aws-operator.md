---
title: Kubernetes - awx-operator 배포해 보기
date: 2023-02-16 10:01:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, awx-operator]
description: Kubernetes에서 Ansible을 GUI로 관리하는 aws-operator를 배포해 보았다.
---

>Kubernetes v1.24.7 / awx-operator v1.2.0 / Ubuntu 22.04 LTS
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* `AWX`는 Ansible에서 만든 GUI를 통한 automation deployment & management tool 이다.
* `AWX`의 약자는 **A**nsible **W** or **X**를 의미하며 전체적으로 풀어보면 **A**nsible **W**eb e**X**ecutable이라고 한다.
* Ansible tower에서 좀 더 진화한 형태로 평가된다.
* `AWX`은 단일(standalone) 형태로 사용(보통은 container로 사용)이 가능하지만, `awx-operator`를 이용하여 k8s에서도 구축이 가능하다.
* 설치 방법은 2가지를 제공하고 있으며 `kustomize`를 이용한 방법 과 `helm`을 이용한 방법이다.

## Deploy via kustomize
---

`kustomize`를 사용하게 되면 기본적으로 build 후 deployment를 한다.  
<https://github.com/ansible/awx-operator/blob/devel/README.md#basic-install>  
`kustomize`는 file로 이뤄진 config 값(`deployment`, `service`등)들을 하나의 file에서 관리할 수 있게 만들어진 configuration management solution이다.  
<https://kubectl.docs.kubernetes.io/installation/kustomize/>  
우선, `kustomize`를 설치 후 `Basic Install`항목을 따라하여 진행한 뒤 다음 항목에서 설정 값들을 추가해준다.

```yaml
# kustomize 설치(binaries)
dor1@is-master:~$ curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
v5.0.0
kustomize installed to /home/dor1/kustomize
dor1@is-master:~$ sudo mv ./kustomize /usr/local/bin

# Source
dor1@is-master:~$ git clone https://github.com/ansible/awx-operator.git
Cloning into 'awx-operator'...
remote: Enumerating objects: 7994, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 7994 (delta 4), reused 12 (delta 4), pack-reused 7976
Receiving objects: 100% (7994/7994), 2.17 MiB | 5.73 MiB/s, done.
Resolving deltas: 100% (4581/4581), done.

# AWX operator deployment(v1.2.0)
dor1@is-master:~$ cd awx-operator
dor1@is-master:~/awx-operator$ vi kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=1.2.0

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 1.2.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

```shell
# kustomize build(Warning은 github의 source에서 deprecated 항목이 있어 발생)
dor1@is-master:~/awx-operator$ kustomize build . | kubectl apply -f -
# Warning: 'patchesStrategicMerge' is deprecated. Please use 'patches' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
namespace/awx unchanged
customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com unchanged
customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com unchanged
customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com unchanged
serviceaccount/awx-operator-controller-manager unchanged
role.rbac.authorization.k8s.io/awx-operator-awx-manager-role configured
role.rbac.authorization.k8s.io/awx-operator-leader-election-role unchanged
clusterrole.rbac.authorization.k8s.io/awx-operator-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/awx-operator-proxy-role unchanged
rolebinding.rbac.authorization.k8s.io/awx-operator-awx-manager-rolebinding unchanged
rolebinding.rbac.authorization.k8s.io/awx-operator-leader-election-rolebinding unchanged
clusterrolebinding.rbac.authorization.k8s.io/awx-operator-proxy-rolebinding unchanged
configmap/awx-operator-awx-manager-config unchanged
service/awx-operator-controller-manager-metrics-service unchanged
deployment.apps/awx-operator-controller-manager configured
awx.awx.ansible.com/awx created

# 확인
dor1@is-master:~/awx-operator$ kubectl get pod -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-65b875f4bf-rfjgl   2/2     Running   0          35m
```

`awx-operator`까지는 올라오고 실제 `AWX`를 사용하기 위해서는 내용을 추가해야 진행이 된다.

```yaml
# awx-demo(Basic Install)
dor1@is-master:~/awx-operator$ vi awx-demo.yml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport

dor1@is-master:~/awx-operator$ vi kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=1.2.0
  # awx-demo 추가
  - awx-demo.yml

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 1.2.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

```shell
# rebuild 후 다시 deployment
dor1@is-master:~/awx-operator$ kustomize build . | kubectl apply -f -
# Warning: 'patchesStrategicMerge' is deprecated. Please use 'patches' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
namespace/awx unchanged
customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com unchanged
customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com unchanged
customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com unchanged
serviceaccount/awx-operator-controller-manager unchanged
role.rbac.authorization.k8s.io/awx-operator-awx-manager-role configured
role.rbac.authorization.k8s.io/awx-operator-leader-election-role unchanged
clusterrole.rbac.authorization.k8s.io/awx-operator-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/awx-operator-proxy-role unchanged
rolebinding.rbac.authorization.k8s.io/awx-operator-awx-manager-rolebinding unchanged
rolebinding.rbac.authorization.k8s.io/awx-operator-leader-election-rolebinding unchanged
clusterrolebinding.rbac.authorization.k8s.io/awx-operator-proxy-rolebinding unchanged
configmap/awx-operator-awx-manager-config unchanged
service/awx-operator-controller-manager-metrics-service unchanged
deployment.apps/awx-operator-controller-manager configured
awx.awx.ansible.com/awx configured

# 확인
dor1@is-master:~/awx-operator$ kubectl get pod -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-fdb8cbf95-bwn9z                                4/4     Running   0          31s
awx-operator-controller-manager-65b875f4bf-rfjgl   2/2     Running   0          35s
awx-postgres-13-0                                  1/1     Running   0          33s

dor1@is-master:~/awx-operator$ kubectl get svc -n awx
NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
awx-operator-controller-manager-metrics-service   ClusterIP   10.233.18.94    <none>        8443/TCP       16m
awx-postgres-13                                   ClusterIP   None            <none>        5432/TCP       13m
awx-service                                       NodePort    10.233.17.249   <none>        80:30020/TCP   11m
```

초기 비밀번호는 다음과 같이 입력하여서 찾을 수 있다.

```bash
dor1@is-master:~/awx-operator$ kubectl get secret -n awx awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode; echo
tMFbJSwnD3zpCoMo6uXtI4PIxP2I9PbY
```

![AWX 메인 페이지](/assets/img/post/cluster/kubernetes/2023-02-16-kubernetes-deploy_aws-operator/1.png)
_AWX 메인 페이지_

### config 값 추가하기

detail한 config 값은 github에 다 나와있다.  
<https://github.com/ansible/awx-operator#advanced-configuration>  
현재 서비스를 올려서 사용 중인 구성 값은 다음과 같다.

1. `rook-ceph`으로 구성되어있는 storage를 사용하기 위하여 **DB**의`storageClass`를 `rook-cephfs`로 변경 및 용량 할당
2. `AWX`에서 사용 할 **project**의 storage값도 `rook-ceph`을 사용하기 위하여 `rook-cephfs`로 변경 및 용량 할당
3. `AWX`의 resource 사용량은 생각보다 높기 때문에 k8s에 부하를 덜어주기 위하여 resource 제한

config들은 awx-demo.yaml에 추가하여서 진행하면 된다.

```yaml
dor1@is-master:~/awx-operator$ vi kustomization.yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  # service NodePort
  service_type: NodePort
  nodeport_port: 30020
  # DB
  postgres_storage_class: rook-cephfs
  postgres_storage_requirements:
    requests:
      storage: 50Gi
  # project
  projects_persistence: true
  projects_storage_class: rook-cephfs
  projects_storage_access_mode: ReadWriteMany
  projects_storage_size: 50Gi
  # limit resource
  web_resource_requirements:
    requests:
      cpu: 250m
      memory: 2Gi
    limits:
      cpu: 1000m
      memory: 4Gi
  task_resource_requirements:
    requests:
      cpu: 250m
      memory: 1Gi
    limits:
      cpu: 2000m
      memory: 2Gi
  ee_resource_requirements:
    requests:
      cpu: 250m
      memory: 100Mi
    limits:
      cpu: 500m
      memory: 2Gi
```

## Deploy via helm
---

`helm`의 버전은 `kustomize`보다는 높게 나온다.  
<https://github.com/ansible/awx-operator/blob/devel/README.md#helm-install-on-existing-cluster>  
보다 쉽게 deployment가 가능하며 parameters에 config 값을 넣어주면 된다.

```shell
# vaules.yaml
dor1@is-master ~/awx-operator ❯ mkdir helm && cd helm
dor1@is-master ~/awx-operator/helm ❯ helm show values awx-operator/awx-operator > values.yaml
```

기본적으로 제공되는 parameters가 많이 없다.  
`spec` 이하에는 kustomize의 config와 똑같이 제공해주기 때문에 resource 이름인 `AWX.name` 부분과 `AWX.enabled`에 대해서 잘 신경 써주면 된다.  
`admin_password_secret`의 parameter는 `secret`을 제공하여 사용하고 싶다면 항목을 아예 비워 준 다음, `secret`을 만들 때 `metadata`의 `name`을 `<resourcename>.admin-password`으로 입력 해주면 자동으로 찾아준다.  
또한 외부의 DB를 사용한다면 `secret`을 `postgres_configuration_secret` 항목을 추가하여 제공해준다.

```yaml
# values.yaml
dor1@is-master ~/awx-operator/helm ❯ vi values.yaml
AWX:
  # enable use of awx-deploy template
  enabled: true
  name: awx
  spec:
    admin_user: deepadmin
    admin_email: deepadmin@deepnoid.com
    service_type: NodePort
    nodeport_port: 30200
    postgres_configuration_secret: awx-postgres-secrets
    projects_persistence: true
    projects_storage_class: rook-cephfs
    projects_storage_access_mode: ReadWriteMany
    projects_storage_size: 50Gi

  # configurations for external postgres instance
  #postgres:
    # enabled: false
    # host: Unset
    # port: 5678
    # dbName: Unset
    # username: admin
    # for secret management, pass in the password independently of this file
    # at the command line, use --set AWX.postgres.password
    # password: Unset
    # sslmode: prefer
    # type: unmanaged

# awx-secrets.yaml (admin password)
dor1@is-master ~/awx-operator/helm ❯ vi awx-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: awx-admin-password
  namespace: awx
type: Opaque
data:
  password: YW9vbmkzNjUhQA==

# awx-db-secrets.yaml (DB)
dor1@is-master ~/awx-operator/helm ❯ vi awx-db-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: awx-postgres-secrets
  namespace: awx
type: Opaque
stringData:
  host: postgresql-ha-pgpool.postgresql-ha.svc.cluster.local
  port: "5432"
  database: awx
  username: deepadmin
  sslmode: prefer
  type: unmanaged
data:
  password: YW9vbmkzNjUhQA==
```

다 되었다면 deploy.

```shell
# create namespace "awx"
dor1@is-master ~/awx-operator/helm ❯ kubectl create ns awx
namespace/awx created

# secrets
dor1@is-master ~/awx-operator/helm ❯ kubectl apply -f ./awx-secrets.yaml -f ./awx-db-secrets.yaml
secret/awx-admin-password created
secret/awx-postgres-secrets created

# helm install
dor1@is-master ~/awx-operator/helm ❯ helm install awx-operator awx-operator/awx-operator -n awx -f ./values.yaml
NAME: awx-operator
LAST DEPLOYED: Thu Mar 16 16:00:06 2023
NAMESPACE: awx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWX Operator installed with Helm Chart version 1.3.0
```