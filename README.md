# study-aws-ecs-laravel

ECSでLaravelコンテナを実行するサンプル

## AWSリソース

- ECR
- VPC

## AWS側作業

### ECR作成

1. リポジトリを作成する

- ecs-laravel/nginx
- ecs-laravel/laravel

1. イメージをpushする

### VPC作成

#### VPC

- タグ
  - `ecs-laravel`
- IPv4 CIDR ブロック
  - `10.0.0.0/16`
- AZ
  - 1つだけ

#### サブネット

- パブリックサブネットの数
  - 1
    - IPv4 CIDR ブロック
      - `10.0.0.0/24`
- プライベートサブネットの数
  - 1
    - IPv4 CIDR ブロック
      - `10.0.128.0/24`

#### インターネットゲートウェイ

自動的に作成され、VPCにアタッチされる

#### ルートテーブル

自動的に作成され、各サブネットにアタッチされる

### サブネット設定変更

パブリックサブネットのパブリックIPv4自動割り当てを有効化する

### セキュリティグループ作成

- セキュリティグループ名
  - `ecs-laravel`
- VPC
  - 作成したVPC
- インバウンドルール
  - ルール追加
    - タイプ: `HTTP`
    - ソース: `0.0.0.0/0`
- アウトバウンドルール
  - 追加なし

### IAMロール作成

ESCタスク用のIAMロールを作成する

- エンティティタイプ
  - AWSのサービス
- ユースケース
  - `Elastic Container Service Task`
- ポリシー
  - AmazonSSMReadOnlyAccess
  - AmazonECSTaskExecutionRolePolicy

### ECSサービスにリンクしたロールを作成

```bash
aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com

{
    "Role": {
        "Path": "/aws-service-role/ecs.amazonaws.com/",
        "RoleName": "AWSServiceRoleForECS",
        "RoleId": "AROA56KELXPCYUXETMG3Y",
        "Arn": "arn:aws:iam::958457953221:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS",
        "CreateDate": "2024-03-02T12:13:24+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "sts:AssumeRole"
                    ],
                    "Effect": "Allow",
                    "Principal": {
                        "Service": [
                            "ecs.amazonaws.com"
                        ]
                    }
                }
            ]
        }
    }
}
```

### CloudWatchの設定

#### CloudWatchにロググループを作成

- ロググループ名
  - `ecs-laravel`

### ECSクラスターの作成

- クラスター名
  - `ecs-laravel`
- インフラストラクチャ
  - AWS Fargete(サーバーレス)

### パラメータストアを作成

- 名前
  - `APP_WORD`
- タイプ
  - 安全な文字列
- 値
  - `Hoge Hoge!!`

### バッチ処理タスク

#### タスク定義を作成

- ecs-task-definition-for-command.json

タスク定義を登録

```bash
aws ecs register-task-definition --cli-input-json file://ecs-task-definition-for-command.json
```

### Webアプリケーションタスク

#### タスク定義を作成

- ecs-task-definition-for-web.json

タスク定義を登録

```bash
aws ecs register-task-definition --cli-input-json file://ecs-task-definition-for-web.json
```

## 実行

### バッチ処理タスク

ECSタスク実行

```bash
aws ecs run-task \
    --cluster ecs-laravel \
    --task-definition ecs-laravel-for-command \
    --overrides '{"containerOverrides": [ {"name": "laravel", "command": [ "sh", "-c", "php artisan print:helloworld" ]} ]}' \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-097b58401e07aa251], securityGroups=[sg-056fdf12c789a21db], assignPublicIp=ENABLED}"
```

### Webアプリケーション

サービス作成

```bash
aws ecs create-service \
    --cluster ecs-laravel \
    --service-name ecs-laravel \
    --task-definition ecs-laravel-for-web \
    --launch-type FARGATE \
    --desired-count 1 \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-097b58401e07aa251], securityGroups=[sg-056fdf12c789a21db], assignPublicIp=ENABLED}"
```

## サービスの削除

以下のコマンド実行後、GUIから削除する(コマンド無しで強制削除でも言いかも)

```bash
# サービスを確認
aws ecs list-services --cluster arn:aws:ecs:ap-northeast-1:958457953221:cluster/ecs-laravel

# desired-countを0にする
aws ecs update-service --cluster ecs-laravel --service ecs-laravel --desired-count 0
```
