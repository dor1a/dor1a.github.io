---
title: Terraform - AWS Instance 배포 해보기
date: 2024-08-13 18:03:00 +0900
categories: [Cloud]
tags: [cloud, terraform, aws]
description: Terraform을 통해 간단히 AWS의 Instance를 배포 해보는 방법이다.
---

>Terraform v1.9.4
{: .prompt-info}

>Linux, Web
{: .prompt-warning}

>CLI
{: .prompt-tip}

## 개요
---

* Terraform은 Hashcorp에서 만든 자동화(Automation)를 위한 IaC(Infrastructure as Code)이다.
* 간단한 code로 t2.micro(free tier 제공)을 배포하는 방법을 진행한다.

> ref.
> - <https://ch4njun.tistory.com/171>

## Terraform 시작하기
---

우선 Windows든 Linux든 무언가의 client가 필요하다.
여기서는 Ubuntu에서 진행 하는 방법으로 진행한다.

### 1. terraform 설치하기

설치는 공식홈페이지에서 나와있는 그대로 진행하면 된다.

<https://developer.hashicorp.com/terraform/install?product_intent=terraform>

```shell
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

### 2. AWS에서 API Key 받기

필요한 key는 2가지다.
기본적으로 IAM에 등록된 계정 중 배포 권한이 들어가 있는 권한이여야지 error가 안난다.

우선 **User account → Security credentials** 로 이동한다.  
아래에 Access keys라는 항목에서 받을 수 있다.

![2개의 key는 다른 곳에서 사용하기 위해 미리 받아 놓은 것](/assets/img/post/cloud/2024-08-23-terraform-deploy_aws_instance/1.png)
_2개의 key는 다른 곳에서 사용하기 위해 미리 받아 놓은 것_

key는 최대 2개까지가 생성이 가능하다.  
생성시 다음과 같이 진행하면 된다.

**Create access key → Command Line Interface (CLI) 또는 Local code → Description tag value 작성(ex. terraform/hq-is-lxc-provisioning) → Access key 발급**

Retrieve access keys 페이지까지 도달하게 된다면 다음과 같이 Access key가 생성이 된 것이다.

![Access key와 Secret access key 두가지가 사용이 됨](/assets/img/post/cloud/2024-08-23-terraform-deploy_aws_instance/2.png)
_Access key와 Secret access key 두가지가 사용이 됨_

### 3. .tf 파일 만들어서 배포하기

Terraform은 .tf의 확장자를 가지고 있는 파일로 배포 및 운영이 된다.  
각 파일마다 provider와 key들에 대해서 설정할 수 있지만 디렉토리 구조로 운영되는 terraform 특성 상 하나의 파일에 key를 넣어서 운영하는 것이 편하다.

```shell
# 디렉토리 생성 및 이동
jhkim@hq-is-lxc-provisioning ~ > mkdir -p ./aws_manage/terraform && cd ./aws_manage/terraform
# aws-provider.tf 생성
jhkim@hq-is-lxc-provisioning ~/aws_manage/terraform > vi aws-provider.tf
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "ap-northeast-2"
}
```

Cloud 특성상 과금 되는 것을 고려하면 다른 것들은 많이 비싸니 AWS free tier에서 제공하는 t2.micro의 ec2 instance를 배포해본다.

```shell
jhkim@hq-is-lxc-provisioning ~/aws_manage/terraform > vi main.tf
resource "aws_instance" "example" {
  # Ubuntu 24.04 image
  ami = "ami-062cf18d655c0b1e8"
  instance_type = "t2.micro"
}
```

여기까지 만들어졌다면 terraform을 초기화 해본다.

```shell
jhkim@hq-is-lxc-provisioning ~/aws_manage/terraform ❯ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.62.0...
- Installed hashicorp/aws v5.62.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

디렉토리 구조는 다음과 같이 되어있다.

```shell
jhkim@hq-is-lxc-provisioning ~/aws_manage/terraform ❯ ll
total 24
drwxr-xr-x 3 jhkim SCV 4096 Aug 13 18:28 .
drwxr-xr-x 3 jhkim SCV 4096 Aug 13 16:27 ..
drwxr-xr-x 3 jhkim SCV 4096 Aug 13 18:28 .terraform
-rw-r--r-- 1 jhkim SCV 1377 Aug 13 18:28 .terraform.lock.hcl
-rw-r--r-- 1 jhkim SCV  143 Aug 13 18:25 aws-provider.tf
-rw-r--r-- 1 jhkim SCV   94 Aug 13 18:28 main.tf
```

실제 배포 전 dry run을 해본다.

```shell
jhkim@hq-is-lxc-provisioning ~/aws_manage/terraform ❯ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                                  = "ami-062cf18d655c0b1e8"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags_all                             = (known after apply)
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification (known after apply)

      + cpu_options (known after apply)

      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + instance_market_options (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

Dry run으로 체크 시 정상이면 apply를 한다.

```shell
jhkim@hq-is-lxc-provisioning ~/aws_manage/terraform ❯ terraform apply
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                                  = "ami-062cf18d655c0b1e8"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags_all                             = (known after apply)
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification (known after apply)

      + cpu_options (known after apply)

      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + instance_market_options (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 32s [id=i-0aac8c4a6a48b8dd6]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

정상적으로 Apply complete!가 떳다면, AWS EC2 inatance에서 확인이 가능하다.

![Instance Console](/assets/img/post/cloud/2024-08-23-terraform-deploy_aws_instance/3.png)
_Instance Console_