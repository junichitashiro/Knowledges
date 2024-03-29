# AWSアカウント作成後の初期設定手順

---

## １．料金を使いすぎないための設定をする

### 通貨を日本円に設定する

1. アカウント名からアカウント設定画面を開く
2. お支払い通貨の設定を編集する
3. お支払い通貨の選択から **JPY - Japanses Yen** を選択する
4. 支払い通貨の更新を実行する

### 請求アラートの受信設定をする

1. アカウント名から請求ダッシュボードを開く
2. 請求設定画面を開く
3. 基本的にどの情報も受け取れるようにチェックを入れて設定を保存する
    * E メールで PDF 版請求書を受け取る
    * 無料利用枠の使用のアラートの受信
      * E メールアドレスに受信アドレスを入力する
    * 請求アラートを受け取る

### 料金アラートを設定する

* 使用料金が設定料金を超えた場合にアラートを出す

#### Step1 メトリクスと条件の指定

1. CloudWatch 画面を開く
2. リージョンを **バージニア北部** に切り替える
3. すべてのアラーム画面を開く
4. アラーム作成画面に進む
5. メトリクスの選択画面を開く
6. 参照タブを開く
7. 請求 - 概算合計請求額を選択する
    * USD・EstimatedChargesにチェック
    * メトリクスの選択画面に進む
8. 条件の設定で以下の選択をして次へ進む
    * 静的
    * より大きい
    * 10USD

#### Step2 アクションの設定

* アクションの設定で以下の選択をして次へ進む
  * アラーム状態トリガー：アラーム状態
  * SNS トピックの選択：既存の SNS トピック
    * 通知の送信先：Default_CloudWatch_Alarms_Topic
  * 既存がない場合は **新しいトピックの作成** から作成する

* SNSトピックの補足
  * SNSトピックというのは **Amazon Simple Notification Service** という通知用サービスにおけるトピックを指す
  * CloudWatchでアラートを設定する際、実際のメール通知はAmazon SNSが行う

#### Step3 名前と説明を追加

* アラーム名と必要があれば説明を入力して次へ進む

#### Step4 プレビューと作成

* これまでの設定を確認してアラームの作成を実行する

---

## ２．作業用ユーザーを作成する

### IAMユーザーの請求情報アクセスを許可しておく

* rootユーザー以外でも請求情報を見えるようにする

1. アカウント名からアカウント設定画面を開く
2. IAM ユーザー/ロールによる請求情報へのアクセスを編集する
3. IAM アクセスのアクティブ化にチェックを入れて更新する

### ユーザーを追加する

* [IAM_ユーザー追加手順](https://github.com/junichitashiro/Knowledges/blob/master/Cloud/AWS/IAM_ユーザー追加手順.md) に従ってユーザーグループとユーザーを追加する
* 一通りの環境構築が可能な権限が必要になるので **AdministratorAccess** のポリシーをアタッチする

---

## ３．操作ログの記録期間を変更する

* CloudTrailではデフォルトで証跡を保存しているが90日までとなっている
* 保存場所をS3に変更することで永久的に保存が可能となる
* S3は有料であり、個人利用なら必須ではない