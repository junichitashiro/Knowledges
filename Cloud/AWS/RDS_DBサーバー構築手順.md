# RDS_DBサーバー構築手順

---

## １．データベースを作成する

### データベースの設定

1. RDSダッシュボードを開く
2. データベース画面を開く
3. データベースの作成に進む

### データベース作成方法

* 標準作成

### エンジンのオプション

* エンジンのタイプ：MySQL
* エディション：MySQL Community
* バージョン：安定版を選択

### テンプレート

* 開発/テスト
  * 料金の制限がなければ本番稼働用でもよい
  * 無料利用枠は機能制限があるのでここでは採用しない

### 可用性と耐久性

* デプロイオプション：単一のDBインスタンス
  * スタンバイインスタンスを作成しない

### 設定

* DBインスタンス識別子：magenta-magenta-web
* マスターユーザー名：root
* マスターパスワード：xxxxx
  * 自動生成したほうが安全

### インスタンスの設定

* DBインスタンスクラス：バースト可能クラス

### ストレージ

* ストレージタイプ：汎用(SSD)
* ストレージ割り当て：20
* ストレージの自動スケーリング：無効
  * 高負荷にしない想定のため

### 接続

* EC2コンピューティングリソースへの接続：しない
* VPC：magenta-magenta-vpc
* サブネットグループ：magenta-magenta-subnet-group
* パブリックアクセス：なし
  * このDBは外部からアクセスさせない設計
* セキュリティグループ：既存から選択
  * magenta-magenta-db
  * デフォルトは外す
* アベイラビリティーゾーン：ap-northeast-1a
* 追加設定：ポート番号を指定したい場合設定する

### データベース認証

* データベース認証オプション：パスワード認証

### モニタリング

* 拡張モニタリング：無効

### 追加設定

#### データベース

* 最初のデータベース名：空白のまま
* DBパラメータグループ：magenta-magenta-mysql80
* オプショングループ：magenta-magenta-mysql80

#### バックアップ

* 自動バックアップ：有効
* バックアップ保持期間：30日間
* バックアップウィンドウ：ウィンドウを選択
* 開始時間：19:00 UTC , 0.5時間
* スナップショットにタグをコピー：有効

#### 暗号化

* 暗号化：しない

#### ログ

* ログのエクスポート：無効

#### メンテナンス

* マイナーバージョン自動アップグレード：有効
* メンテナンスウィンドウ：選択ウィンドウ
  * 開始：日曜日 , 20:00 , 0.5時間
* 削除保護：無効
