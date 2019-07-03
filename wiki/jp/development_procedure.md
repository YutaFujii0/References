**開発ルールに関するLa Sagrada Familia**:grin:

こちらに開発ルールを記録していく予定です。
気づいた度に付け足していき、あくまで開発スピードを優先することとします。
想定対象は着任者（コーディングスキルを問わない）です。

# 開発手順（藤井ドラフト）

1. ビジネス上の要件を検討
    * 顧客セグメンテーションとターゲティング
    * Painは何か
    * ユーザーはどんな「穴」を欲しがっているのか？（[＊](https://ferret-plus.com/6125)）
2. 要件定義
    * ビジネス要件（どういった作業やフローをシステム化するのか）
    * サイト要件（開発言語、サイトデザイン、ターゲットユーザ、OS）
    * インフラ要件（サーバー、ドメインを取得するか、固定IPを持つか等）
    * [参考サイト](https://uxmilk.jp/54677)

3. データベーススキーマの検討
    * 追加すべきテーブルはあるか
    * テーブル間の関係は（one to many? many to many?）ｎ：ｎ関係の場合はjoin tableを設ける
    * 追加・削除すべきカラムは存在するか
4. 画面遷移図・ルート
    * そもそもその画面はどのHTTPリクエストの結果表示されるのか
    * 画面の中に含まれるボタンやリンクは、どういったHTTPリクエストを生成するのか
    * [RESTful](https://railstutorial.jp/chapters/toy_app?version=4.2#aside-REST)な設計にすることが可能か
    * HTTP動詞（GET/POST/PATCH・PUT/DELETE）、コントローラー、コントローラーメソッドを明確にしておく
5. 開発（ここからコードが書ける！！:satisfied: ）
**（大前提）masterブランチでは原則として作業を行わないこと！**
    * 現行の本番最新ソースコードを自分のローカルに移す
    * ローカルで作業ブランチを作成する（そのブランチに移る）
    * 作業の区切りで`git add`, `git commit`を行う
    >コミットメッセージは現在形で書くようにしましょう。Gitのモデルは、(単一のパッチではなく) 一連のパッチとしてコミットされます。そのため、コミットメッセージを書くときには、そのコミットが「何をしたのか」と履歴スタイルで書くよりも「何をする」ためのものなのかを書く方が、後から見返したときにわかりやすくなります。さらに、現在形で書いておけば、Gitコマンド自身によって生成されるコミットメッセージとも時制が整合します。[Rails Tutorial #1](https://railstutorial.jp/chapters/beginning?version=4.0#sec-git_commit)
    * こまめにリモートにもpushしておく。ローカル環境が突然壊れてせっかく手を加えたソースコードが消えてしまっても、こまめにpushしておけば被害は最小限で済む（藤井は実際に仮想環境が壊れてすべてのソースがローカルから消えた経験あり）
    * ＜重要＞こまめにリモートのmasterブランチを自分のローカルにpullしておく。自分が作業をしている間にも他のエンジニアの機能追加などで最新のソースコードは変わっている。こまめにリモートmasterブランチをローカルにmergeして、コンフリクトを都度都度解消しておくことが、非常に好ましい

gitコマンドについては[こちら](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf)のcheat sheetをご参照

```
# bash(Git Bash)

# 自分の作業フォルダに移動
$ cd WORK_FOLDER

# 最初だけ実行！！
$ git init

# 作業を開始する前に最新の本番ソースコードを取得しておく
$ git pull origin master

# 作業ブランチを作成する（"YOUR_BRANCH"は自分で名前を付ける）
$ git checkout -b YOUR_BRANCH

# 作業に区切りがついたタイミングで適宜
$ git add FILE_NAME
$ git commit -m "COMMIT_MESSAGE"

# 作業に区切りがついたタイミングで適宜（現実的にcommitの都度行う必要はない）
$ git push origin YOUR_BRANCH

# ＜重要＞作成した機能を本番環境で使用（テスト含む）前に再度最新の本番ソースコードを取得する
$ git fetch origin master
# ＜重要＞取得した本番コードをマージする
$ git merge master
# なお、git fetchとgit mergeを合わせた作業はgit pull origin masterと実行しても可能

# この間にMERGE CONFLICTSを解消する

# コンフリクト解消したファイルの追加
$ git add FILE_NAME
$ git commit -m "resolve merge conflict"

# リモートレポジトリへのプッシュ
$ git push origin YOUR_BRANCH

# Githubをリモートとして用いている場合はプルリクエストを生成する
$ git pull-request -m "MESSAGE"

# リモートでのmasterブランチへのmergeは極力他のエンジニアのダブルチェックを経て行う

```

# コードレビュー
実装が終わったら、コードレビューを実施する
コードレビューの実施方法については別途Wikiを作成するのでそちらをご参照

# 設計思想

## 基本原則
- Railsにおける開発ではMVC( Model, View, Controller )モデルを採用する。人によってはパターンとかフレームワークと言ったりする。
> RailsではWebアプリケーションの構成にMVC (Model-View-Controller) というモデルを採用している（[Rails Tutorial](https://railstutorial.jp/chapters/toy_app?version=5.1#sec-mvc_in_action)）
- **convention over configuration**

cf. 他のフレームワークについての参考[Design Patterns](https://www.tutorialspoint.com/design_pattern/mvc_pattern.htm)

## Database
- [database normalization](https://en.wikipedia.org/wiki/Database_normalization)を意識して作成する
- ただし、上記の考え方はRelational Databaseに関しての概念であり、Nosql DBを用いる場合は気にする必要がない
- INDEXを貼ることでクエリの実行結果の効率化を図る。ただし、貼りすぎは書き込み時のパフォーマンスを悪化させるので注意
- 外部キーとしてIDを持つか、ただのint型としてIDを持つかは、テーブル間のリレーションの重要性に応じて検討する

## Model
* `rails g model` は使わない

## View
- partialを用いることを心掛ける
- layoutを用いる
- layoutは最小限のパターンにとどめるべきである

## Controller
* `rails g controller` は使わない
- ストロングパラメーターを用いてこちらの想定外のパラメータの影響を排除する（adminパラメータの排除など）
- SQLを生成するときは、 `sanitize`を行う
- クエリをDBに投げるときは[N+1問題](https://www.sitepoint.com/silver-bullet-n1-problem/)に注意する


## Route
- RESTfulなリクエストに対応したURLを極力採用する
- 具体的には、 `index`, `show`, `new`, `create`, `edit`, `update`, `destroy`に相当するアクションは以下のような慣例に従って名前付けをするべきである
- 当然、これら7つの基本機能に該当しない機能もあるため（検索による絞り込みなど）、それらはオリジナルの名前を付与する
- リレーションを持つ（外部キーとしないまでも他のテーブルのIDを持つ）場合は、それに合わせたURLを作成する

RESTfulリクエスト

verb|route|controller action|path name
-------|-------|-------|-------
GET|/users|index|users_path
GET|/users/:id|show|user_path
GET|/users/new|new|new_user_path
POST|/users|create|(none)
GET|/users/:id/edit|edit|edit_user_path
PATCH|/users/:id|update|(none)
DELETE|/users/:id|destroy|(none)

テーブルリレーションとURL
userとcompanyがn:1のリレーションを持っている場合

verb|route|controller action|path name
-------|-------|-------|-------
GET|/companies/:company_id/users|index|company_users_path
GET|/companies/:company_id/users/:id|show|company_user_path
...|...|...|...

## オブジェクトについて
Logicとしてまとめることができ、複数源からの呼び出しが想定されるもの（端的に言えば、いろんな場面でやる作業や一般的な作業）については、
`app/logics/`フォルダに格納する。RailsはMVCデザインを基本にしているが、当然ながらすべてがM・V・Cのどれかにフィットするオブジェクトとは限らない

- ファイル名はクラス名と一致している必要がある（でないとRailsが読み込めない）
- クラスメソッドは`.call`のみが外部から利用できる状態を大原則とする。ただしクラス内に`private`でサブ関数を定義することは全く問題ない
- クラス名は、そのLogicが何をするものなのかを明確にわかる名前にする。「動詞＋名詞」の形を意識するとよい

```
# app/logics/say_hello_to_audience.rb

class SayHelloToAudience
  def self.call  #引数をとっても良い
    [...]
  end

  private

    def self.submethod
      [...]
    end
end
```

## Global constantsについて
**大前提として、コードの中に定数がハードコーディングされていてはいけない**
アプリケーション内で使用する定数については、以下の通り管理する
- `config/constants.rb`ファイルにすべてを集約する（従い、修正が必要な場面や探す必要が生じる場面では一様にこのファイルを見ればよいことになる）
- 名前の重複を避ける、できるだけどのように使われるのかわかるような名前で
- なお、Modelに関わる定数については、`ActiveHash`（gemを参照のこと）を用いて運用する


```
# Not goodの例）
# 1000という数字は何を意味するか分かりづらいく、修正時にも修正漏れを起こしやすい
# ファイルパスについても同様のことがいえる
# これらは事前にグローバル定数として定義しておくべきである

  def self.set_path
    raise ArgumentError, "argument should not include / or ." if @filename.match(/(\/|\.)/)
    if @filename.match(/\A\d+\z/)
      quotient = @filename.to_i / 1000
      folder   = (quotient == 0 ? 1 : quotient * 1000).to_s
      @path_to_save = "/vagrant/alarmbox_internal/public/static/ab/alarms/" + folder + "/" + @filename + ".png"
    else
      @path_to_save = "/vagrant/alarmbox_internal/public/static/ab/alarms/" + "tmp/"       + @filename + ".png"
    end

    @path_to_save_pdf = "/vagrant/alarmbox_internal/public/static/ab/alarms/" + "tmp/" + @filename + ".pdf" if @original_url.end_with?(".pdf")
  end
```

## Configuration
* masterブランチにプッシュする際は `database.yml`ファイルにおける個別の設定を元に戻してから行う

## 外部ライブラリの活用について
* gem使用を検討するときはcontributersに注目する。ちゃんとメンテナンスされそうなgemかどうか
* 外部サイト（URL）を使用するときは、対象相手が信頼できるか調べる。HTTPリクエストのレスポンスでJavaScriptを返されてデータが破壊・盗まれたらとりかえしがつかない
