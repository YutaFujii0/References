## GoogleAPI環境構築

[このページ](https://developers.google.com/analytics/devguides/reporting/core/v4/quickstart/service-py)で手順を確認

**Service Account**

* Project作成
* Service Account作成
* CredentialキーをJSONで発行（ダウンロードされる）
* キーを大切に保存する
* ユーザーにサービスアカウントの権限を付与するの部分で、"xxx@gmail.com"を追加

**APIとサービス**

* ProjectにおけるAPIとサービス画面でGoogle Analyticsを追加

**Google Analytics ユーザー管理**

* 上記のService Accountで割り当てられたメンバーEmail（analytics@xxx.iam.gserviceaccount.com）を追加
* これを追加していないと、チュートリアルにあるHelloAnalyticsを流したときに"Analytics Google API Error 403: “User does not have any Google Analytics Account"というエラーが出てしまう

**開発環境の整備**

* 開発環境用のGoogleAnalyticsAccountを作成し（URL：開発環境のURLで）、このURLに付与されたTrackerIDで `<script>`タグを埋め込む必要があるほか
* そもそもGoogleTagManagerと連携するための `<script>`タグが埋め込まれていないので、これも併せて埋め込む必要がある

```js
</head>


  <!-- Google Tag Manager -->
  <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
  new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
  j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
  'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-xxxxxxx');</script>
  <!-- End Google Tag Manager -->
</head>

<body>
  <!-- Google Tag Manager (noscript) -->
  <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-xxxxxxx"
  height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
  <!-- End Google Tag Manager (noscript) -->

<body>
```


## 外部ライブラリ準備

```
pip install google-api-python-client
pip install oauth2client
```

## Metrics and Dimensions

基本的な構文の書き方
https://developers.google.com/analytics/devguides/reporting/core/v4/basics

最低必要な3つのパラメータをリクエストJSONに乗っける必要がある

* A valid view ID for the viewId field.
* At least one valid entry in the dateRanges field.
* At least one valid entry in the metrics field.

特に重要なのはMetricsで、取りたい定量指標を指す。基本的にはリクエストJSONのこの部分だけ変えていくことになると思われる。2つ以上同時にリクエストできる。
Metricsと並んでよく使うものがDimensionsという項目で、MetricsのDimensionごとの内訳が表示できる。
例えば、アクセスURLごとのセッション数を取りたい場合はdimensionがアクセスURLに相当する `pagePathLevel1`で、Metricsが `sessions`となる

MetricsになっているものをDimensionに使うことはできないし、逆も然り。

```
{
  'viewId': VIEW_ID,
  'dateRanges': [{'startDate': '7daysAgo', 'endDate': 'today'}],
  'metrics': [
      {'expression': 'ga:sessions'}
  ],
  'dimensions': [{'name': 'ga:pagePathLevel1'}]
}

```


Metrix一覧
https://developers.google.com/analytics/devguides/reporting/core/dimsmets


# クエリ小ネタ

ページごとのアクセスを見るときの注意点

Dimensionに `pagePath`というものがあるが、これだとすべてのURLごとに結果が戻ってくるので、IDを含むようなURLもそれぞれ別々に集計されてしまう。ルートURLから何階層まで区切りで集計するかを指定できるのが `pagePathLevel1`（２や３もある）であり、これを使うことが現実的。

例）
```
...
ga:pagePath: /posts/18266
Date range: 0
ga:sessions: 0
ga:exits: 1
ga:pagePath: /posts/18295
Date range: 0
ga:sessions: 0
ga:exits: 1

```

## Filetering

Metrics, Dimensionともに一定条件を満たすもののみを返すようにフィルターをかけることができる

```
# 例）postsページだけを対象にMetricsを見たい場合のDimemsionフィルター

    body={
        'reportRequests': [{
            'viewId': VIEW_ID,
            'dateRanges': [{'startDate': '7daysAgo', 'endDate': 'today'}],
            'metrics': [
                {'expression': 'ga:sessions'},
            ],
            'dimensions': [
                {'name': 'ga:previousPagePath'},
                {'name': 'ga:pagePathLevel1'}
            ],
            "dimensionFilterClauses": [{
                "filters": [{
                    "dimensionName": "ga:pagePathLevel1",
                    "operator": "EXACT",
                    "expressions": ["/posts"]
                }]
            }]
        }]
    }
```

# Databaseとの連携

Google Analyticsは当然ながらDBとの連携がなされていないので、例えば「ユーザーID1234のページ遷移動向」などは調べることができない。


# Dimensions

name|usage
----------|-----------
ga:exitScreenName|screen name where user exit the app
ga:channelGrouping|display channel groups
