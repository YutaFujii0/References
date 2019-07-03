# テスト環境用のDBを準備する

* テスト環境のDBを作成する必要がある
* したがって、 `config/database.yml`ファイルでDBに関する環境パラメータを設定しつつ、DBを作成する
* ここで、現在のうちのDBはrailsのマイグレーションと決別しているので、DB作成にはDBクライアント側からすべてのテーブルを作成するか
* `db:crete db:schema:load`を用いる
* `db:schema:load`は既存のDBレコードを全削除するので絶対にproduction環境で実行しないこと


*database.yml* ファイル

```
test:
  adapter: mysql2
  encoding: utf8mb4
  database: alarmbox_test
  pool: 5
  username: alarmbox
  password: alarmbox
  socket: /var/lib/mysql/mysql.sock
  host: 192.168.33.11
```


DB作成手順

```
# 既存のDevelop環境のスキーマをコピー
bundle exec rake db:schema:dump

# Test環境のDB作成
bundle exec rake db:create RAILS_ENV=test

# Test環境のテーブル作成をmigrationファイルではなくschema.rbファイルから行う
bundle exec rake db:schema:load RAILS_ENV=test
```

なお、Rails5からは `bundle exec rake db:create`ではなく `rails db:create`になる



# RSpec の導入

Railsのテストツールは大きく分けて2つ存在するが、RSpecを導入する理由は大きく分けて2つ

* ネットの情報が多い（肌感覚でUnitTestの5倍以上存在する）
* 使用している企業が多い（結局上と同義だが、要は汎用的なスキルであることや、周りに聞ける人が多いということ）

Test tool|Pros|Cons
-------|-------|-------
Unit Test|Railsのデフォルト、DHHも推奨、公式Docが非常に丁寧|ネット情報が少ない
RSpec|ネット情報が多い、使用している企業が多い|公式Docが陳腐


Unit TestはRailsのデフォルトとして用意されているため、何も指定せずに `rails new`すると自動的にフォルダが作成されている（そのほか `rails generate model post`とかやるとついてくる）
Gemfileに以下の2つのgemを追加してbundle install

```
group :development, :test do
  [...]
  gem 'rspec-rails', '~> 3.8'
  gem 'factory_bot_rails'
end
```

RSpecのフォルダ構造は
RAILS_ROOT/spec/ を基準に
models/ controllers/ integration/ views/ features/ mailers/ helpers/ jobs/ system/ routing/ factories/*.rb
があり、FactoryBotがさらに support/フォルダを作成することを推奨している。[Factory bot rails : github](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#configure-your-test-suite)
Factorybotはなくてもfixtureを作ることができるが、ベストプラクティスとして推奨されていない[Better Spec](http://www.betterspecs.org/)

>This is an old topic, but it's still good to remember it. Do not use fixtures because they are difficult to control, use factories instead. Use them to reduce the verbosity on creating new data.

gemインストール後最初にすることは、初期ファイルの自動生成である（これをしないで自分でやろうとするとconfigで躓く）

```
$ rails generate rspec:install
```

できあがったファイルのなかで `rails_helper.rb`のconfigにこれを追加する。/spec/rails_helper.rb

```
RSpec.configure do |config|
  config.infer_spec_type_from_file_location! # ref.1

  config.expect_with :rspec do |c|
    c.syntax = :expect
  end # ref.2

  config.include FactoryBot::Syntax::Methods # ref.3
end

# 1. どうしてこれを記載しているのか理解できていない
#     https://relishapp.com/rspec/rspec-rails/v/3-8/docs/directory-structure

# 2.  Better Specsによる、ベストプラクティスに従った
#     http://www.betterspecs.org/

# 3.  Factory Bot RailsのRSPecに対する推奨設定
#     https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#configure-your-test-suite

```

# DatabaseCleanerの導入によるテストDBの自動トランケート

テスト用DBはテストの毎にレコードを生成してしまう（Rollbackするところもあるようだが）
これでは同じテストコードでも、DBの状況によって結果が変わってしまう可能性を排除できないため、テスト時にはいつもDBをクリーンにしておくことが理想的である
よく使われているのがDatabaseCleanerというgemのようなので、これを導入する

*Gemfile*

```
group :test do
  gem 'database_cleaner`
end
```

*spec/rails_helper.rb*

```
require 'capybara/rspec'

#...

RSpec.configure do |config|

  config.use_transactional_fixtures = false

  config.before(:suite) do
    if config.use_transactional_fixtures?
      raise(<<-MSG)
        Delete line `config.use_transactional_fixtures = true` from rails_helper.rb
        (or set it to false) to prevent uncommitted transactions being used in
        JavaScript-dependent specs.

        During testing, the app-under-test that the browser driver connects to
        uses a different database connection to the database connection used by
        the spec. The app's database connection would not be able to access
        uncommitted transaction data setup over the spec's database connection.
      MSG
    end
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, type: :feature) do
    # :rack_test driver's Rack app under test shares database connection
    # with the specs, so continue to use transaction strategy for speed.
    driver_shares_db_connection_with_specs = Capybara.current_driver == :rack_test

    unless driver_shares_db_connection_with_specs
      # Driver is probably for an external browser with an app
      # under test that does *not* share a database connection with the
      # specs, so use truncation strategy.
      DatabaseCleaner.strategy = :truncation
    end
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.append_after(:each) do
    DatabaseCleaner.clean
  end

end
```



# FactoryBotの使い方

RSpecを用いている場合は、
spec/factory/*.rbファイルが自動的に最初に読み込まれる。この設定を解除したい場合はRails.configを変更しておく必要がある[]()


FacrotyBotを用いたtestdataの作成方法（メソッド）は多岐にわたるが、例えば以下のようなもの。詳細は[公式Doc](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#using-factories)参照

```
# This will guess the User class
FactoryBot.define do
  factory :user do
    first_name { "John" }
    last_name  { "Doe" }
    admin { false }
  end
end

# Returns a User instance that's not saved
user = build(:user)

# Returns a saved User instance
user = create(:user)

# Returns a hash of attributes that can be used to build a User instance
attrs = attributes_for(:user)

# Returns an object with all defined attributes stubbed out
stub = build_stubbed(:user)

# Passing a block to any of the methods above will yield the return object
create(:user) do |user|
  user.posts.create(attributes_for(:post))
end
```

>It is highly recommended that you have one factory for each class that provides the simplest set of attributes necessary to create an instance of that class. If you're creating ActiveRecord objects, that means that you should only provide attributes that are required through validations and that do not have defaults. Other factories can be created through inheritance to cover common scenarios for each class.

# DeviseによるLoginの実装

userモデルのfixture構築に関しては[こちら](https://github.com/plataformatec/devise/wiki/How-To:-Test-with-Capybara)
controllerテストにおける環境設定に関しては[こちら](https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-%28and-RSpec%29#controller-specs)
を参照する

最終的な例はこちら（spec/controllers/member_change_plan_controller_spec.rb）

```
require 'rails_helper'
require_relative "../support/devise"

RSpec.describe Member::ChangePlanController, type: :controller do

  describe "GET /change_plan" do
    login_user

    context "when logged in" do
      # note the fact that you should remove the "validate_session" parameter if this was a scaffold-generated controller
      it { expect(subject.current_user).to_not eq(nil) }
    end
    # To test if it get status 200
    it "when change_plan page clicked" do
      get :show
      expect(response).to have_http_status(:success)
    end
  end

  describe "POST /change_plan" do
    # test it get 200 and create new change_plan record when given a valid change

    # test it get 200 and won't create any new change_plan record when given am invalid change

    # test it creates a new unsubscribe form when given a change to zero plan

  end

  # method that something to login before conduct each test

end
```