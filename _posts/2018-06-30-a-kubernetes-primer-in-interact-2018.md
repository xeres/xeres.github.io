---
title: "「今からはじめる Kubernetes 入門」を聞いてきた #MSInteract18 #T41"
date: 2018-06-30 13:54:00 +0900
---

## Kubernetes (以降、K8s) とは

### なんでコンテナ？

1. イメージ＝アプリケーション＋ミドルウェア
   - イメージの可搬性

2. 仮想マシンよりも小さい
   - 起動が早い
   - オーバーヘッドが小さい

3. コンテナリポジトリから pull することでイメージを共有できる

4. Dockerfile のシンプルさ
- ただのテキスト
- スタッキングでのカスタマイズの容易さ

### K8s とは

- 役割: コンテナオーケストレーション
  - デファクトスタンダード

- オーケストレーションとは？
  - 複数のコンテナをノードに配置して1つのプラットフォームとして扱うことができる
  - どこのノードに配置されるかは意識する必要がない
  - 自動でスケールイン、スケールアウト
  - たくさんのアプリやサービスをデプロイするのに適している
  - 障害を検知すると自動で新しいコンテナーが起動し、指定した台数を常に維持してくれる
    - "落ちない" ありきの今までの考え方と異なる
  - 更新は自動でローリングアップデート

### 構成要素

- Master Node
  - 各ノードの稼働状況やリソース消費を管理する

- Agent Node
  - 実際にワークロードが配置されるノード

- Pod
  - 複数のコンテナをまとめて管理する単位

### アーキテクチャ

- Pod は通常、外部からアクセスできない
- 外部 IP アドレスを持つ Service を作成し、コンテナでサービス提供ができるようになる

### 死活監視

- Deployment
  - 複数の Replica Set を管理する
  - バージョンアップなどで使う
- Replica Set
  - Pod の Replica を管理する
  - Pod の数を保証する (最低2台とか)
- Pod

### クラウドで提供される Kubernetes

- サービス提供内容
  - Master Node は隠蔽されており、無料で提供
  - 課金対象は Agent のみ
  - Agent Node のスケールアウト、スケールインが可能
  - Kubernetes Cluster のバージョンアップが可能

- 何がうれしい？
  - K8s の運用が大変なので、管理の手間が減る

- 代表的なサービス
  - GKE/AKS/EKS

## K8s を構築する

- アプリケーション開発に注力できるのでクラウドの利用推奨
- 今日は Azure K8s Service (AKS) で

### Azure K8s Service (以下、AKS)

- 東日本リージョンで GA
- アクセス方法
  - Azure Portal
  - Azure CLI 2.0

### `kubectl`

- クラスタの作成は Azure Portal/CLI からだが、その後は `kubectl` を使う
- .kube/config は Azure CLI で取得する
  - `az aks get-credentials --resource-group=<リソースグループ名> --name=<AKS名>`

- あとは大体どこのサービスも同じ

## K8s にアプリケーションをデプロイする

### コマンドを使ったデプロイ

- `kubectl run` コマンド

```
kubectl run <アプリ名> --image <コンテナリポジトリ名>
kubectl expose deployments <アプリ名> --port=80 --type=LoadBalancer
```

- `kubectl apply` コマンド

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Helm Charts

- K8s のパッケージマネージャー
  - 複数の YAML ファイルをパッケージ管理できる
  - 値を YAML にハードコーディングするのではなくパラメータとして与えることができる

- 構成
  - Client (helm)
  - Server

- KubeApps Hub
  - https://hub.kubeapps.com/
  - 既にパッケージングされたファイルが提供されている

### Demo

- ASP.NET Core アプリケーションをデプロイ

```
dotnet new mvc -o jazapp
cd jazapp
```

- Dockrfile を作成
  - Multi-staging 使ってる
  - より小さいサイズで docker image が作れる

- Docker イメージを作成し、リポジトリに登録

```
docker build -t user/repos:v14
docker push user/repos:14
```

- Helm パッケージを作成してインストールする

```
mkdir charts
helm create jazapp
code values.yaml
helm install jazapp -n <名前>
```

- 動作確認
```
kubectl get pod,svc
kubectl proxy
```

- インスタンスを増やしてみる
- あえて Pod を 1つ削除してみる
  - 代わりが数秒で立ち上がる

## K8s を運用監視する

- Prometheus
  - K8s と親和性の高い pull 型の監視ツール
  - メトリックがしきい値を越えたら、、Slack やメールでアラート可能
  - 一応、UI があるけどイケてない(笑)ので Grafana を利用するのがオススメ
- Grafana
  - Prometheus が取得した戸陸すをダッシュボードで可視化
  - プラグインを追加すると色々なデータソースを利用可能

## K8s にサービスメッシュを導入する

- Istio
  - Blue / Green Deployment
  - Canaria Release
  - Circuit Breaker
  - Bulkhead
  - 分散トレーシング

- K8s 単体で頑張ることもできるが、Istio を使う簡単にできる

### Istio アーキテクチャ

- Envoy 大事
  - サイドカーパターン
  - Istio Proxy コンテナーをインジェクションできる
  - アプリケーションに修正を加える必要ない

### 分散トレーシング

- Demo
- アプリ側に手を入れなくても分散トレーシングできる

## まとめ

- K8s は複数のコンテナを複数のノードに配置して1つのプラットフォームとして利用できる
  - アプリケーション1つだとあんまり向いてない
- 自分で運用してもいいけど手間が多いのでマネージドサービスがオススメ
- デファクトスタンダードになった K8s と、それを取り巻く OSS の進化の恩恵を受けることができる
