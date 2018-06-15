---
title: "「マルチクラウド対応「Terraform」で、ITインフラを簡単にプロビジョニングしよう」に参加してきた #hashicorp #terraformjp"
date: 2018-06-15 21:00:00 +0900
---

[マルチクラウド対応「Terraform」で、ITインフラを簡単にプロビジョニングしよう][terraform-meetup] に参加してきました。

## タイムテーブル

| 時間          | タイトル                                        | 発表者                                                           |
|---------------|-------------------------------------------------|------------------------------------------------------------------|
| 18:30 - 19:00 | 受付、ネットワーキング                          |                                                                  |
| 19:00 - 19:10 | ご挨拶、HashiCorp についてご紹介                | Brian Burns, HashiCorp Japan株式会社 ゼネラルマネージャ          |
| 19:10 - 19:40 | Terraform Enterprise ユーザー事例               | 河野 隆志様, GMOメディア株式会社 サービス開発部 シニアエンジニア |
| 19:40 - 20:00 | Terraform Enterprise 紹介・デモ                 | 森 英悟, HashiCorp Japan株式会社 ソリューションエンジニア        |
| 20:00 - 20:30 | Lightning Talk セッション                       |                                                                  |
|               | Terraform と Azure を組み合わせて使うときの勘所 | 森山 京平様, 日本マイクロソフト株式会社                          |
|               | Terraform だって YAML で書きたい                | 羽深 修 様, 伊藤忠テクノソリューションズ株式会社                 |
|               | InSpec を使った Terraform のテスト              | 古川 貴朗様,　楽天株式会社                                       |
| 20:30 - 21:30 | ネットワーキング                                |                                                                  |

## ご挨拶、HashiCorpについてご紹介

ちゃんとしたカンファレンスじゃないので同時通訳無しを覚悟していたが、日本語がめっちゃ上手くてビビった。

- 元々、創業者がアドテク企業で働いており、そこで担っていたミッションが下記のようなものだった
  - アジャイルにする
  - クラウドに移行する
- これを実現するために自分たちで使うために作ったツールが terraform の元になっている
- マルチクラウドを共通のインターフェースで実現

- 日本法人
  - 今年から株式会社になった
  - 7名
    - GM: 日本、韓国を兼務

- 今日のタイムテーブルについて

## Terraform Enterprise ユーザー事例

- 採用時期: 昨年から

- GMO メディア株式会社
  - 20個のサービス、500台くらいのサーバー
  - オンプレ300、プライベートクラウド100、パブリッククラウド100 (主に AWS)

- GMO メディアにおけるインフラエンジニアの役割
  - サーバーの保守対応
  - 構築、運用
  - パフォーマンスチューニング
  - 社内システム

- Terraform とは
  - いわゆる Infrastructure as Code (IaC) を実現するためのツール

- Terraform の利用状況
  - 主にパブリッククラウドの100台が管理対象
  - サービスあたり多くても20台くらい

- リポジトリ構成
  - `terraform` の workspace 機能は使っていない
  - サービスA.git
    - production/
      - main.cf
      - variables.cf
      - outputs.tf
      - web-server.tf
      - ap-server.tf
    - development/
  - サービスB.git
    - 同上
  - サービスC.git
    - 以下省略

- Terraform になってできるようになったこと
  - インフラにもコードレビューの文化
    - GitHub の private repository に push して PR みたいな

  - チーム開発
    - RBAC によるロール管理
    - GitHub 連携 (GitHub 以外のバージョン管理ツールも OK)
    - tfstate ファイルの管理
      - 共有
      - 世代管理
    - Private Module Registry の利用

  - tfstate ファイルの保存先
    - Amazon S3 とかに保存ができるけど、複数名開発をするには State Lock するのに MongoDB とかが必要でめんどい

- Terraform Enterprise でできること
  - アカウント作成
    - Role 付与
    - Personal Access Token の発行
      - ローカルから `terraform plan` できるように
    - API の利用

  - チーム
    - アカウントを紐付け
    - workspace はアカウント、チーム単位で割り当て可能
    - チームへの Role や Team Access Token
      - 具体的なユースケースは思いつかない
      - 感想: 他ツールと連携するときとかかな？

  - Role
    - Read: `terraform plan` のみ
    - Write: `terraform apply` できる
    - Admin: Write + workspace の設定変更

  - 旨味
    - デプロイ(`terraform apply`)が被ることがない
      - ローカルからデプロイするのではなく、`master` ブランチへの変更をトリガーにして Terraform Enteprise がデプロイする
      - 必ずマージした順番にデプロイされる
        - 複数名開発でリポジトリを利用せずに Amazon S3 に tfstate を置くだけだとデプロイが同時に走る可能性がある
        - 単に CI/CD を使うだけだとマージ自体は  Git でアトミックな操作になるけど、デプロイ中に別のマージが行われると同様
          - 感想: CI/CD パイプライン側の設定で並列デプロイを防げばいいのでは…
      - 管理画面から State Lock が可能
        - State Lock 中はマージされてもデプロイできない
      - 誰がいつデプロイしたか分かる
      - 二要素認証

  - 対応する VCS
    - GitHub (含む Enterprise)
    - GitLab CE/EE
    - BitBucket
    - etc. メジャーなのは対応してる

  - 画面: GitHub の PR からリンクで Terraform Enterprise に飛べる

  - tfstate の管理
    - backend として atlas というのが使えるようになる
      - 普通に使う分には Amazon S3 とかと使用感は変わらない
    - Terraform Enterpise から tfstate の変更履歴が見れる
    - Personal Access Token を使ってアクセスする

  - Private Module Registry
    - サービス毎に workspace を分けたい
    - コードの共通化は Module でやりたい
    - でも Module を Public に公開したくないので Terraform Registry は使えない
    - タグによるバージョニングをしたい
    - セキュアに保ちたい

  - Private Module Registry の置き先
    - Local file Paths
    - Terraform Registry
      - タグによるバージョニングができる
    - GitHub
      - Private リポジトリは参照できるけど、credntial 情報をどこかに持たないといけないのでセキュアじゃない
    - BitBucket, Generic Git, HTTP URLs, Amazon S3 etc.

  - Private Module Registry の旨味
    - 使用感は Terraform Registry と同じ
    - タグによるバージョニング
    - GitHub の Repository がセキュアに参照できる
      - Terraform Enterprise 側で credential を保持するので .tf の中などに Git の credential 情報を持つ必要がない

  - よかったこと
    - コードレビュー文化
    - インフラエンジニア以外でもインフラ構築
      - 開発者でも教育すればちょっとした変更などができる
    - インフラ環境の複製など、スピードアップを実感

  - ハマったこと
    - 大量のリソースを管理すると遅い
      - 1 workspace に 2000 リソース → `terraform plan` だけで 20-30 分掛かる
      - workspace を分割して対応した
    - terraform 自体や provider のバージョンアップが早いので、追いつくのが大変
      - 何もしてないのにバージョンアップしただけで tfstate の差分が発生したり…
        - 最近はさすがになくなったかな
    - `value of 'count' cannot be computed`
      - Module 分けるとたまになるっぽい
      - issue もあるけど、まだちゃんと治ってないと思われる

  - Q&A
    - Q. なんで terraform の workspace 使ってないの？
      - A. Terraform Enteprise が未対応くさい。試せてない。1 つの workspace に複数の tfstate を持たせられないかも。 (あとで森さんから補足あり)
    - Q. モジュール、何個ぐらい使ってんの？
      - A. 30-40個くらい。
    - Q. モジュールにしてよかったもの、悪かったものがあったら教えて？
      - A. EC2 とか SG とかはよかった。ELB とかは、ターゲットグループに入れたり外したりしたいときがあるので微妙かも。
      - 注釈: Terraform は状態を宣言してその状態にするものなので、操作をモジュール化すると良くないんだろうな、と思った。
    - Q. 何人くらいで使ってる？
      - A. 10数人。インフラエンジニア、開発メンバーで半々くらい。
    - Q. Terraform Enteprise 自体の管理はどうしてる？
      - A. Terraform Enteprise 自体の管理は今のところ手作業でやってる。workspace 作ったりとかも手動だよ。

## Terraform Enterprise 紹介・デモ

- アマチュアボクサーの選手だそうで…すごい

- CI/CD の流れでデモンストレーション

- 前セッションの Q&A に関する補足
  - Terraform Enteprise の提供方法は GitHub Enteprise と同様に SaaS と Private Install がある
  - SaaS で利用してもらえれば管理は楽

- 管理単位について
  - organization
    - workspace
      - workspace がリポジトリ(つまり .tf のセット)と紐づく
  - 1 organization = 1プロジェクトにすると、複数の workspace を持てるので、production と staging を上手く管理できるかも
    - 注釈: organization - project - workspace という管理単位になるといいな、エンタープライズ開発を想定した作りになってないだけかも

- 機能紹介: Sentinel Policy (Policy as Code)
  - "本番適用は土日にしかできない"、"開発環境では t2.micro しか使わせない" みたいなポリシーもコード化できる
  - git で merge しても Terraform Enteprise が plan/deploy する前にポリシーに適合してるかチェックしてくれる

- 機能紹介: Variable Store
  - Terraform Enteprise 側で変数を持つことができる
  - credential 情報など、開発者にバラ撒きたくない値に使うと良さそう

  - OSS の Terraform を使ってくれる人めっちゃいるけど、Terraform Enteprise もよろしくな！
    - トライアル30日無料
    - 営業に言ってもらえると長くできると思うよ！笑

## LT: Terraform と Azure を組み合わせて使うときの勘所

- 2018/06/15追記: スライド上がってた
  - [TerraformとAzureを組み合わせて使うときの勘所][terraform-with-azure]

- Terraform で指定する値の正しい表記が知りたい
  - ロケーション: `az account list-locations`
  - インスタンスサイズ: `az vm list-sizes`
  - インスタンスをリサイズしたい: `az vm list-vm-resize-options`
  - イメージを指定したい:
    - `az vm image list-offers`
    - `az vm image list-publishers`
    - `az vm image list-skus`

- tfstate
  - もちろん Blob Storage に置けます
  - Blob Storage も Terraform で作れるよ

- `terraform deploy` するときに、いちいち `az login` するのめんどい
  - RBAC service principal 使ってね！
  - `az ad sp create-for-rbac` のあれ

- CI/CD で Secure Token を使いたくない、セキュアなやり方はないの？
  - Managed Service Identity (Preview)
    - 要は IAM Role のインスタンスプロファイル

- Terraform 環境作るのダルい
  - Azure Marketplae から全部入りの Terraform VM をデプロイできるよ

- Azure Kubernetes Services (AKS)が GA しました
  - [Terraform 使ったチュートリアル][terraform-aks-tutorial]があるので、ぜひ使ってね

## LT: Terraform だって YAML で書きたい

- 2018/06/15追記: スライド上がってた
  - [Terraform だって YAML で書きたい][terraform-with-yaml]
  - スライド中にタイマーあるのいいなと思った、真似したい。

- 宣伝
  - [Ignite][ignite] やってます
  - [インフラCI実践ガイド][infra-ci-guide] 6/18発売

- 好きなツール
  - Henchman
  - Goss
  - 要は Go と YAML が大好きだ
    - ツール自体の利便性は二の次!笑

- Terraform
  - HCL と JSON に対応している

- JSON
  - ハッシュの最後の要素に `,` (カンマ) が打てないのがダメ
  - ハッシュのキーを `""` で囲う必要があるのがダメ

- YAML
  - Terraform のページで dis られてるらしい

- 仕事で CloudFormation を YAML で書くか JSON で書くか検討した際も YAML 推しだった

- Terraform のソースコードに手を入れようと思ったが…
  - yaml2json になりました
    - LT のために手を入れるにはボリュームが大きくなりそうだった…
  - いつかちゃんとパーサーを書いて contribute するぞ!笑

## LT: InSpec を使った Terraform のテスト

- 2018/06/15追記: スライド上がってた
  - [InSpec を使った Terraform のテスト][terraform-with-inspec]

- Terraform は便利だが、一歩間違うと大惨事
  - Terraform 自体や Provider のバージョンアップ
  - モジュールやプラグイン開発で手を加えたとき
  - 変更をくわえるとき

- テストの重要性
  - 動作確認のためにマネージメントコンソールを触りたくない

- InSpec の紹介
  - serverspec と一緒じゃね？
    - serverspec にインスピレーションを受けたツール
    - DSL は serverspec に類似しているが、Ruby に詳しくない人でも使えるように、という思想
      - 例えば、`Rakefile` や `spec_helper.rb` を用意しなくても使える
  - InSpec で書かれたテストケースの例
  - テストの実効結果の例
  - 対応しているリソースは、まだ Azure より AWS の方が多い

- InSpec は便利だけど…
  - 一からテストケース(Profile)を書くのが辛い
  - Azure だと Azure Resource Exporter と睨めっこ

- inspec-iggy
  - tfstate から InSpec の Profile が生成される
  - やってみた
    - AWS だといけるけど、Azure は未対応
    - contribute するならチャンス!

## ネットワーキング

所用につき、未出席。ピザとビールがあったのに…ぐぬぬ。

## その他

- 会場について
  - GMO Yours
  - オフィス入り口は遅い時間(20:00だったかな？)になると閉まる
  - 遅い時間は B1F から出る
  - 改札みたいになってるんで、入館カードはゲートの口に入れて返却

- HashiCorp 主催 Webiner
  - 7/9(月) 18:00〜
  - 詳細は Web で!!

主催の HashiCorp Japan 様、会場提供をしていただいた GMO 様、ありがとうございました!

[terraform-meetup]: https://hashicorp.connpass.com/event/89569/
[terraform-aks-tutorial]: https://docs.microsoft.com/en-us/azure/terraform/terraform-create-k8s-cluster-with-tf-and-aks
[ignite]: https://www.ignite.ci/
[infra-ci-guide]: http://www.amazon.co.jp/o/ASIN/B07D2YCMJ5/xeres-22/
[terraform-with-azure]: https://www.slideshare.net/kyoheim/terraformazure-102448779
[terraform-with-yaml]: https://speakerdeck.com/habuka036/terraformdatuteyamldeshu-kitai
[terraform-with-inspec]: https://talks.godoc.org/github.com/tkak/talks/2018/06/testing-terraform-code-with-inspec.slide#1
