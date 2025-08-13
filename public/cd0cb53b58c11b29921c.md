---
title: 【AWS】VPC の外部にある ECS やら S3 のサービスに接続するなら NAT-GW と VPC Endpoints のどっちを選ぶべき？
tags:
  - AWS
  - vpc
  - アーキテクチャ
  - endpoints
  - nat-gateway
private: false
updated_at: '2024-11-25T19:59:42+09:00'
id: cd0cb53b58c11b29921c
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- VPC の外部にある AWS の各サービスへ接続を行う際に NAT-GW と VPC Endpoints のどちらを選択するべきかを考えた記事です
- あくまで自分の理解での話なので、状況によって適切な構成は変わってくることはご了承ください


# それぞれの候補サービスの概要説明

### NAT-GW

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html

- NAT = Network Address Translation (ネットワークアドレス変換)
- プライベートサブネット内のインスタンスが VPC 外のサービスと通信を行うことを可能にする
- 外部サービスがそのインスタンスとの接続を開始することはできない
- IPv4 通信専用（IPv6 通信を行うためには [Egress Only Internet-Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/egress-only-internet-gateway.html) を利用する）
- 接続タイプ
    - Public:
        - パブリック NAT ゲートウェイを介してインターネットに接続可能
        - インターネットから未承諾のインバウンド接続を受信することはできない
        - インバウンド接続はパブリックサブネット内にパブリック NAT ゲートウェイを作成し、作成時に Elastic IP アドレスを NAT ゲートウェイに関連付ける必要がある
        - NAT ゲートウェイへのトラフィックは、VPC のインターネットゲートウェイにルーティングする
    - Private:
        - プライベート NAT ゲートウェイを介して他の VPC またはオンプレミスのネットワークに接続できる
        - NAT ゲートウェイからのトラフィックを Transit Gateway または仮想プライベートゲートウェイ経由でルーティングできる
        - Elastic IP アドレスをプライベート NAT ゲートウェイに関連付けることは不可能
        - プライベート NAT ゲートウェイを使用して VPC にインターネットゲートウェイをアタッチできるが、プライベート NAT ゲートウェイからインターネットゲートウェイにトラフィックをルーティングすると、インターネットゲートウェイによってトラフィックがドロップする


### VPC Endpoints

https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html#concepts-vpc-endpoints

- VPC 内のリソースがインターネットを経由せずに他の AWS サービスにアクセスできるようにするためのサービス
- VPCエンドポイントを経由して VPC 内のインスタンスと VPC 外のサービスをプライベート接続で通信可能となる
- エンドポイントの種類
    - Interface: 
        - AWS PrivateLink を使用して、 VPC 内のリソースがAWSサービスにプライベート IP アドレスを使用してアクセスできるようにする
        - エンドポイントは VPC 内のサブネットに配置され、Elastic Network Interface（ENI）として機能する
    - Gateway: 
        - S3 や DynamoDB などの特定の AWS サービスに対して、 VPC 内のリソースがプライベート IP アドレスを使用してアクセスできるようにする
        - ルートテーブルにエントリを追加することで、VPC内のリソースがこれらのサービスにアクセスできるようになる

Interface | Gateway
--- | ---
・AWS 各種サービスにアクセス可能だが、ENI を利用するので追加コストが発生する。<br>・インターネットを介さずに各サービスにアクセス可能でセキュア |・ルートテーブルの設定だけなのでシンプル<br>・高スループット<br>・Interface タイプに比較してコストが安い<br>・特定のサービス（S3 や DynamoDB が代表例として挙げられる）に対してのみ利用が可能
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/be8cd334-9876-0e7f-94e0-4869a3e90bd0.png) | ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d5033928-43ee-669c-d79f-1a3c8da6d0e7.png)


# 今回検討するケース

- Private subnet にある EC2 インスタンスから ECS, ECR に接続して docker image を取得したり docker のデプロイをしたりしたい
- そのためには VPC の外にある ECS, ECR に接続する必要がある

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/b7b24665-bc89-3dc3-96f1-cfea41bb6798.png)

### まずは結論

- 費用対効果、セキュリティ制御対応を考えると NAT-GW が良い
- 比較ポイントは以下の通り

### NAT-GW の比較ポイント

- コスト
  - 起動時間ベース: 0.062 USD/hour
  - 処理データ量ベース: 0.062 USD/GB
- アウトバウンド通信制御を行うにあたってセキュリティ制御を行えている

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/80567869-c94b-9600-5d88-5dc19e5dee2a.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d2fc08d6-b96a-a225-6316-ab9e79e78c92.png)

https://aws.amazon.com/vpc/pricing/?nc1=h_ls

### VPC Endpoints の比較ポイント

- コスト
  - 起動時間ベース: 0.014 USD/hour
  - 処理データ量ベース: 0.001 USD/GB
- 各サービスごとに Security Group の設定を行う必要があることで起動時間などが6倍になることに加えて、マルチAZ分なのでその2倍の料金がかかる
    - `com.amazonaws.region.ecs-agent`
    - `com.amazonaws.region.ecs-telementry`
    - `com.amazonaws.region.ecs`
    - `com.amazonaws.region.ecr.api`
    - `com.amazonaws.region.ecr.dkr`
    - `com.amazonaws.region.logs`
- インターネットに出ることなく通信が行える

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/90b97c67-86ed-cfde-22d5-d724a9ac59a8.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/93befdec-72f6-7fc4-427f-6ba7733231f3.png)

https://aws.amazon.com/privatelink/pricing/


# あとがき

- これといって特にあるわけではないのですが、VPC Endpoints と NAT-GW がなぜそれぞれ存在するのかを知る良いきっかけになってよかった
- ちなみに色々触って調べている間に気づいたら VPC の課金が走っててあたふたしていました、アラート設定しておいてよかった…

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/c243853c-e775-33fc-f4a7-6a4a49f22c14.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/d0e3b7f4-8323-c3a6-3d15-0178963707f0.png)

