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
