# VPC_ネットワーク構築手順

* VPC：Virtual Private Cloud
* 下記構成のVPCネットワークを作成する

### VPC

* リージョン：東京
* CIDR ブロック：10.0.0.0/16

### サブネット

#### パブリックサブネット

* Webサーバーを配置する
* CIDR ブロック：10.0.10.0/24

#### プライベートサブネット

* データベースサーバーを配置する
* CIDR ブロック：10.0.20.0/24

### アベイラビリティゾーン

* 上記のサブネットを同一のアベイラビリティゾーンに設定する
* アベイラビリティゾーンを分けるのはサーバーの冗長化を検討する段階になったらでよい

---

## １．VPCを作成する

* AWS上に仮想ネットワークを作成する
* ルーターやハブの準備に相当

### リージョンを確認する

* リージョンが **東京** になっていることを確認する
* または上部メニューからプルダウンで **アジアパシフィック(東京)** のリージョンを選択する

### VPCを作成する

1. VPCダッシュボードを開く
2. VPC作成画面を開く
3. 以下の設定をしてVPCを作成する
    * 作成するリソース：VPCのみ
    * 名前タグ：magenta-magenta-vpc
    * IPv4 CIDR ブロック：10.0.0.0/16
    * IPv6 CIDR ブロック：IPv6 CIDR ブロックなし
    * テナンシー：デフォルト
      * 物理ハードウェアを専有するオプション、EC2の料金が割高になる、ここでは不要

---

## ２．サブネットを作成する

* VPCを分割して利便性を高める
    * アベイラビリティゾーンを分けることで耐障害性を高める
    * ネットワークをパブリックとプライベートに分けることでセキュリティを高める
    * 業務目的に応じてネットワークごとのセキュリティレベルを設定する

### パブリックサブネット

1. サブネット画面を開く
2. サブネット作成画面を開く
3. 以下の設定をする
    * VPC ID：magenta-magenta-vpc
    * サブネット名：magenta-magenta-public-subnet-1a
    * アベイラビリティゾーン：アジアパシフィック(東京) / ap-northeast-1a
    * IPv4 CIDR ブロック：10.0.10.0/24
4. 続けて新しいサブネットをする

### プライベートサブネット

1. 以下の設定をしてサブネットを作成する
    * サブネット名：magenta-magenta-private-subnet-1a
    * アベイラビリティゾーン：アジアパシフィック(東京) / ap-northeast-1a
    * IPv4 CIDR ブロック：10.0.20.0/24

---

## ３．ルーティングを設定する

* インターネット回線を引く作業に相当
* ルートテーブルにインターネットゲートウェイ向けのターゲットを設定する
* サブネットからインターネットに接続できるようにする

### インターネットゲートウェイを作成する

1. インターネットゲートウェイ画面を開く
2. インターネットゲートウェイの作成画面を開く
3. 名前タグに入力（magenta-magenta-igw）する
4. インターネットゲートウェイを作成する

### VPCにアタッチする

1. 一覧から作成したインターネットゲートウェイを選択する
2. アクションメニューからVPCにアタッチを選択する
3. 作成済みのVPC（magenta-magenta-vpc）を選択する
4.  インターネットゲートウェイをアタッチする

### ルートテーブルを作成する

1. ルートテーブル画面を開く
2. ルートテーブル作成画面を開く
3. 以下の設定をしてルートテーブルを作成する
    * 名前タグ：magenta-magenta-public-route
    * VPC：magenta-magenta-vpc
4. 一覧に戻り作成したルートテーブルを選択する
5. サブネットの関連付けタブを開く
6. サブネットの関連付けの編集画面に進む
7. パブリックサブネット（magenta-magenta-public-subnet-1a）を選択する
8. 関連付けを保存する

### デフォルトルートをインターネットゲートウェイに設定する

1. 一覧から作成したルートテーブルを選択する
2. ルートタブを開く
3. ルート編集画面に進む
4. ルートを追加する
5. 以下の設定をして変更を保存する
    * 送信先：0.0.0.0/0
    * ターゲット：igw-xxxxx(magenta-magenta-igw)

* この時点で作成されているルートテーブル

  | 送信先     | ターゲット | 意味                                       |
  | ---------- | ---------- | ------------------------------------------ |
  | 1.0.0.0/16 | local      | 同VPC領域内の通信は内部でルーティング      |
  | 0.0.0.0/0  | igw-xxxxx  | それ以外はインターネットゲートウェイに転送 |

---

## VPC設計ポイント

### IPアドレス

* プライベートIPアドレスの使用を推奨
* 作成後の変更ができないためレンジを大きめに取る（/16推奨）
* 他システムとの重複に注意

### プライベートIPアドレス

| プライベートIPアドレス範囲    |
| ----------------------------- |
| 10.0.0.0 - 10.255.255.255     |
| 172.16.0.0 - 172.31.255.255   |
| 192.168.0.0 - 192.168.255.255 |

### アカウントの使い分け

#### システムが異なる場合

* システムごとにアカウントを分ける

#### 同一システムで環境が異なる（本番・ステージング・開発）場合

1. アカウントは同一でVPCとリージョンを分ける
   * ○：IAMの設定が一度で済む
   * ✕：別環境のリソースが見えるため事故につながる
2. アカウントを分ける
   * ✕：IAMの設定が環境ごとに必要になる
   * ○：別環境のリソースが見えないので事故を防げる

### サブネット

#### IPアドレス

* 将来的に枯渇しないIPアドレス範囲を見積もる（/24が一般的）

#### 分割ポリシー

* サブネットに割り当てられるルートテーブルは1つだけ
* ルーティングとアベイラビリティゾーンを基準にする

#### ルーティングポリシー

* インターネットアクセスの有無
* 拠点アクセスの有無

#### アベイラビリティゾーンポリシー

* 高可用性、耐障害性のためにアベイラビリティゾーンを分けて冗長化するためにサブネットを分割する