# ColorMeShopApi API client
GMOベポパ が提供している ColorMeショップ の API を PHP から利用するためのライブラリです。

現在一部のみ実装済み。

## Environment
* PHP 8.1 以上
* composer

## Installation
利用するプロジェクトの `composer.json` に以下を追加する。

```composer.json
"repositories": {
    "color-me-shop-api": {
        "type": "vcs",
        "url": "https://github.com/shimoning/color-me-shop-api.git"
    }
},
```

その後、以下のコマンドでインストールが可能になる。

```bash
composer require shimoning/color-me-shop-api
```

## Preparation
API を利用するためには利用登録が必要。

### デベロッパー登録
1. [開発者サイト](https://developer.shop-pro.jp/) の「デベロッパー登録」から登録。
2. アプリを作成
3. `クライアントID` と `クライアントシークレット` を控える
4. `リダイレクトURI` を設定し、疎通を確認しておく

### ショップ登録
[カラーミーショップページ](https://shop-pro.jp/) の「無料お試し」から登録。
お試し期間は **30日** しかないので、開発は計画的に。


## How to Use
利用方法。

### エラーハンドリング
API からエラーが返ってきた場合 `Communicator\Errors` クラスで返却する。
基本的には [エラー仕様書](https://developer.shop-pro.jp/docs/colorme-api#section/API/%E3%82%A8%E3%83%A9%E3%83%BC) に基づいたエラーが返ってくるが、APIによっては仕様を無視したエラーが返ってくるので気をつけること。

```php
if ($result instanceof Communicator\Errors) {
    foreach ($tokenOrErrors->all() as $error) {
        // @var Entities\Error
        $error->getMessage();
    }
}
```

### OAuth
#### 共通準備
以下のものをPHPで扱えるようにしておく。
* `クライアントID`
* `クライアントシークレット`
* `リダイレクトURI`

以降のコードサンプルでは、それぞれ `$clientId`, `$clientSecret`, `$redirectUri` として表現する。
OAuth に使う情報をあらかじめ作っておく。

```php
$oAuthOptions = new Entities\OAuth\Options($clientId, $clientSecret, $redirectUri);
```

#### OAuthアプリケーションの登録用URLの取得
```php
$oAuthScopes = new Values\Scopes([
    Constants\AuthScope::READ_PRODUCTS,
    Constants\AuthScope::READ_SALES,
]);
$oAuthUri = (new Client)->getOAuthUrl($oAuthOptions, $oAuthScopes); // https://api.shop-pro.jp/oauth/authorize?client_id=ff....
```

#### 認可コードをアクセストークンに交換
上記で取得したURLを開くとショップのログインや認可を操作する画面に移動する。
捜査を行なった後 `リダイレクトURI` に遷移する。
その時クエリに `code=....`　として `認可コード` がついてくるので、それを使う。

```php
$queryData = filter_input_array(INPUT_GET, $_GET); // Pure PHP
$queryData = $request->query(); // Laravel

$code = $queryData['code'];

$tokenOrErrors = (new Client)->exchangeCode2Token($oAuthOptions, $code);
if ($tokenOrErrors instanceof Communicator\Errors) {
    // error...
    // エラー仕様を無視したエラーが返ってくる...
    // おそらく oauth は厳密には api ではないので、規格が違うと思われる。
    // 追って対応予定 (OAuthError)
} else {
    // success!
    $token = $tokenOrErrors->getAccessToken(); // d461ab8XXXXXXXXXXXXXXXXXXXXXXXXX
}
```
ここで取得できた `$token` を保存しておく。
以降コードサンプルでは `$token` として表現する。

#### アクセストークンの有効期限について
ドキュメントには以下の様にあるので、一度生成すれば永続するものと思われる。
> アクセストークンに有効期限はありません

必要に応じてデータベースなどに、安全な方法で保存する。

### 受注
#### 受注データのリストを取得
* 第1引数: 検索条件 (optional)
* 第2引数: アクセストークン (optional *)
両方とも省略可能だが、初回利用時はアクセストークンは必須。
検索条件を省略した場合の条件は公式を参照。

```php
$client = new Client;

// 検索条件を省略パターン
$salePageOrErrors = $client->getSales(null, $token);

// 検索条件を設定し、アクセストークンを省略パターン
$searchParameters = new Entities\Sales\SearchParameters([
    'make_date_min' => '2022-12-01',
    'make_date_max' => '2022-12-31 23:59:59',
    'accepted_mail_state' => 'not_yet',
]);
$salePageOrErrors = $client->getSales($searchParameters);

// 両方省略するパターン (インスタンス作成時に引数としてトークンを与える必要がある)
$salePageOrErrors = $client->getSales();

// 結果の処理
if ($salePageOrErrors instanceof Communicator\Errors) {
    // error...
} else {
    // success!
    foreach ($salePageOrErrors->all() as $sale) {
        // @var Entities\Sales\Sale
        $sale;
    }

    // 取得できた件数
    $salePageOrErrors->count();

    // 全ての件数
    $salePageOrErrors->getTotal();

    // 指定した件数
    $salePageOrErrors->getTotal();

    // 取得開始位置
    $salePageOrErrors->getTotal();
}
```

#### 売上集計の取得
TODO: write

#### 受注データの取得
TODO: write

#### 受注データの更新
TODO: write

#### 受注のキャンセル
TODO: write

#### メールの送信
TODO: write

### 顧客
#### 顧客データの一覧を取得
TODO: write

#### 顧客データの取得
TODO: write

#### 顧客データを追加
(Unimplemented)

### 商品グループ
#### 商品グループ一覧を取得
TODO: write

### 決済
#### 決済設定の一覧を取得
TODO: write

### 配送
#### 配送方法一覧を取得
TODO: write

#### 配送日時設定を取得
(Unimplemented)

-----

## 未実装
* [ショップ](https://developer.shop-pro.jp/docs/colorme-api#tag/shop)
* [商品](https://developer.shop-pro.jp/docs/colorme-api#tag/product)
* [在庫](https://developer.shop-pro.jp/docs/colorme-api#tag/stock)
* [商品カテゴリー](https://developer.shop-pro.jp/docs/colorme-api#tag/category)
* [ギフト](https://developer.shop-pro.jp/docs/colorme-api#tag/gift)
* [ショップクーポン](https://developer.shop-pro.jp/docs/colorme-api#tag/shop_coupon)

-----

## CLI
コマンドラインから以下で実行可能。
終了する際は `exit` もしくは `Control + C` を入力。

```bash
php client
```

### .env
`.env` を設定することで、一部の変数が自動で生成され、確認等がしやすくなる。

`.env.example` を参考に設定する。
#### OAuth 用の 環境変数
* CLIENT_ID
* CLIENT_SECRET
* REDIRECT_URI

上記を設定することで、 `$oAuthOptions` を利用できる。
これと `$oAuthScopes` (自動ですべての権限として生成される) を利用することで、 OAuth の実施ができる。

#### アクセストークン
* TOKEN

上記を設定することで、 `$token` と `$client` が生成される。
そのまま `$client->getSales()` の様に利用できる。

-----

## ライセンスについて
当ライブラリは *MITライセンス* です。
[ライセンス](LICENSE) を読んでいただき、範囲内でご自由にご利用ください。
