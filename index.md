# 勉強の動機

AWSやAzureなどクラウドサービスが話題になって久しいが、私はこれまでこれらを勉強したことがなかった。昨年、今いる現場からインフラ周りの世話をするDevOpsメンバーをボールドから出して欲しいとの要請を受け、メンバーの一人にそれをお願いした。その後、そのメンバーからDevOpsチームの状況を何度か確認したが、不勉強のため正確にその報告を理解することができなかった。特にメンバーの報告の中で鍵となっている言葉の一つであるTerraformはさっぱり分からなかった。このため、良い機会だとDevOpsなどを含めこれを勉強することとした。

# Terraformとは

Terraformとは、DevOpsの目的を実現するためのInfrastructure as
Codeツールの一つである。まず、DevOpsとは何かから説明する。

## DevOpsとは

DevOpsとはDevelopment（開発）とOperations（運用）を組み合わせた造語であり、開発担当者と運用担当者が連携して協力する開発手法をいう。この造語が生まれた背景には、ハードウェアのソフトウェア化がある。ハードウェアのソフトウェア化とは、従来は物理的な機械だったコンピュータを仮想的な機械（ソフトウェア）で実現してしまうことだ。これにより１台の物理的なコンピュータの中に何台もの仮想的なコンピュータを載せることが可能となった。ハードウェアがソフトウェア化するとソフトウェア開発で培われた技術（バージョン管理など）を運用でも利用することが可能となった。この結果、開発と運用のの両者がより協力しあってソフトウェアのデリバリを今までより遥かに効率的にするというDevOpsの発想が生まれた。

## Infrastructure as Codeとは

ソフトウェアのデリバリを今までより遥かに効率的にするためには、従来は手動で行っていたソフトウェアのデリバリのプロセスを可能な限り自動化することが必要となる。このためのツールがIaC（Infrastructure
as Code）と呼ばれるツールだ。IaCツールには大きく分けて次の4つのカテゴリがある。

- 設定管理ツール
- サーバテンプレーティングツール
- オーケストレーションツール
- プロビショニングツール

それぞれについて説明する。

### 設定管理ツール

Chef、Puppet、Ansible
といったツールはどれも、既存のサーバ上にソフトウェアをインストー
ルし、管理するための設定管理ツールだ。これらのツールは設定ファイルの内容にもとづきサーバにソフトウェアをインストールする。サーバをまとめてアップ
デートできるローリングデプロイも可能となる。

### サーバテンプレーティングツール

設定管理ツールとは違った方法でサーバ上にソフトウェアをインストールするツールに、Docker、Packer、Vagrant
といったサーバテンプレーティングツールがある。サーバをまとめて起動し、それぞれのサー
バ上で同じコマンドを実行して設定する代わりに、サーバテンプレーティングツールは、OS、ソ
フトウェア、ファイル、その他すべてを含んだ完全な「スナップショット」を使ったイメージを作るものだ。サーバテンプレーティングツールには、ハードウェアを含むコンピュータシステム全体のイメージ（仮想マシン）を利用するPacker
や Vagrant といったツールと、より限定されたOS
のユーザスペースのイメージ（コンテナ）を利用するDockerなどのツールがある。

### オーケストレーションツール

サーバテンプレーティングツールにより作られた仮想マシンやコンテナを管理するために作られたのがオーケストレーションツールと呼ばれるツールだ。例えば
Kubernetes を使うと、Docker コンテナをどのように管
理するかをコードとして定義できるようになる。このため、主要なクラウドプロバイダのほとんどはKubernatesのネイティブサポートを提供している。

### プロビショニングツール

設定管理、サーバテンプレーティング、オーケストレーションツールは、各サーバで動くコード
を定義するのに対して、Terraform、CloudFormation、OpenStack Heat
といったプロビジョニ
ングツールは、サーバ自身を作成する役割を持つ。プロビジョニングツールを使うと、データベース、キャッシュ、ロードバランサ、キュー、監視、サブネットの設定、ファイアウォール設定、ルーティングのルール、SSL
証明書、その他インフラに関するほとんどあらゆるものを作成できる。

## Terraformはどう動くのか

Terraform は、Hashicorp によって作られ、Go
言語で書かれたプロビショニングツールだ。Terraformは手元のPCに作成したTerraform設定ファイルの内容を元に、AWS、Azure、Google
Cloudといったプロバイダに 対して
インフラをデプロイする。このとき利用するTerraform設定ファイルはサーバの設定をTerraform独自の形式で記述したファイルだ。

# 実際のTerraform

実際にTerraformを利用して、クラウドのAWSに次のような環境をデプロイしたので、その報告をする。

- 2台のWebサーバ
- Webサーバの負荷を均等化するためのロードバランサ1台
- WebサーバからアクセスされるDBサーバ1台

これらの環境をTerraformを用いてAWS上に作るためには次の作業が必要となる。

- AWSアカウントのセットアップ
- Terraformのローカルマシンへのインストール
- Terrafrom設定ファイルの記述
- Terraformの実行

それぞれを簡単に解説する。

## AWSアカウントのセットアップ

AWSアカウントは、https://aws.amazon.com/jp/
を開き、メニューに沿ってメールアドレスや名前などの情報を入力することで取得できる。ただ、以前クラウドベースの統合開発環境であるCloud9に興味を持ったとき、その使い勝手が知るためにAWSのアカウントを作っていた。このため、今回はこのアカウントをそのまま利用することができた。AWSは一切お金がかからない無料枠のアカウントで利用することもできるが、有料枠のアカウントでも一定の制限を超えない場合は無料で利用できる。私が作成していたのは有料枠のアカウントだ。

## Terraformのローカルマシンへのインストール

私が普段利用しているPCはMacであり、ソフトウェアのインストールにはHomebrewというパッケージマネージャが利用できる。Terraformのインストールは下記のコマンドを実行するだけだ。

- brew tap hashicorp/tap
- brew install hashicorp/tap/terraform

## Terrafrom設定ファイルの記述

Terraform設定ファイルは、作成するサーバの内容をTerraform独自の形式で記述した拡張子
.tf
のファイルだ。このファイルの文字コードはUTF-8の限定されており、改行コードはLFが推奨されている（CRLFも問題なく使用できる）。
私が作成したTerraform設定ファイルは
githubにアップロードしたため、下記のURLで内容を確認することができる。
https://github.com/umidori/Terraform

## Terraformの実行

私の記述したTerraform設定ファイルからAWSクラウドにサーバを作成するには、main.tfファイルが存在する下記の3つのディレクトリのそれぞれで、terraform
apply コマンドを実行するだけだ。

- live/global/s3
- live/stage/data-stores/mysql
- live/stage/services/webserver-cluster

### live/global/s3 での実行結果

live/global/s3 ディレクトリで terraform apply を実行した結果は下記の通り

```bash
tori@tortoise:s3$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_dynamodb_table.terraform_locks will be created
  + resource "aws_dynamodb_table" "terraform_locks" {
      + arn              = (known after apply)
      + billing_mode     = "PAY_PER_REQUEST"
      + hash_key         = "LockID"
      + id               = (known after apply)
      + name             = "terraform-up-and-running-locks"
      + read_capacity    = (known after apply)
      + stream_arn       = (known after apply)
      + stream_label     = (known after apply)
      + stream_view_type = (known after apply)
      + tags_all         = (known after apply)
      + write_capacity   = (known after apply)

      + attribute {
          + name = "LockID"
          + type = "S"
        }
    }

  # aws_s3_bucket.terraform_state will be created
  + resource "aws_s3_bucket" "terraform_state" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "terraform-up-and-running-state-tori5"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)
    }

  # aws_s3_bucket_public_access_block.public_access will be created
  + resource "aws_s3_bucket_public_access_block" "public_access" {
      + block_public_acls       = true
      + block_public_policy     = true
      + bucket                  = (known after apply)
      + id                      = (known after apply)
      + ignore_public_acls      = true
      + restrict_public_buckets = true
    }

  # aws_s3_bucket_server_side_encryption_configuration.default will be created
  + resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + rule {
          + apply_server_side_encryption_by_default {
              + sse_algorithm = "AES256"
            }
        }
    }

  # aws_s3_bucket_versioning.enabled will be created
  + resource "aws_s3_bucket_versioning" "enabled" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + versioning_configuration {
          + mfa_delete = (known after apply)
          + status     = "Enabled"
        }
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + s3_bucket_arn       = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_dynamodb_table.terraform_locks: Creating...
aws_s3_bucket.terraform_state: Creating...
aws_s3_bucket.terraform_state: Creation complete after 3s [id=terraform-up-and-running-state-tori5]
aws_s3_bucket_public_access_block.public_access: Creating...
aws_s3_bucket_versioning.enabled: Creating...
aws_s3_bucket_server_side_encryption_configuration.default: Creating...
aws_s3_bucket_public_access_block.public_access: Creation complete after 0s [id=terraform-up-and-running-state-tori5]
aws_s3_bucket_server_side_encryption_configuration.default: Creation complete after 1s [id=terraform-up-and-running-state-tori5]
aws_s3_bucket_versioning.enabled: Creation complete after 1s [id=terraform-up-and-running-state-tori5]
aws_dynamodb_table.terraform_locks: Creation complete after 7s [id=terraform-up-and-running-locks]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Outputs:

dynamodb_table_name = "terraform-up-and-running-locks"
s3_bucket_arn = "arn:aws:s3:::terraform-up-and-running-state-tori5"
aws_dynamodb_table.terraform_locks: Refreshing state... [id=terraform-up-and-running-locks]
aws_s3_bucket.terraform_state: Refreshing state... [id=terraform-up-and-running-state-tori5]
aws_s3_bucket_public_access_block.public_access: Refreshing state... [id=terraform-up-and-running-state-tori5]
aws_s3_bucket_versioning.enabled: Refreshing state... [id=terraform-up-and-running-state-tori5]
aws_s3_bucket_server_side_encryption_configuration.default: Refreshing state... [id=terraform-up-and-running-state-tori5]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

dynamodb_table_name = "terraform-up-and-running-locks"
s3_bucket_arn = "arn:aws:s3:::terraform-up-and-running-state-tori5"
```

### live/stage/data-stores/mysql での実行結果

live/stage/data-stores/mysql ディレクトリで terraform apply
を実行した結果は下記の通り

```bash
tori@tortoise:mysql$ terraform apply
var.db_password
  The password for the database

  Enter a value:

var.db_username
  The username for the database

  Enter a value:


Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_db_instance.example will be created
  + resource "aws_db_instance" "example" {
      + address                               = (known after apply)
      + allocated_storage                     = 10
      + apply_immediately                     = false
      + arn                                   = (known after apply)
      + auto_minor_version_upgrade            = true
      + availability_zone                     = (known after apply)
      + backup_retention_period               = (known after apply)
      + backup_target                         = (known after apply)
      + backup_window                         = (known after apply)
      + ca_cert_identifier                    = (known after apply)
      + character_set_name                    = (known after apply)
      + copy_tags_to_snapshot                 = false
      + db_name                               = "example_database"
      + db_subnet_group_name                  = (known after apply)
      + delete_automated_backups              = true
      + domain_fqdn                           = (known after apply)
      + endpoint                              = (known after apply)
      + engine                                = "mysql"
      + engine_version                        = (known after apply)
      + engine_version_actual                 = (known after apply)
      + hosted_zone_id                        = (known after apply)
      + id                                    = (known after apply)
      + identifier                            = (known after apply)
      + identifier_prefix                     = "terraform-up-and-running"
      + instance_class                        = "db.t3.micro"
      + iops                                  = (known after apply)
      + kms_key_id                            = (known after apply)
      + latest_restorable_time                = (known after apply)
      + license_model                         = (known after apply)
      + listener_endpoint                     = (known after apply)
      + maintenance_window                    = (known after apply)
      + master_user_secret                    = (known after apply)
      + master_user_secret_kms_key_id         = (known after apply)
      + monitoring_interval                   = 0
      + monitoring_role_arn                   = (known after apply)
      + multi_az                              = (known after apply)
      + nchar_character_set_name              = (known after apply)
      + network_type                          = (known after apply)
      + option_group_name                     = (known after apply)
      + parameter_group_name                  = (known after apply)
      + password                              = (sensitive value)
      + performance_insights_enabled          = false
      + performance_insights_kms_key_id       = (known after apply)
      + performance_insights_retention_period = (known after apply)
      + port                                  = (known after apply)
      + publicly_accessible                   = false
      + replica_mode                          = (known after apply)
      + replicas                              = (known after apply)
      + resource_id                           = (known after apply)
      + skip_final_snapshot                   = true
      + snapshot_identifier                   = (known after apply)
      + status                                = (known after apply)
      + storage_throughput                    = (known after apply)
      + storage_type                          = (known after apply)
      + tags_all                              = (known after apply)
      + timezone                              = (known after apply)
      + username                              = (sensitive value)
      + vpc_security_group_ids                = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + address = (known after apply)
  + port    = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_db_instance.example: Creating...
aws_db_instance.example: Still creating... [10s elapsed]
aws_db_instance.example: Still creating... [20s elapsed]
aws_db_instance.example: Still creating... [30s elapsed]
aws_db_instance.example: Still creating... [40s elapsed]
aws_db_instance.example: Still creating... [50s elapsed]
aws_db_instance.example: Still creating... [1m0s elapsed]
aws_db_instance.example: Still creating... [1m10s elapsed]
aws_db_instance.example: Still creating... [1m20s elapsed]
aws_db_instance.example: Still creating... [1m30s elapsed]
aws_db_instance.example: Still creating... [1m40s elapsed]
aws_db_instance.example: Still creating... [1m50s elapsed]
aws_db_instance.example: Still creating... [2m0s elapsed]
aws_db_instance.example: Still creating... [2m10s elapsed]
aws_db_instance.example: Still creating... [2m20s elapsed]
aws_db_instance.example: Still creating... [2m30s elapsed]
aws_db_instance.example: Still creating... [2m40s elapsed]
aws_db_instance.example: Still creating... [2m50s elapsed]
aws_db_instance.example: Still creating... [3m0s elapsed]
aws_db_instance.example: Still creating... [3m10s elapsed]
aws_db_instance.example: Still creating... [3m20s elapsed]
aws_db_instance.example: Still creating... [3m30s elapsed]
aws_db_instance.example: Still creating... [3m40s elapsed]
aws_db_instance.example: Still creating... [3m50s elapsed]
aws_db_instance.example: Still creating... [4m0s elapsed]
aws_db_instance.example: Still creating... [4m10s elapsed]
aws_db_instance.example: Still creating... [4m20s elapsed]
aws_db_instance.example: Creation complete after 4m25s [id=db-DQSRKZ57GPBMHHPAJYBIEVOWAQ]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

address = "terraform-up-and-running20240624065325574200000001.cjoqq4ym2bhp.ap-northeast-1.rds.amazonaws.com"
port = 3306
```

### live/stage/services/webserver-cluster での実行結果

live/stage/services/webserver-cluster ディレクトリで terraform apply
を実行した結果は下記の通り

```bash
tori@tortoise:webserver-cluster$ terraform apply
module.webserver_cluster.data.terraform_remote_state.db: Reading...
module.webserver_cluster.data.aws_vpc.default: Reading...
module.webserver_cluster.data.terraform_remote_state.db: Read complete after 1s
module.webserver_cluster.data.aws_vpc.default: Read complete after 0s [id=vpc-0d3e3cabbd64c3fcb]
module.webserver_cluster.data.aws_subnets.default: Reading...
module.webserver_cluster.data.aws_subnets.default: Read complete after 0s [id=ap-northeast-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.webserver_cluster.aws_autoscaling_group.example will be created
  + resource "aws_autoscaling_group" "example" {
      + arn                              = (known after apply)
      + availability_zones               = (known after apply)
      + default_cooldown                 = (known after apply)
      + desired_capacity                 = (known after apply)
      + force_delete                     = false
      + force_delete_warm_pool           = false
      + health_check_grace_period        = 300
      + health_check_type                = "ELB"
      + id                               = (known after apply)
      + ignore_failed_scaling_activities = false
      + launch_configuration             = (known after apply)
      + load_balancers                   = (known after apply)
      + max_size                         = 2
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-05dd3973902da1016",
          + "subnet-0832d759aed38a78b",
          + "subnet-0c0bd5e5c45270cd0",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "webservers-stage"
        }
    }

  # module.webserver_cluster.aws_launch_configuration.example will be created
  + resource "aws_launch_configuration" "example" {
      + arn                         = (known after apply)
      + associate_public_ip_address = (known after apply)
      + ebs_optimized               = (known after apply)
      + enable_monitoring           = true
      + id                          = (known after apply)
      + image_id                    = "ami-07c589821f2b353aa"
      + instance_type               = "t2.micro"
      + key_name                    = (known after apply)
      + name                        = (known after apply)
      + name_prefix                 = (known after apply)
      + security_groups             = (known after apply)
      + user_data                   = "b69994ee99d4d3e0c684017889969401270e9180"
    }

  # module.webserver_cluster.aws_lb.example will be created
  + resource "aws_lb" "example" {
      + arn                                                          = (known after apply)
      + arn_suffix                                                   = (known after apply)
      + desync_mitigation_mode                                       = "defensive"
      + dns_name                                                     = (known after apply)
      + drop_invalid_header_fields                                   = false
      + enable_deletion_protection                                   = false
      + enable_http2                                                 = true
      + enable_tls_version_and_cipher_suite_headers                  = false
      + enable_waf_fail_open                                         = false
      + enable_xff_client_port                                       = false
      + enforce_security_group_inbound_rules_on_private_link_traffic = (known after apply)
      + id                                                           = (known after apply)
      + idle_timeout                                                 = 60
      + internal                                                     = (known after apply)
      + ip_address_type                                              = (known after apply)
      + load_balancer_type                                           = "application"
      + name                                                         = "terraform-asg-example"
      + name_prefix                                                  = (known after apply)
      + preserve_host_header                                         = false
      + security_groups                                              = (known after apply)
      + subnets                                                      = [
          + "subnet-05dd3973902da1016",
          + "subnet-0832d759aed38a78b",
          + "subnet-0c0bd5e5c45270cd0",
        ]
      + tags_all                                                     = (known after apply)
      + vpc_id                                                       = (known after apply)
      + xff_header_processing_mode                                   = "append"
      + zone_id                                                      = (known after apply)
    }

  # module.webserver_cluster.aws_lb_listener.http will be created
  + resource "aws_lb_listener" "http" {
      + arn               = (known after apply)
      + id                = (known after apply)
      + load_balancer_arn = (known after apply)
      + port              = 80
      + protocol          = "HTTP"
      + ssl_policy        = (known after apply)
      + tags_all          = (known after apply)

      + default_action {
          + order = (known after apply)
          + type  = "fixed-response"

          + fixed_response {
              + content_type = "text/plain"
              + message_body = "404: page not found"
              + status_code  = "404"
            }
        }
    }

  # module.webserver_cluster.aws_lb_listener_rule.asg will be created
  + resource "aws_lb_listener_rule" "asg" {
      + arn          = (known after apply)
      + id           = (known after apply)
      + listener_arn = (known after apply)
      + priority     = 100
      + tags_all     = (known after apply)

      + action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }

      + condition {
          + path_pattern {
              + values = [
                  + "*",
                ]
            }
        }
    }

  # module.webserver_cluster.aws_lb_target_group.asg will be created
  + resource "aws_lb_target_group" "asg" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = (known after apply)
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancing_algorithm_type      = (known after apply)
      + load_balancing_anomaly_mitigation  = (known after apply)
      + load_balancing_cross_zone_enabled  = (known after apply)
      + name                               = "terraform-asg-example"
      + name_prefix                        = (known after apply)
      + port                               = 8080
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTP"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = "vpc-0d3e3cabbd64c3fcb"

      + health_check {
          + enabled             = true
          + healthy_threshold   = 2
          + interval            = 15
          + matcher             = "200"
          + path                = "/"
          + port                = "traffic-port"
          + protocol            = "HTTP"
          + timeout             = 3
          + unhealthy_threshold = 2
        }
    }

  # module.webserver_cluster.aws_security_group.alb will be created
  + resource "aws_security_group" "alb" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "webservers-stage-alb"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

  # module.webserver_cluster.aws_security_group.instance will be created
  + resource "aws_security_group" "instance" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + description      = ""
              + from_port        = 8080
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 8080
            },
        ]
      + name                   = "terraform-example-instance"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

  # module.webserver_cluster.aws_security_group_rule.allow_http_inbound will be created
  + resource "aws_security_group_rule" "allow_http_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 80
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 80
      + type                     = "ingress"
    }

  # module.webserver_cluster.aws_security_group_rule.allow_http_outbound will be created
  + resource "aws_security_group_rule" "allow_http_outbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 0
      + id                       = (known after apply)
      + protocol                 = "-1"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 0
      + type                     = "egress"
    }

Plan: 10 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + alb_dns_name = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.webserver_cluster.aws_security_group.alb: Creating...
module.webserver_cluster.aws_security_group.instance: Creating...
module.webserver_cluster.aws_lb_target_group.asg: Creating...
module.webserver_cluster.aws_lb_target_group.asg: Creation complete after 1s [id=arn:aws:elasticloadbalancing:ap-northeast-1:353584740272:targetgroup/terraform-asg-example/438ed6fe22e850e3]
module.webserver_cluster.aws_security_group.alb: Creation complete after 2s [id=sg-0714bc169b60178bb]
module.webserver_cluster.aws_security_group_rule.allow_http_outbound: Creating...
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Creating...
module.webserver_cluster.aws_lb.example: Creating...
module.webserver_cluster.aws_security_group.instance: Creation complete after 2s [id=sg-003b0e1b42f1e77a5]
module.webserver_cluster.aws_security_group_rule.allow_http_outbound: Creation complete after 0s [id=sgrule-1405491395]
module.webserver_cluster.aws_launch_configuration.example: Creating...
module.webserver_cluster.aws_launch_configuration.example: Creation complete after 1s [id=terraform-20240624074117447500000001]
module.webserver_cluster.aws_autoscaling_group.example: Creating...
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Creation complete after 1s [id=sgrule-3822690036]
module.webserver_cluster.aws_lb.example: Still creating... [10s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [10s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [20s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [20s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [30s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [30s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Creation complete after 35s [id=terraform-20240624074117964500000002]
module.webserver_cluster.aws_lb.example: Still creating... [40s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [50s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m0s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m10s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m20s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m30s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m40s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m50s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m0s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m10s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m20s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m30s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m40s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m50s elapsed]
module.webserver_cluster.aws_lb.example: Creation complete after 2m52s [id=arn:aws:elasticloadbalancing:ap-northeast-1:353584740272:loadbalancer/app/terraform-asg-example/1b39d6b244ae8aab]
module.webserver_cluster.aws_lb_listener.http: Creating...
module.webserver_cluster.aws_lb_listener.http: Creation complete after 0s [id=arn:aws:elasticloadbalancing:ap-northeast-1:353584740272:listener/app/terraform-asg-example/1b39d6b244ae8aab/976022dd8206362d]
module.webserver_cluster.aws_lb_listener_rule.asg: Creating...
module.webserver_cluster.aws_lb_listener_rule.asg: Creation complete after 1s [id=arn:aws:elasticloadbalancing:ap-northeast-1:353584740272:listener-rule/app/terraform-asg-example/1b39d6b244ae8aab/976022dd8206362d/e065c3e60f52fcfb]

Apply complete! Resources: 10 added, 0 changed, 0 destroyed.

Outputs:

alb_dns_name = "terraform-asg-example-567919316.ap-northeast-1.elb.amazonaws.com"
tori@tortoise:webserver-cluster$
```

## 作成されたサーバの稼働確認

上記コマンドにより開始されたサーバの稼働を確認するには、作成されたロードバランサのDNS名をWebブラウザのURLに指定すると良い。指定するURL名は下記の通り。

- http://terraform-asg-example-567919316.ap-northeast-1.elb.amazonaws.com

上記URLをブラウザに指定すると下記の内容が表示される。

```
Hello, World
DB address: terraform-up-and-running20240624065325574200000001.cjoqq4ym2bhp.ap-northeast-1.rds.amazonaws.com
DB port: 3306
```

# Terraformを試してみた感想

Terraformは宣言型のIaCツールと言われている。宣言型のツールは最終的なサーバのイメージを記述すると、現在のサーバの状態と最終的な状態との差分をツールが検出し、その差分のみを実施する。今回、Terraformを試してみてその点は確かに便利だと感じた。今後はさらに研究を進め、宣言型のツールの弱点やその対策まで探ることができると良いと感じている。
