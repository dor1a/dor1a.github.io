---
title: AWS(Amazon Web Service) - 기본적인 구조와 서비스 알아보기
date: 2024-08-21 21:50:00 +0900
categories: [Cloud]
tags: [cloud, aws]
description: AWS에서 사용하는 기본적인 구조와 서비스를 알아보았다.
---

>AWS(Amazon Web Service)
{: .prompt-info}

>Web
{: .prompt-warning}

>GUI
{: .prompt-tip}


## 개요
---

>미완성 문서이며 항목 지속 추가 예정
{: .prompt-danger}

* 간단하게 기본적인 구조를 설명한다.
* 필요한 것들에 대해서 찾아보면서 기록한다.
* 가능하면 순차적으로 구성을 할 수 있도록 적는다.


## 각 서비스들에 대한 설명
---

### 1. VPC

아래는 기본적으로 필요한 구성들이다.  
아래와 같이 구성을 하면 차후 다른 서비스 구성이 간단해진다.

>VPC → Subnets → Route tables → IGW(Internet Gateways) / NAT gateways / Endpoints → Security groups
{: .prompt-info}

* VPC는 가장 큰 네트워크 망으로 생각하면 된다.
  즉, region내 최상위 network라고 생각하면 된다.
  보통 VPC를 생성하게 된다면, 기본적으로 위의 구성을 GUI로 선택하여 생성이 가능하다.
* Subnets은 On-prem으로 생각했을 때의 subnet과 같다.
  Region 내 AZ(Availability Zone, AZ마다 IDC 사이트로 예상)에 대해 설정하면 된다.
* Route tables는 On-prem의 route table과 같다.
  Routing은 “subnet associations”으로 쉽게 구성이 가능하다.
* IGW는 말 그대로 Gateway와 같다.
* NAT Gateway는 On-prem에서 L3 S/W에서 구성하는 NAT와 같다.
* Endpoints는 보통 S3와 같이 AWS에서 제공하는 정해진 서비스에 대해서 단독으로 사용이 가능하다.
  간단하게 Policy 작성과 Routing tables 선택이 가능하다.
* Security Group은 ON-prem에서 L3 S/W와 같이 In/Outbound rules(Port forwarding) 작성 또는 VPC associations와 같은 구성이 가능하다.

### 2. EC2

아래는 기본적으로 필요한 구성들이다.

>Instances → Elastic Block Store(EBS) → Elastic IPs(EIP) → Load balancers(LB) → Auto Scaling
{: .prompt-info}

* Instances는 말 그대로 On-prem에서 Host machine과 같다.
  기본적으로 Cloud의 Instance는 VM(Virtual Machine)으로 구성이 되며, Host machine(1 Node)단위로도 구성이 가능하다.
  Instance 생성을 하면 EBS도 같이 생성이 된다.
* Elastic Block Store(EBS)는 attach/detach가 가능한 block storage이다.
  EBS는 기본적으로 EC2 생성 시 같이 생성 된다.
  단, EC2를 삭제 시에 같이 삭제가 될 수 있다.
  즉, migration과 같은 작업이 필요할 경우에는 데이터 이전을 한 뒤 EBS를 삭제하는 방법을 택해야한다.
* Elastic IPs(EIP)는 Public IP(공인IP)를 사용할 때 보통 사용한다.
  VPC 망에 있는 경우가 아닌 Public IP가 필요할 경우 사용한다고 보면 된다.
  보통 EIP 설정 후 ssh로 접근을 하여 Instance에 접근한다.
* Load balancers(LB)는 다수의 Instance에서 서비스에 대해 LB 또는 active-backup과 같이 필요할 경우 생성하여 사용한다.
  구성 자체는 GUI로 간단히 가능하다.