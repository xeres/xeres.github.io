---
title: "行人坂.tech #0 に参加して AWS Fargate のハンズオンをやってみた #gyoninzaka_tech"
date: 2018-05-09 23:30:00 +0900
---

[行人坂.tech #0][gyoninzaka-tech] に行って、AWS Fargate を触ってきました。

## 行人坂.tech について

- スピーカー: [@hamburgerkid][hamburgerkid]さん

- 目的
  - AWS のマネージドサービスをハンズオンで体験してもらいたい
  - マネージドサービスを体験して、その利便性を業務で活かしてほしい
  - 勉強会自体が成熟してきたら、AWS のマネージドサービス以外も取り上げるかも

## AWS Fargate とは (座学)

- スピーカー: [@afukui][afukui]さん

### コンテナのいいところ

- 3つの利点
  - パッケージング
  - 配布
  - イミュータブル インフラストラクチャ

- 得られるもの
  - アプリケーションの高速なデプロイ
  - CI/CD を用いた、高速かつ高いポータビリティ性を生かしたテスト環境の構築

- 結果
  - アプリケーションの質とデプロイ速度が向上

### Amazon ECS

- growth
  - 年間アクティブ ユーザー 450+% (2016/2017年の比較)
  - 毎週、数億コンテナが起動
  - 数百万ものインスタンス上で
  - 2015年の GA 以降、50+ の新機能

- 他 AWS サービスとの連携
  - AWS VPC ネットワーク
  - Auto-Scaling
  - CloudWatch
  - ELB
  - ...etc.

### フィードバックで一番多かった要望

- アプリケーション開発だけをしたい
- インスタンスの管理なんかしたくない

- それができるのが AWS Fargate

### AWS Fargate の特徴

- インスタンス管理なし
- タスクネイティブAPI
- 使用リソースベース課金

- 結果
  - シンプルで、使いやすく、協力な新しいリソース利用モデル

### (コンテナの)ユースケース

- マイクロサービスアーキテクチャ
  - 多数のマイクロサービスを同じように管理

- 非同期ジョブ実行 (バッチ処理)
  - 柔軟にスケール
  - Lambda と違って実行時間に制約がない
    - (ワイ補足) Azure Functions だと App Service プランにすると実行時間については
      ある意味気にしなくていいので「ほへー」と思った

- 継続的インテグレーション、継続的デプロイ (CI/CD)
  - 開発、テスト、本番まで一貫したイメージを利用

- クラウドへのマイグレーション
  - レガシーシステムでもコンテナにさえしてしまえば容易にクラウドに持っていける
    - (ワイ補足) 各クラウド プロバイダーが提供しているマイグレーション サービスで
      非コンテナのまま持ってく方が楽なのではといつも思うのだが、偉い人どうすか

### コンテナ管理環境の変遷

1. ローカル実行
2. 各 EC2 インスタンス上で実行
3. Amazon ECS
4. AWS Fargate ← 今ココ

### Amazon ECS の構造

- Scheduling and Orchestration
  - Cluster Manager
  - Placement Engine
  - 結局、こいつらが裏で EC2 を動かしている

- Amazon ECS 用 EC2 インスタンスの AMI (ECS AMI)
  - ECS agent
  - Docker agent

- AWS Fargate だと…
  - EC2 インスタンスは見なくてよい
  - フロントの Scheduling and Orchestration はそのまま

### ECS を使用した Fargate コンテナの実行

- 同じ Task Definition スキーマ
- ECS API を使ってアクセス可能
- AWS Fargate と Amazon ECS の共存が可能
  - (ワイ補足) クラスターAは Amazon ECS、クラスターB は AWS Fargate というだけで、
    同一クラスタで共存できるわけではないと思われる

### AWS Fargate を利用した場合の Auto Scaling の違い

- Amazon ECS では EC2 Auto-Scaling と Task Auto-Scaling を設定していた
- AWS Fargate では Task Auto-Scaling だけを設定すればいい
- Task の実行で問題が出ないように EC2 Auto-Scaling 側で余裕を持たせる、
  などの工夫は不要なので無駄な課金がない

### ネットワーク

- AWS VPC ネットワークモードのみ
  - (ワイ補足) 要は bridge は使えないってことだと思われる
- Task 毎に ENI を自動割り当て
  - Security Group を Task 毎に設定可能
    - 同一クラスタ内であればツーツー
- Task 内のコンテナは localhost を共有
  - Link 不要で互いにアクセス可能
- VPC ないの他のリソースへ Private IP で通信が可能
  - AWS Fargate では Public IP の割り当ても可能

### ロードバランサー

- ALB / NLB どちらにも対応

### セキュリティ

- クラスタ単位での分離が可能
- IAM Role を Task 毎に設定可能

### 可視化とモニタリング

- CloudWatch Logs
- CloudWatch Events
- Service-level metrics

### ストレージ

- クラスターと同じライフサイクルのストレージサービス
  - Ephemeral は EBS サポート
  - コンテナストレージ領域 10GB
  - コンテナ間の共有ストレージ 4GB

- 永続化したい場合は別途で他のストレージサービスを利用すること

### レジストリサポート

- Amazon ECR
- Public Repositry
- 3rd-Party Private Repositry (comming soon)

### 料金体系

- CPU/メモリ×実行時間
- ストレージは不要
- データ転送量は標準
- CloudWatch を使う場合はその分は別途で

### SLA/コンプライアンス

- 99.99% (Amazon ECSと同様)
- HIPAA など、他サービスと同様のコンプライアンス認定

### AWS Fargate for Amazon EKS

- comming soon

(ワイ補足) [AWS Fargate][aws-fargate] のページを見ると下記のように記載あり
> AWS Fargate での Amazon EKS のサポートは 2018 年に開始される予定です。

### 使い分けも可能

- こんなことも可能
  - リリース当初は AWS Fargate で簡易にスタート
  - GPU が使いたい等、何らかの事情で一時的に Amazon ECS に
  - リファクタリングしてコンテナ利用に最適化したら AWS Fargate に戻す

- (ワイ補足) 多分、同じレジストリを参照して別クラスターにデプロイするだけ

## ハンズオン

- スピーカー: [@hamburgerkid][hamburgerkid]さん
- 資料が private Qiita だったので URL は割愛

1. マネジメントコンソールから AWS Fargate の sample-app + ALB をデプロイ
2. アプリケーションの更新
3. Blue/Green デプロイ
4. スケールアウト
5. スケールイン
6. クリーンアップ

[gyoninzaka-tech]: https://gyoninzaka-tech.connpass.com/event/85527/
[hamburgerkid]: https://twitter.com/hamburger_kid
[afukui]: https://twitter.com/afukui
[aws-fargate]: https://aws.amazon.com/jp/fargate/
