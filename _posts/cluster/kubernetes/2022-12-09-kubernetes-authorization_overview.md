---
title: Kubernetes - Authorization Overview
date: 2022-12-14 15:48:00 +0900
categories: [Cluster, Kubernetes]
tags: [cluster, kubernetes, cephfs, csi]
description: Kubernetes에서 CephFS의 CSI를 사용해서 StorageClass를 사용해보았다.
---

>Kubernetes v1.21.5
{: .prompt-info}

>Kubernetes
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 기본 개념
---

* Kubernetes의 권한 관리는 RBAC(Role-based access control), ABAC(Attribute-based access control)로 구성되어있다.
* RBAC는 대다수 플랫폼에서 많이 사용하는 권한 관리의 일종이며, 사용자와 역할을 별개로 미리 선언한 후 나중에 binding을 하여 사용자에게 권한을 부여해준다.
* ABAC는 속성(Attribute) 기반의 권한 관리이며, 일반적으로 사용자(User), 그룹(Group), 요청 경로(Request path), 요청 동사(Request verb) 외에도 네임스페이스(NameSpace), 리소스(Resource) 등등 각각의 속성 별로 설정이 가능하다.
* ABAC는 많이 사용하지 않으므로, RBAC로 사용하는 것을 권장한다.

## .kube

기본적으로 Kubenetes의 RBAC을 위한 config는`/home/<user>/.kube/config`에 들어가있다.  
`root`가 아니더라도 어떠한 계정에서 `kubectl`와 같은 명령어를 사용하기 위해서는 `.kube`가 꼭 있어야만 API server에 접근이 가능하다.

```yaml
# kubekey로 설치한 master node의 기본 config 값
dor1@is-m1:~$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1USXdOekF5TVRnMU1sb1hEVE15TVRJd05EQXlNVGcxTWxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTlBpCmJ1SjU3YisrTFNjTm1haVJYcnhwanBKcnBhTWQvc1pHbHpHV1RvallBVXBiaHhlRVhPdzRkbFBzMDhxb1R4T3MKTWt2Z002bXJVdEhQQkh0TmpIdjRMYmp6YytqUEoyclZmWVY2a0diZ0tEZkhpVUpxcGU1VHI5TVlJeThqSXV2RQpvVno4N2lVQzFVcExVdWdFc3lNQnB5Mi90VzR4eG93dXFLS1M3UlNsWE9jY0RvS1hXSzYzUHd1WWtjdUFZNlJkCndwd3JiM3NGQm1Cb255M2Q2aHI1Y1Jtd1FqV2NYa2hjZjJ6MXRDbW14Mk0wU1pMYVR0MmtlcWRXMHB1bWhIeXEKNDJVK1FiekJjU3pwSUEvRWNVcHpIQXo0MkZTV1BSWGkwMEY5cHRXU3NkTER3Y1N2M2t4SmQ3bEYzWjdmdGVEZQorRDlCQ3VXV1p2dXRtSzZPWFk4Q0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZFMW41cGRIVU1FZVNtL1pkditLRkloQXVtcVBNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCa1FrNDV3YVBrZDVvYjFENjNTYlg4WkJwOENEU0NDaXNheCtjKzl3L1dldTgxQUJzVgo2bW9yYVh1WENLT0IvZ0NKZVRDQWZPaGZLdkVXVU50TWhNYUs5OUV4R092YXM0QUtOZU9mb2NacjNONjRkeEdiCndxcXBOWWxBMFdVUnFzUGpraGxrTUFGUWZCUTcvZWxwQWx5NWxyUnprbVdRUS9wUHlRVk9QQWY4SHYvMmx2bVQKZUNpTWo4VGM5RnpTVDJ2MG0zT1RIUlhSY2FObm5ick5CYWFCZlpUaXdZQ0x5RmRDcERPYlQzbUlSNVltOE53Kwp2WXRqdmJCWWUrcCtZeEovU0N6SXBwYUFGcnN2K0pZMWI3SExsbG5BL3dkQjJCWTlHR0pGMzlDVEJjY0xEL3Z1CnF2ZG5DK1M4ZlJrNDBseGVUNE9NWklDcTdxVGVESkZPYTJSRQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://lb.kubesphere.local:6443
  name: cluster.local
contexts:
- context:
    cluster: cluster.local
    user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJWkpTSTNSZE14SVF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TWpFeU1EY3dNakU0TlRKYUZ3MHlNekV5TURjd01qRTROVE5hTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXRYRHJkNlJ1cVQwMW16a3kKWVNxbXFoWDNmSWVhUFpYd0hDbytYb2RsS1lRNVpSeHordEI4ODQ0T3VhcFZuZ1lYR1pvWGVYOU5YVG01RTJ3cApHbHVpQ0YrbDJEYjYxV3RHekJGMGRsVUkvK3hHcmkrTjNyQlZ0RWJzeHd0RVg1TzExYjlNekhmQ2xmT3VGWUloCkFYL0VvbER5WWxXRllVa2w5b0tGYkxmaWxDdjlxVE9EOTBoQmFEOGVtM2NoZFVvNnlSL0JPV29CTTBJdytHejgKalNjSmFrQTZ1Y0NMMDNISTlVSytXQk5FYmNBTi9OVjFwV01mOHVTNFY5K2JFd0NYYW5wYzhseDBJWnhiZWRicQo4VEFGTVI1NitRdFVaNWQ1ME9rYThvQnVQbkZIRklCSlJHZnpLZlNwSFp0NHpETWpmbU4wc3R0ejlER0RyU2RXCjFOUXVEd0lEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JSTlorYVhSMURCSGtwdjJYYi9paFNJUUxwcQpqekFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBVk9RZStEclppNmR6L0hYRUlvTnZEdzZURnJqTFREeFRtUlRSCk01U0M5ZG96alhQeXNIeThLT1ZJd2NoQ0Y2M0lwNC9qY1hSd01qNXJUK0lKMUJ6YXdNVjR5YzV2alFERjZFdisKNS9kUkFYUGpTTXdkdFZtSDV6NzRVZVdPOHMxL241Q2JCa0xnUFNXb2tiK3FQd3RkTDEzQ1djSm1UK1JHZ1BYdQpXZVVua3IvclUyYlo3Slp5MjBvZVppMExOcURtNmYxblB5WkhFUWhUSjROT25jTjFQcm9naVRnajRlWHo5Y05VCllrcWkvcFc4aVR3dzcwQi8vdGtONHk0d3dTT2dQZGRSWDN5QldmTm5JSzIyZjV2VnlPNUVScjdrVUY1M0tLQU0KaVZPd3pTMndCNWw5YmFoQkMvM3NZbHV3dEdSbjNWald2REZkVjFwY1ZZbTZCQWUxSnc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdFhEcmQ2UnVxVDAxbXpreVlTcW1xaFgzZkllYVBaWHdIQ28rWG9kbEtZUTVaUnh6Cit0Qjg4NDRPdWFwVm5nWVhHWm9YZVg5TlhUbTVFMndwR2x1aUNGK2wyRGI2MVd0R3pCRjBkbFVJLyt4R3JpK04KM3JCVnRFYnN4d3RFWDVPMTFiOU16SGZDbGZPdUZZSWhBWC9Fb2xEeVlsV0ZZVWtsOW9LRmJMZmlsQ3Y5cVRPRAo5MGhCYUQ4ZW0zY2hkVW82eVIvQk9Xb0JNMEl3K0d6OGpTY0pha0E2dWNDTDAzSEk5VUsrV0JORWJjQU4vTlYxCnBXTWY4dVM0VjkrYkV3Q1hhbnBjOGx4MElaeGJlZGJxOFRBRk1SNTYrUXRVWjVkNTBPa2E4b0J1UG5GSEZJQkoKUkdmektmU3BIWnQ0ekRNamZtTjBzdHR6OURHRHJTZFcxTlF1RHdJREFRQUJBb0lCQUROdkNURG5TZjlydkpCKwpERXdESFMvRi9sd3N6SXA4d0k0Ylk0YkVkdWJuOXFVMUJhT3FDbUc0ZVhBa1d4VHF3UTJlNHR5c083QWJ0dDFNCm9mSTQyNXZvRVVsVGZKT1hUNEIxeWovcEp4MzFTcXdDQ3dOL2xTdi9sd3R0cERvNzB5WCtqclMvbGtlUHhsK08KZmZEQTJXcng2MlA0dmxDdnZiVTlscmtVLzRQazRqQnpKQ284bUs4UEVwejBPZUdHTnYwWExYUHRYbUp0a3B6bApBQjNaWnVhNFo4YVdka1luTTJ4bGV6WnNEa0FFUEhrOXBWVU9oOHhIcXkzTmp2TkJraWdrZ3RIZ0pjc1QxUVkrCk1XN2l3VHhDMVY3L0hGS1ppZEZ4ZytydU42czNkRFk0a3Qzb1VobUhTRkg2SCtVRVNlR3dwT0J4SkZOZnpGd2oKRkJTMkVlRUNnWUVBNHJuN1VPL0JBcG1SdWJJdkhTaWxTdjVCb0pEaEk1TjhWeHd6RTdWUGYvayszU1hTSWsvaApGTmdjSkZabmN1bGFURnVLTTlvWHZUNzNXbzZPWFhlcXNSV2NvUUVvdEVZMTBQUEI4RXpEWEltbkU1TTJXWWN3CnpnZmVURHY2SnU1N2RDdU4vQ1RLUVZtTElhTjE3OGx1V2lTbFJTTFdLbHFWNzB2OHBLenZIS2tDZ1lFQXpONGQKUStlRmNQaG4zYnRNOEQxUjVKWFhTUXVaZ2VHai9sN0EwNnB1T01Yczl2WFJ2UW5KZ20zL2NlaW9RUVdXOFhzRApJRDZhOENCOXlSVXRyYlVmd09IUzNNbXJKRVJUSERyTTNFSENTTkRjSEg1VFlJd25WV1ovcXlDbmVyOENiU295ClRwUFljNGdoMlYzV1A1T2VhVTVhaGQwb3pSUkY5THAzbklaSXIvY0NnWUJKU0hINS9EUzNvV21meXY4OWZvakcKejUzb3gwdHVFMXJLVVR3Vkw3S05tOE44K1orTkphS0wrVHBIYUlJeGUwbUxpcjhGK1lWWXp3Um1pZE5zVktTZwpibXJkQTZIamV4b2ordFlCMU40RWlCMnZ6eEp2SjZwWHZlVlZZTUYvV2ZBZllZQ1lNbEFKaFdiYUxacU9NZDV3ClZvM3c1Y3l4amV3T2w5SUdiRHN4V1FLQmdRQ3hpeHlacUorQWxBYVBwcTY2MUttUUREdVMxamFtMU1HbXhMOGYKc09mczA3clZHNXcwMDdLTEVvRDZXc0xWOXQ0bFVKSVk4NmlheWMyNDRsMi8yT1EzNkgweFVxUzZ2V3U1WDB3Qwo1Z3BWeUl1NU5kRlVMcUkzNUtobnlkamJDNFl5elFya0JrVGpldXE2MGhQRzdVdXZ2M083NXpwZzRGendCbGw2CmtQV1ZhUUtCZ0dKNDZ1VmVPeGhNZ2VLYnF2bVNxOVBYTlpscmQ3c1loMFZCV0NTMWJ3eUtMTnBmRmNrTllIOFAKQ2w4WUdJQXhveGZCTkUxRmsyZzRQb21aZjd2QUFhM3ozVlFYekpER2QzV2pSaTViU25OcGZOc09PeUJnT2hkMQpNRmNoVE5Udm0rVzl1aFZMcW1WMnpSNTRTKzlUZ2ZWaVFXYXo1WmdUdWg0Y0hWNjhXaDhoCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```

### kubectl config

```yaml
dor1@is-m1:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://lb.kubesphere.local:6443
  name: cluster.local
contexts:
- context:
    cluster: cluster.local
    user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

`.kube`에서 확인하였을 때는 user의 key값이 다 나오지만 `kubectl`에서는 `--raw`옵션 없이는 따로 나타나지는 않는다.

## RBAC 기반으로 새로운 User에 설정해보기
---

k8s는 기본적으로 API Server을 거쳐 운영이 된다.  
인증 방법은 Custom 까지 포함하여 총 5가지로 다음과 같은 방법이 있다.

- Basic HTTP Auth
- Access token via HTTP Header
- Client cert
- Bearer token
- Custom made

`Basic HTTP Auth`는 HTTP 요청에 사용자 아이디와 비밀번호를 같이 보내서 인증하는 방식인데, 아이디와 비밀번호가 네트워크를 통해서 매번 전송되기 때문에 보안 상 좋지 않고 번거로우며 권장하지 않는 방법이다.  
`Access token via HTTP Header`는 일반적인 REST API 인증에 많이 사용되는 방식인데, 사용자 인증 후 사용자에 부여된 API Token을 HTTP Header에 같이 보내는 방식이다.  
`Client cert`는 클라이언트의 식별을 인증서(Certification)를 이용해서 인증하는 방식이다.  
`Bearer token`은 Authorizing mechanism 중에서 가장 간단한 방법으로 API Token을 HTTP Header에 넣는 방식이다.  
OAuth 2.0 같은 곳에서도 많이 쓰이는 방식인 Bearer token 방식이 현재에서도 가장 권장하는 방법이다.

### 1. Role 구성

Role은 특정 API나 Resource에 대한 권한을 명시해준 규칙이다.  
Role의 큰 범위로는 사용자(User)의 Role와 클러스터(Cluster) 단위의 Role이 존재한다.  
Role은 그 Role이 속한 네임스페이스(NameSpace)에만 적용 된다.

#### 1. Role 적용

```yaml
# role.yaml(사용자에게 kubectl get pods, services의 read-role만 지정)
dor1@is-m1:~$ cat role.yaml
kind: Role 
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods", "serivces"]
#   resourceNames: [“mypod"]
    verbs: ["get"]
```

```shell
# role.yaml 적용
dor1@is-m1:~$ kubectl apply -f ./role.yaml
role.rbac.authorization.k8s.io/read-role created
```

`resourceNames`을 지정해주면 해당 네임스페이스에서만 작동한다.  
`resourceNames`이 지정되면 `create`, `watch`, `list`, `deletecollection`등의 동사(Verb)는 작동하지 않는다.  
Role의 동사(Verb)는 다음과 같이 이뤄져있다.

| Verb             | Description                             |
| ---------------- | --------------------------------------- |
| create           | 신규 Resource를 생성한다.               |
| get              | 개별 Resource를 조회한다.               |
| list             | 여러개의 Resource를 조회한다.           |
| update           | 기존 Resource의 내용 전체 업데이트한다. |
| patch            | 기존 Resouce 중 일부 내용 변경한다.     |
| delete           | 개별 Resouce를 삭제한다.                |
| deletecollection | 여러 Resouce를 삭제한다.                |

#### 2.  ClusterRole 적용

ClusterRole은 Role과 구문이 흡사하다.

```yaml
# clusterrole.yaml(사용자에게 전체 NameSpace의 Cluster Pods를 조회할 수 있는 read-clusterrole 지정)
dor1@is-m1:~$ cat clusterrole.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

```shell
# clusterrole.yaml 적용
dor1@is-m1:~$ kubectl apply -f ./clusterrole.yaml
clusterrole.rbac.authorization.k8s.io/read-clusterrole created
```

ClusterRole에서는 `metadata`부분에 `namespace`가 따로 존재하지는 않으며, 전체 네임스페이스가 기본적으로 적용 된다.  
ClusterRole은 k8s에서 편의를 위해 다음과 같이 미리 정해져 있는 Role이 존재한다.

| Default ClusterRole | Default ClusterRoleBinding | Description                                                                                                                                                                                                                                                                                        |
| ------------------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| cluster-admin       | system:masters group       | k8s Cluster에 대해서 SuperUser(Admin) 권한을 부여한다. <br> ClusterRoleBinding을 이용해서 Role을 연결할 경우에는 모든 네임스페이스와 모든 리소스에 대한 권한을 부여한다. <br> RoleBinding을 이용하여 Role을 부여하는 경우에는 해당 네임 스페이스에 있는 리소스에 대한 모든 컨트롤 권한을 부여한다. |
| admin               | None                       | 관리자 권한을 부여한다. <br> RoleBinding을 이용한 경우에는 해당 네임스페이스에 대한 대부분의 리소스에 대한 권한을 부여한다. <br> 새로운 Role을 정의하고 RoleBinding을 정의하는 권한을 포함하지만, resource에 대한 quota에 대한 조정 기능은 가지지 않는다.                                          |
| edit                | None                       | 네임스페이스 내의 내용을 읽고 쓰는 기능에 대해 갖게 되지만, Role이나 RoleBinding을 쓰거나 수정은 불가능 하다.                                                                                                                                                                                      |
| view                | None                       | 해당 네임스페이스 내의 내용에 대한 읽기 기능을 갖는다. <br> Role이나 RoleBinding을 조회하는 권한은 가지고 있지 않다.                                                                                                                                                                               |

추가적으로, ClusterRole은 `aggregationRule`을 이용해서 다른 ClusterRole들을 조합하여 사용이 가능하다.

```yaml
# 예제
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin-aggregation
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      # 다른 Cluster의 Label을 가져올 수 있음, 예제에서는 기본 Label인 kubernetes.io/bootstrapping: rbac-defaults을 적용 함
      kubernetes.io/bootstrapping: rbac-defaults
rules: []
```

`aggregationRule`의 아래에 `clusterRoleSelectors`부분이 있는데 이 부분에 Label이 되어있는 다른 Cluster의 항목을 가져오게 된다.

#### 3. SA(Service Account) 생성 및 RoleBinding 적용

RoleBinding은 앞서 적용했던 Role을 대상으로 사용자(User)에게 binding 해주는 역할이다.  
사용자는 보통 Service Account 또는 User로 구성이 된다.  
User는 일반 사용자 계정이며, SA(Service Account)는 시스템에서 define된 service만 제공하는 계정이다.  
SA는 Google API 사용할 때도 많이 사용이 되는 계정이므로 익숙할 수도 있다.

```yaml
# sa.yaml(dor1라는 SA를 생성)
dor1@is-m1:~$ cat sa.yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: dor1
  namespace: default
```

```shell
# sa.yaml 적용
dor1@is-m1:~$ kubectl apply -f sa.yaml
serviceaccount/dor1 created
```

```yaml
# 적용 확인
dor1@is-m1:~# kubectl describe sa dor1
Name:                dor1
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   dor1-token-pq4k7
Tokens:              dor1-token-pq4k7
Events:              <none>
```

SA 계정이 생성 되었다면, RoleBinding도 적용해준다.

```yaml
# rolebinding.yaml(dor1라는 SA에 Role을 적용)
dor1@is-m1:~$ cat rolebinding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: dor1
  apiGroup: ""
roleRef:
  kind: Role
  name: read-role
  apiGroup: rbac.authorization.k8s.io
```

```shell
# rolebinding.yaml 적용
dor1@is-m1:~$ kubectl apply -f rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/read-rolebinding created
```

RoleBinding까지 적용 해 준다면 위에서 만든 Role이 `dor1`라는 SA에 적용이 된다.

#### 4. ClusterRoleBinding 적용

ClusterRoleBinding 또한 앞서 적용했던 ClusteerRole을 사용자(User)에게 binding 해주는 역할이다.  
동일하게 위에서 만들어주었던 SA에 적용해주면 된다.

```yaml
# clusterrolebinding.yaml(dor1라는 SA에 ClusterRole을 적용)
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: dor1
  namespace: default
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: read-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

```shell
# clusterrolebinding.yaml 적용
dor1@is-m1:~$ kubectl apply -f clusterrolebinding.yaml
clusterrolebinding.rbac.authorization.k8s.io/read-clusterrolebinding created
```

여기까지 적용이 다 잘되었다면 실제로 SA로 `contexts`(kubenetes 계정 정보)를 변경하여 사용이 가능하다.

### 2. kubectl config를 통한 계정 switching

kubectl config는 앞서 기본 개념에서 보았던 것과 같이 `.kube/config`의 내용 및 `kubectl config view`의 명령어를 통한 내용을 의미한다.  
보통 config에는 cluster의 정보와 contexts(Cluster의 부여된 계정)에 대한 정보 그리고 user의 대한 정보가 들어있다.

```shell
# context 리스트 확인
dor1@is-m1:~$ kubectl config get-contexts
CURRENT   NAME                             CLUSTER         AUTHINFO           NAMESPACE
*         kubernetes-admin@cluster.local   cluster.local   kubernetes-admin

# 현재 계정에서 사용하고 있는 oontext 확인
dor1@is-m1:~$ kubectl config current-context
kubernetes-admin@cluster.local

# SA의 Bearer token 확인
dor1@is-m1:~$ kubectl describe secrets dor1-token-pq4k7
Name:         dor1-token-pq4k7
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dor1
              kubernetes.io/service-account.uid: f2a17ae9-d0e6-4bf3-92ca-779acc50694f

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Ikw1VlpwSW05cy16TllTelBJSUYyMjFidzlBNm0zYnpWVVJvbHAySzNZSDgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRvcjEtdG9rZW4tcHE0azciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZG9yMSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImYyYTE3YWU5LWQwZTYtNGJmMy05MmNhLTc3OWFjYzUwNjk0ZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRvcjEifQ.fyfAtU6GdshDq741uVaBLmLtdfZFkK6saW8qoj1A3_NrELyTlfPVArZhKw3Hd8J5JV8_937aYRU5gDkNWSXPWHfPfLnrXOr_1qD6vjRAYf6rmIObJi4W8Ve4WshszUj5zpUduuxoVl3xJGKpEZN4ji58QRNVNI2_kE0YFq1WTxkIpQMeI0TwP8fO2NzU-rWUo74xKqwnBtTkq4NvTchHpEcNPY7j3IXCCG4So5Q2FadirF0HTxHPlT29LenMDgIEYRWeXRdXYc_WvEMzfLVx-U07A6s2Aa9yVsVyqErt7aGdyTYHDh345Qo89qvkeCwV226S7S7l5A4wrNtVTMyaGw

# Bearer token 적용
dor1@is-m1:~$ kubectl config set-credentials dor1 --token=eyJhbGciOiJSUzI1NiIsImtpZCI6Ikw1VlpwSW05cy16TllTelBJSUYyMjFidzlBNm0zYnpWVVJvbHAySzNZSDgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRvcjEtdG9rZW4tcHE0azciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZG9yMSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImYyYTE3YWU5LWQwZTYtNGJmMy05MmNhLTc3OWFjYzUwNjk0ZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRvcjEifQ.fyfAtU6GdshDq741uVaBLmLtdfZFkK6saW8qoj1A3_NrELyTlfPVArZhKw3Hd8J5JV8_937aYRU5gDkNWSXPWHfPfLnrXOr_1qD6vjRAYf6rmIObJi4W8Ve4WshszUj5zpUduuxoVl3xJGKpEZN4ji58QRNVNI2_kE0YFq1WTxkIpQMeI0TwP8fO2NzU-rWUo74xKqwnBtTkq4NvTchHpEcNPY7j3IXCCG4So5Q2FadirF0HTxHPlT29LenMDgIEYRWeXRdXYc_WvEMzfLVx-U07A6s2Aa9yVsVyqErt7aGdyTYHDh345Qo89qvkeCwV226S7S7l5A4wrNtVTMyaGw
User "dor1" set.

# 새로운 context 생성
dor1@is-m1:~$ kubectl config set-context dor1 --cluster cluster.local --user dor1 --namespace default
Context "dor1" created.

# 사용 할 context 적용
dor1@is-m1:~$ kubectl config use-context dor1
Switched to context "dor1".

# config 적용 확인
dor1@is-m1:~$ kubectl config view --raw
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1USXdOekF5TVRnMU1sb1hEVE15TVRJd05EQXlNVGcxTWxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTlBpCmJ1SjU3YisrTFNjTm1haVJYcnhwanBKcnBhTWQvc1pHbHpHV1RvallBVXBiaHhlRVhPdzRkbFBzMDhxb1R4T3MKTWt2Z002bXJVdEhQQkh0TmpIdjRMYmp6YytqUEoyclZmWVY2a0diZ0tEZkhpVUpxcGU1VHI5TVlJeThqSXV2RQpvVno4N2lVQzFVcExVdWdFc3lNQnB5Mi90VzR4eG93dXFLS1M3UlNsWE9jY0RvS1hXSzYzUHd1WWtjdUFZNlJkCndwd3JiM3NGQm1Cb255M2Q2aHI1Y1Jtd1FqV2NYa2hjZjJ6MXRDbW14Mk0wU1pMYVR0MmtlcWRXMHB1bWhIeXEKNDJVK1FiekJjU3pwSUEvRWNVcHpIQXo0MkZTV1BSWGkwMEY5cHRXU3NkTER3Y1N2M2t4SmQ3bEYzWjdmdGVEZQorRDlCQ3VXV1p2dXRtSzZPWFk4Q0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZFMW41cGRIVU1FZVNtL1pkditLRkloQXVtcVBNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCa1FrNDV3YVBrZDVvYjFENjNTYlg4WkJwOENEU0NDaXNheCtjKzl3L1dldTgxQUJzVgo2bW9yYVh1WENLT0IvZ0NKZVRDQWZPaGZLdkVXVU50TWhNYUs5OUV4R092YXM0QUtOZU9mb2NacjNONjRkeEdiCndxcXBOWWxBMFdVUnFzUGpraGxrTUFGUWZCUTcvZWxwQWx5NWxyUnprbVdRUS9wUHlRVk9QQWY4SHYvMmx2bVQKZUNpTWo4VGM5RnpTVDJ2MG0zT1RIUlhSY2FObm5ick5CYWFCZlpUaXdZQ0x5RmRDcERPYlQzbUlSNVltOE53Kwp2WXRqdmJCWWUrcCtZeEovU0N6SXBwYUFGcnN2K0pZMWI3SExsbG5BL3dkQjJCWTlHR0pGMzlDVEJjY0xEL3Z1CnF2ZG5DK1M4ZlJrNDBseGVUNE9NWklDcTdxVGVESkZPYTJSRQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://lb.kubesphere.local:6443
  name: cluster.local
contexts:
- context:
    cluster: cluster.local
    user: dor1
  name: dor1
- context:
    cluster: cluster.local
    user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
preferences: {}
users:
- name: dor1
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6Ikw1VlpwSW05cy16TllTelBJSUYyMjFidzlBNm0zYnpWVVJvbHAySzNZSDgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRvcjEtdG9rZW4tcHE0azciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZG9yMSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImYyYTE3YWU5LWQwZTYtNGJmMy05MmNhLTc3OWFjYzUwNjk0ZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRvcjEifQ.fyfAtU6GdshDq741uVaBLmLtdfZFkK6saW8qoj1A3_NrELyTlfPVArZhKw3Hd8J5JV8_937aYRU5gDkNWSXPWHfPfLnrXOr_1qD6vjRAYf6rmIObJi4W8Ve4WshszUj5zpUduuxoVl3xJGKpEZN4ji58QRNVNI2_kE0YFq1WTxkIpQMeI0TwP8fO2NzU-rWUo74xKqwnBtTkq4NvTchHpEcNPY7j3IXCCG4So5Q2FadirF0HTxHPlT29LenMDgIEYRWeXRdXYc_WvEMzfLVx-U07A6s2Aa9yVsVyqErt7aGdyTYHDh345Qo89qvkeCwV226S7S7l5A4wrNtVTMyaGw
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJWkpTSTNSZE14SVF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TWpFeU1EY3dNakU0TlRKYUZ3MHlNekV5TURjd01qRTROVE5hTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXRYRHJkNlJ1cVQwMW16a3kKWVNxbXFoWDNmSWVhUFpYd0hDbytYb2RsS1lRNVpSeHordEI4ODQ0T3VhcFZuZ1lYR1pvWGVYOU5YVG01RTJ3cApHbHVpQ0YrbDJEYjYxV3RHekJGMGRsVUkvK3hHcmkrTjNyQlZ0RWJzeHd0RVg1TzExYjlNekhmQ2xmT3VGWUloCkFYL0VvbER5WWxXRllVa2w5b0tGYkxmaWxDdjlxVE9EOTBoQmFEOGVtM2NoZFVvNnlSL0JPV29CTTBJdytHejgKalNjSmFrQTZ1Y0NMMDNISTlVSytXQk5FYmNBTi9OVjFwV01mOHVTNFY5K2JFd0NYYW5wYzhseDBJWnhiZWRicQo4VEFGTVI1NitRdFVaNWQ1ME9rYThvQnVQbkZIRklCSlJHZnpLZlNwSFp0NHpETWpmbU4wc3R0ejlER0RyU2RXCjFOUXVEd0lEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JSTlorYVhSMURCSGtwdjJYYi9paFNJUUxwcQpqekFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBVk9RZStEclppNmR6L0hYRUlvTnZEdzZURnJqTFREeFRtUlRSCk01U0M5ZG96alhQeXNIeThLT1ZJd2NoQ0Y2M0lwNC9qY1hSd01qNXJUK0lKMUJ6YXdNVjR5YzV2alFERjZFdisKNS9kUkFYUGpTTXdkdFZtSDV6NzRVZVdPOHMxL241Q2JCa0xnUFNXb2tiK3FQd3RkTDEzQ1djSm1UK1JHZ1BYdQpXZVVua3IvclUyYlo3Slp5MjBvZVppMExOcURtNmYxblB5WkhFUWhUSjROT25jTjFQcm9naVRnajRlWHo5Y05VCllrcWkvcFc4aVR3dzcwQi8vdGtONHk0d3dTT2dQZGRSWDN5QldmTm5JSzIyZjV2VnlPNUVScjdrVUY1M0tLQU0KaVZPd3pTMndCNWw5YmFoQkMvM3NZbHV3dEdSbjNWald2REZkVjFwY1ZZbTZCQWUxSnc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdFhEcmQ2UnVxVDAxbXpreVlTcW1xaFgzZkllYVBaWHdIQ28rWG9kbEtZUTVaUnh6Cit0Qjg4NDRPdWFwVm5nWVhHWm9YZVg5TlhUbTVFMndwR2x1aUNGK2wyRGI2MVd0R3pCRjBkbFVJLyt4R3JpK04KM3JCVnRFYnN4d3RFWDVPMTFiOU16SGZDbGZPdUZZSWhBWC9Fb2xEeVlsV0ZZVWtsOW9LRmJMZmlsQ3Y5cVRPRAo5MGhCYUQ4ZW0zY2hkVW82eVIvQk9Xb0JNMEl3K0d6OGpTY0pha0E2dWNDTDAzSEk5VUsrV0JORWJjQU4vTlYxCnBXTWY4dVM0VjkrYkV3Q1hhbnBjOGx4MElaeGJlZGJxOFRBRk1SNTYrUXRVWjVkNTBPa2E4b0J1UG5GSEZJQkoKUkdmektmU3BIWnQ0ekRNamZtTjBzdHR6OURHRHJTZFcxTlF1RHdJREFRQUJBb0lCQUROdkNURG5TZjlydkpCKwpERXdESFMvRi9sd3N6SXA4d0k0Ylk0YkVkdWJuOXFVMUJhT3FDbUc0ZVhBa1d4VHF3UTJlNHR5c083QWJ0dDFNCm9mSTQyNXZvRVVsVGZKT1hUNEIxeWovcEp4MzFTcXdDQ3dOL2xTdi9sd3R0cERvNzB5WCtqclMvbGtlUHhsK08KZmZEQTJXcng2MlA0dmxDdnZiVTlscmtVLzRQazRqQnpKQ284bUs4UEVwejBPZUdHTnYwWExYUHRYbUp0a3B6bApBQjNaWnVhNFo4YVdka1luTTJ4bGV6WnNEa0FFUEhrOXBWVU9oOHhIcXkzTmp2TkJraWdrZ3RIZ0pjc1QxUVkrCk1XN2l3VHhDMVY3L0hGS1ppZEZ4ZytydU42czNkRFk0a3Qzb1VobUhTRkg2SCtVRVNlR3dwT0J4SkZOZnpGd2oKRkJTMkVlRUNnWUVBNHJuN1VPL0JBcG1SdWJJdkhTaWxTdjVCb0pEaEk1TjhWeHd6RTdWUGYvayszU1hTSWsvaApGTmdjSkZabmN1bGFURnVLTTlvWHZUNzNXbzZPWFhlcXNSV2NvUUVvdEVZMTBQUEI4RXpEWEltbkU1TTJXWWN3CnpnZmVURHY2SnU1N2RDdU4vQ1RLUVZtTElhTjE3OGx1V2lTbFJTTFdLbHFWNzB2OHBLenZIS2tDZ1lFQXpONGQKUStlRmNQaG4zYnRNOEQxUjVKWFhTUXVaZ2VHai9sN0EwNnB1T01Yczl2WFJ2UW5KZ20zL2NlaW9RUVdXOFhzRApJRDZhOENCOXlSVXRyYlVmd09IUzNNbXJKRVJUSERyTTNFSENTTkRjSEg1VFlJd25WV1ovcXlDbmVyOENiU295ClRwUFljNGdoMlYzV1A1T2VhVTVhaGQwb3pSUkY5THAzbklaSXIvY0NnWUJKU0hINS9EUzNvV21meXY4OWZvakcKejUzb3gwdHVFMXJLVVR3Vkw3S05tOE44K1orTkphS0wrVHBIYUlJeGUwbUxpcjhGK1lWWXp3Um1pZE5zVktTZwpibXJkQTZIamV4b2ordFlCMU40RWlCMnZ6eEp2SjZwWHZlVlZZTUYvV2ZBZllZQ1lNbEFKaFdiYUxacU9NZDV3ClZvM3c1Y3l4amV3T2w5SUdiRHN4V1FLQmdRQ3hpeHlacUorQWxBYVBwcTY2MUttUUREdVMxamFtMU1HbXhMOGYKc09mczA3clZHNXcwMDdLTEVvRDZXc0xWOXQ0bFVKSVk4NmlheWMyNDRsMi8yT1EzNkgweFVxUzZ2V3U1WDB3Qwo1Z3BWeUl1NU5kRlVMcUkzNUtobnlkamJDNFl5elFya0JrVGpldXE2MGhQRzdVdXZ2M083NXpwZzRGendCbGw2CmtQV1ZhUUtCZ0dKNDZ1VmVPeGhNZ2VLYnF2bVNxOVBYTlpscmQ3c1loMFZCV0NTMWJ3eUtMTnBmRmNrTllIOFAKQ2w4WUdJQXhveGZCTkUxRmsyZzRQb21aZjd2QUFhM3ozVlFYekpER2QzV2pSaTViU25OcGZOc09PeUJnT2hkMQpNRmNoVE5Udm0rVzl1aFZMcW1WMnpSNTRTKzlUZ2ZWaVFXYXo1WmdUdWg0Y0hWNjhXaDhoCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==

# context 사용 확인
dor1@is-m1:~$ kubectl config get-contexts
CURRENT   NAME                             CLUSTER         AUTHINFO           NAMESPACE
*         dor1                             cluster.local   dor1
          kubernetes-admin@cluster.local   cluster.local   kubernetes-admin

# Test 1 (kubectl get pod는 제대로 작동 확인)
dor1@is-m1:~$ kubectl get pods -A
NAMESPACE      NAME                                                              READY   STATUS      RESTARTS   AGE
gpu-operator   gpu-feature-discovery-rl77h                                       1/1     Running     0          3d23h
gpu-operator   gpu-operator-1670469363-node-feature-discovery-master-6579vjvb9   1/1     Running     0          3d23h
gpu-operator   gpu-operator-1670469363-node-feature-discovery-worker-9qbql       1/1     Running     0          3d23h
gpu-operator   gpu-operator-1670469363-node-feature-discovery-worker-flkxm       1/1     Running     0          3d23h
gpu-operator   gpu-operator-7bdd8bf555-cn6tr                                     1/1     Running     0          3d23h
gpu-operator   nvidia-container-toolkit-daemonset-t4w6j                          1/1     Running     0          3d23h
gpu-operator   nvidia-cuda-validator-gpf6f                                       0/1     Completed   0          3d23h
gpu-operator   nvidia-dcgm-exporter-xz4bl                                        1/1     Running     0          3d23h
gpu-operator   nvidia-device-plugin-daemonset-79htm                              1/1     Running     0          3d23h
gpu-operator   nvidia-device-plugin-validator-qhpps                              0/1     Completed   0          3d23h
gpu-operator   nvidia-operator-validator-hr4mx                                   1/1     Running     0          3d23h
kube-system    calico-kube-controllers-69d878584c-k28cw                          1/1     Running     7          4d23h
kube-system    calico-node-6fgnm                                                 1/1     Running     4          4d23h
kube-system    calico-node-m2qq9                                                 1/1     Running     1          4d23h
kube-system    coredns-b5648d655-v6lr6                                           1/1     Running     4          4d23h
kube-system    coredns-b5648d655-zqtwl                                           1/1     Running     4          4d23h
kube-system    haproxy-is-w1                                                     1/1     Running     4          4d23h
kube-system    kube-apiserver-is-m1                                              1/1     Running     1          4d23h
kube-system    kube-controller-manager-is-m1                                     1/1     Running     1          4d23h
kube-system    kube-proxy-p2tlj                                                  1/1     Running     1          4d23h
kube-system    kube-proxy-th59q                                                  1/1     Running     4          4d23h
kube-system    kube-scheduler-is-m1                                              1/1     Running     1          4d23h
kube-system    nodelocaldns-79zcd                                                1/1     Running     1          4d23h
kube-system    nodelocaldns-b4sdd                                                1/1     Running     4          4d23h

# Test 2 (kubectl get services는 권한 없음 확인)
dor1@is-m1:~$ kubectl get services -A
Error from server (Forbidden): services is forbidden: User "system:serviceaccount:default:dor1" cannot list resource "services" in API group "" at the cluster scope
```

Role을 확인 해보면 Role에는 services에 대해 포함이 되어 get이 가능하지만, 그보다 위인 ClusterRole에서는 services 항목이 비어있어 get이 불가능한 것을 확인 할 수 있다.