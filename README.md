# [Ruby on Rails Tutorial 第3章](https://railstutorial.jp/chapters/static_pages?version=7.0#cha-static_pages) 学習用リポジトリ
第3章「ほぼ静的なページの作成」の学習記録と実行手順のまとめです。

## [3.1 セットアップ](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-sample_app_setup)
新しいRailsプロジェクト（`sample_app`）を作成し、環境をセットアップします。

**Directory: `~/environment/`**
```bash
$ cd ~/environment
# アプリケーションの新規作成。実行時の最新バージョンなどを指定。
$ rails _8.1.2_ new sample_app
$ cd sample_app/
```

<details><summary>Gemfileのセットアップとインストール</summary>

Gemfileをチュートリアルの指定に合わせて更新します。チュートリアル用のgemの追加などを行います。

**File: `Gemfile`**
対象のGemに合わせて設定を行った後、インストールを実行します。

**Directory: `sample_app/`**
```bash
$ bundle install
```
</details>

```bash
$ git add -A
$ git commit -m "Initialize repository"
```
---

GitHubでリモートリポジトリを作成する。

```bash
$ git remote add origin https://github.com/your-username/sample_app.git
$ git push -u origin main
```

## [3.2 静的ページ](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-static_pages)
静的なHTMLのみのページを作成するための初期準備として新しくブランチを作成します。

**Directory: `sample_app/`**
```bash
# 静的なページ用のトピックブランチ（static-pages）を作成して移動
$ git switch -c static-pages
```

<details><summary>コントローラとビューに関する作業</summary>

この工程では、`app/controllers` ディレクトリや `app/views` ディレクトリ内で作業を進めます。Home, Help などの静的ページ用のコントローラとアクション、及び対応するビューを作成する作業を行います。
コマンドでジェネレータなどを使用した場合はその都度ファイルが生成されるので確認しながら進めます。
</details>

**Directory: `sample_app/`**
```bash
$ rails generate controller StaticPages home help
```

**Directory: `sample_app/`**
```bash
$ git add -A
$ git commit -m "Add a Static Pages controller"
$ git push -u origin static-pages
```
---

HomeページのHTMLを修正する
**File: `app/views/static_pages/home.html.erb`**
```erb
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```

**File: `app/views/static_pages/help.html.erb`**
```erb
<h1>Help</h1>
<p>
  Get help on the Ruby on Rails Tutorial at the
  <a href="https://railstutorial.jp/help">Rails Tutorial Help page</a>.
  To get help on this sample app, see the
  <a href="https://railstutorial.jp/#ebook"><em>Ruby on Rails Tutorial</em>
  book</a>.
</p>
```

## [3.3 テストから始める](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-getting_started_with_testing)
Aboutページなどを追加する前に「自動化テスト」を作成する習慣を身に着けます。

<details><summary>テスト駆動開発 (TDD) のアプローチ</summary>

1. **Red**: まず「正しいコードがないと失敗するテスト」を書く。
2. **Green**: テストがパスするための最小限のコードを書く。
3. **Refactor**: 機能を変更せずにコードを改善（リファクタリング）する。

チュートリアルではテストファイルを使ってテストを記述し、実行することでエラー（Red）を確認し、のちにパス（Green）するようにコードを修正するサイクルを回します。これにより回帰バグを防ぎ安全に開発を進めることができます。
</details>

StaticPagesコントローラのデフォルトのテスト green
**File: `test/controllers/static_pages_controller_test.rb`**
```ruby
require "test_helper"

class StaticPagesControllerTest < ActionDispatch::IntegrationTest

  test "should get home" do
    get static_pages_home_url
    assert_response :success
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
  end
end
```
### テストの実行(Green:成功)
**Directory: `sample_app/`**
```bash
# $ rails db:migrate     不要だった。
$ rails test
2 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

一部のシステムでは以下のようなファイルがdbディレクトリの下に生成されていることがあります。(今回は作成されていなかった)
```
/db/test.sqlite3-0
```

念の為、生成されたデータベースファイルはGitに登録しないようにする。  
**File: `.gitignore`**
```
# Ignore the default SQLite database.
/db/*.sqlite3
/db/*.sqlite3-*
```

### Aboutページの追加
**File: `test/controllers/static_pages_controller_test.rb`**
```ruby
require "test_helper"

class StaticPagesControllerTest < ActionDispatch::IntegrationTest

  test "should get home" do
    get static_pages_home_url
    assert_response :success
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
  end

  test "should get about" do
    get static_pages_about_url
    assert_response :success
  end
end
```
### テストの実行(Red:失敗)
**Directory: `sample_app/`**
```bash
$ rails test
Error:
StaticPagesControllerTest#test_should_get_about:
NameError: undefined local variable or method 'static_pages_about_url' for an instance of StaticPagesControllerTest
    test/controllers/static_pages_controller_test.rb:15:in 'block in <class:StaticPagesControllerTest>'

3 runs, 2 assertions, 0 failures, 1 errors, 0 skips
```
**解釈:**
「`static_pages_about_url` というメソッドが見つからない」というエラーです。
これは、まだ `config/routes.rb` にabout用ページのルーティングが定義されておらず、このURLを生成するヘルパーメソッドが存在していないことを意味しています。

### about用のルーティングを追加する red
**File: `config/routes.rb`**
```ruby
Rails.application.routes.draw do
  get  "static_pages/home"
  get  "static_pages/help"
  get  "static_pages/about" # ← これを追加
end
```

### テストの実行(Red:失敗)
**Directory: `sample_app/`**
```bash
$ rails test
Failure:
StaticPagesControllerTest#test_should_get_about [test/controllers/static_pages_controller_test.rb:16]:
Expected response to be a <2XX: success>, but was a <404: Not Found>

```
**解釈:**
「ルーティング自体は定義されたが、対応するアクション（ページ）が存在しない」というエラーです。  
`<404: Not Found>`: リクエストされたページが見つからないことを示しています。URLにはアクセスできましたが、その先の処理を担当する `StaticPagesController` 内に `about` アクションが定義されていないため、Railsがページ（アクション）を見つけられずに404を返しています。

### aboutアクションの追加 red
homeやhelpと同じようにaboutアクションを追加します。
**File: `app/controllers/static_pages_controller.rb`**
```ruby
class StaticPagesController < Applicati onController

  def home
  end

  def help
  end

  def about # ← これを追加
  end
end
```

### テストの実行(Red:失敗)
**Directory: `sample_app/`**
```bash
$ rails test
Failure:
StaticPagesControllerTest#test_should_get_about [test/controllers/static_pages_controller_test.rb:16]:
Expected response to be a <2XX: success>, but was a <406: Not Acceptable>
```
**解釈:**
「アクションは存在するが、描画するための画面（ビュー）が存在しない」というエラーです。  
`<406: Not Acceptable>`: 要求された形式（今回の場合はHTMLファイル）をサーバーが提供できない場合に発生します。コントローラに `about` アクションは追加されましたが、Railsがそのアクション終了後に表示しようとした `about.html.erb` というテンプレートファイルがまだ作成されていないため、このエラーが発生しています。

<details><summary>【まとめ】Railsのテスト環境におけるエラーの段階的な変化（NameErrorと404と406の違い）</summary>

Railsの統合テスト（Integration Test）では、処理の進み具合によってエラー内容が段階的に変化します。これを知っておくと、不備がある場所を即座に特定できます。

1.  **NameError (ルーティングがない)**
    *   **内部挙動**: `static_pages_about_url` のようなヘルパーメソッドが定義されていないため、Rubyが「名前解決」に失敗して発生します。
    *   **なぜ NoMethodError ではないのか？**: 
        *   `NoMethodError` は `receiver.method` のように、明確な対象（レシーバ）に対して存在しないメソッドを呼んだ際に発生します。
        *   一方、今回の `get static_pages_about_url` のようにレシーバを明示せずに呼び出した場合、Rubyはこれを「変数」か「メソッド」か判別できず、広義の「名前が見つからない」という理由で `NameError` を投げます。
    *   **根拠**: Rubyの言語仕様として、未定義の変数やメソッドを参照した際に投げられる標準エラーです。
    *   **公式リンク**: [Rubyリファレンス (JA)](https://docs.ruby-lang.org/ja/latest/class/NameError.html) / [Ruby Docs (EN)](https://docs.ruby-lang.org/en/master/NameError.html)

2.  **404 Not Found (アクションがない)**
    *   **内部挙動**: ルーティング設定によりコントローラまでは特定できましたが、その中に対応するメソッド（`def about`）が存在しない状態です。Rails内部では `AbstractController::ActionNotFound` が発生し、最終的に HTTP 404 を返します。
    *   **根拠**: リクエストされた処理の窓口（アクション）が見つからないため「リソースなし」と判定されます。
    *   **公式リンク**: [Rails Guides - Missing Action](https://guides.rubyonrails.org/action_controller_overview.html#nitting-with-a-missing-action)

3.  **406 Not Acceptable (ビューがない)**
    *   **内部挙動**: アクションの実行は完了しましたが、Railsがデフォルトで探す「アクション名と同じ名前のテンプレート（`about.html.erb`）」が見つかりません。Rails内部では `ActionController::UnknownFormat` が発生し、HTTP 406 を返します。
    *   **根拠**: 「処理（アクション）はできたが、結果を表示するための形式（HTML等）が提供できない」という状態です。
    *   **公式リンク**: [Rails Guides - Rendering](https://guides.rubyonrails.org/action_controller_overview.html#rendering-at-the-controller-level)

このように、テストコードから投げられるエラーメッセージやHTTPステータスを確認することで、Railsの「どの部品（Routes / Controller / View）」に修正が必要かを正確に切り分けることができます。
</details>

### about用ビューの追加 green
**Directory: `sample_app/`**
```bash
$ touch app/views/static_pages/about.html.erb
```

**File: `app/views/static_pages/about.html.erb`**
```ruby
<h1>About</h1>
<p>
  <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
  is a <a href="https://railstutorial.jp/#ebook">book</a> and
  <a href="https://railstutorial.jp/screencast">screencast</a>
  to teach web development with
  <a href="https://rubyonrails.org/">Ruby on Rails</a>.
  This is the sample app for the tutorial.
</p>
```

### テストの実行(Green:成功)
**Directory: `sample_app/`**
```bash
$ rails test
3 runs, 3 assertions, 0 failures, 0 errors, 0 skips
```

---

## [3.4 少しだけ動的なページ](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-slightly_dynamic_pages)
作成した静的ページを編集し、ページごとに `<title>` が変わるようにします。

学習のため、レイアウトファイルのファイル名を一時的に次のように変更します。
**Directory: `sample_app/`**
```bash
$ mv app/views/layouts/application.html.erb layout_file
```

### StaticPagesコントローラのタイトルをテストする(Red:失敗)
**File: `test/controllers/static_pages_controller_test.rb`**
```ruby
require "test_helper"

class StaticPagesControllerTest < ActionDispatch::IntegrationTest

  test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Home | Ruby on Rails Tutorial Sample App" # ← これを追加
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
    assert_select "title", "Help | Ruby on Rails Tutorial Sample App" # ← これを追加
  end

  test "should get about" do
    get static_pages_about_url
    assert_response :success
    assert_select "title", "About | Ruby on Rails Tutorial Sample App" # ← これを追加
  end
end
```

red
**Directory: `sample_app/`**
```bash
$ rails test
# 3 runs, 6 assertions, 3 failures, 0 errors, 0 skips
3 runs, 9 assertions, 3 failures, 0 errors, 0 skips
```

### タイトルを追加する

**File: `app/views/static_pages/home.html.erb`**
```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>Home | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```

**File: `app/views/static_pages/help.html.erb`**
```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>Help | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>
      Get help on the Ruby on Rails Tutorial at the
      <a href="https://railstutorial.jp/help">Rails Tutorial help
      page</a>.
      To get help on this sample app, see the
      <a href="https://railstutorial.jp/#ebook">
      <em>Ruby on Rails Tutorial</em> book</a>.
    </p>
  </body>
</html>
```

**File: `app/views/static_pages/about.html.erb`**
```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>About | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
      is a <a href="https://railstutorial.jp/#ebook">book</a> and
      <a href="https://railstutorial.jp/screencast">screencast</a>
      to teach web development with
      <a href="https://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample app for the tutorial.
    </p>
  </body>
</html>

```

green
**Directory: `sample_app/`**
```bash
$ rails test
# 3 runs, 6 assertions, 0 failures, 0 errors, 0 skips
3 runs, 9 assertions, 0 failures, 0 errors, 0 skips
```

### レイアウトとERB（Refactor）
タイトルにERBを使ったHomeページのビュー green
**File: `app/views/static_pages/home.html.erb`**
```ruby
<% provide(:title, "Home") %> # ← これを追加
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title> # ← これを追加
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```

green
**Directory: `sample_app/`**
```bash
$ rails test
# 3 runs, 6 assertions, 0 failures, 0 errors, 0 skips
3 runs, 9 assertions, 0 failures, 0 errors, 0 skips
```

続いて、HelpページとAboutページも同様に変更します。

タイトルにERBを使ったHelpページのビュー green
**File: `app/views/static_pages/help.html.erb`**
```ruby
<% provide(:title, "Help") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>
      Get help on the Ruby on Rails Tutorial at the
      <a href="https://railstutorial.jp/help">Rails Tutorial help
      section</a>.
      To get help on this sample app, see the
      <a href="https://railstutorial.jp/#ebook">
      <em>Ruby on Rails Tutorial</em> book</a>.
    </p>
  </body>
</html>
```

タイトルにERBを使ったAboutページのビュー green
**File: `app/views/static_pages/about.html.erb`**
```ruby
<% provide(:title, "About") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
      is a <a href="https://railstutorial.jp/#ebook">book</a> and
      <a href="https://railstutorial.jp/screencast">screencast</a>
      to teach web development with
      <a href="https://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample application for the tutorial.
    </p>
  </body>
</html>

```
タイトルの可変部分をERBを使って置き換えたので、現在それぞれのページは次のような構造になっています。
```erb
<% provide(:title, "Page Title") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    Contents
  </body>
</html>
```
こうして見ると、HTMLの構造はtitleタグの内容も含めてどのページも完全に同じです。異なる点があるとすれば、bodyタグの内側のコンテンツだけです。

これはもうリファクタリングしてHTMLの重複した構造をDRYにするしかないでしょう。Railsにはそのためにapplication.html.erbという名前のレイアウトファイルがあります。事前にレイアウトファイルの名前をわざわざ変えておきましたが、いよいよ次のコマンドでファイル名を元に戻すことにします。
```bash
$ mv layout_file app/views/layouts/application.html.erb
```
このレイアウトファイルを有効にするには、前述のデフォルトのタイトル部分を次のERBコードに差し替えます。
```erb
<title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
```

サンプルアプリケーションのレイアウト green
**File: `app/views/layouts/application.html.erb`**
```erb
<!DOCTYPE html>
<html>
  <head>
    <!-- <title><%= content_for(:title) || "Sample App" %></title> -->
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
    <meta name="viewport" content="width=device-width,initial-scale=1"> # ← これを追加(注目)
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="application-name" content="Sample App">
    <meta name="mobile-web-app-capable" content="yes">
    <meta charset="utf-8"> # ← これを追加(UTF-8という文字エンコーディングを指定)
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= yield :head %>

    <%# Enable PWA manifest for installable apps (make sure to enable in config/routes.rb too!) %>
    <%#= tag.link rel: "manifest", href: pwa_manifest_path(format: :json) %>

    <link rel="icon" href="/icon.png" type="image/png">
    <link rel="icon" href="/icon.svg" type="image/svg+xml">
    <link rel="apple-touch-icon" href="/icon.png">

    <%# Includes all stylesheet files in app/assets/stylesheets %>
    <%= stylesheet_link_tag :app, "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%= yield %> # ← (注目)
  </body>
</html>
```

```
<%= yield %>
```
このコードは、各ページの内容をレイアウトに挿入するためのものです。  
レイアウトを使う際に、/static_pages/homeにアクセスするとhome.html.erbの内容がHTMLに変換され、<%= yield %>の位置に挿入される。

HTML構造を削除したHomeページ green
**File: `app/views/static_pages/home.html.erb`**
```erb
<% provide(:title, "Home") %>
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```

HTML構造を削除したHelpページのビュー green
**File: `app/views/static_pages/help.html.erb`**
```erb
<% provide(:title, "Help") %>
<h1>Help</h1>
<p>
  Get help on the Ruby on Rails Tutorial at the
  <a href="https://railstutorial.jp/help">Rails Tutorial help
  section</a>.
  To get help on this sample app, see the
  <a href="https://railstutorial.jp/#ebook">
  <em>Ruby on Rails Tutorial</em> book</a>.
</p>
```

HTML構造を削除したAboutページのビュー green
**File: `app/views/static_pages/about.html.erb`**
```erb
<% provide(:title, "About") %>
<h1>About</h1>
<p>
  <a href="https://railstutorial.jp/">Ruby on Rails Tutorial</a>
  is a <a href="https://railstutorial.jp/#ebook">book</a> and
  <a href="https://railstutorial.jp/screencast">screencast</a>
  to teach web development with
  <a href="https://rubyonrails.org/">Ruby on Rails</a>.
  This is the sample application for the tutorial.
</p>
```
green
```bash
$ rails test
# 3 runs, 6 assertions, 0 failures, 0 errors, 0 skips
3 runs, 9 assertions, 0 failures, 0 errors, 0 skips
```

HomeページをルートURLに設定する
**File: `config/routes.rb`**
```ruby
Rails.application.routes.draw do
  root "static_pages#home" # ← これを追加
  get "static_pages/home"
  get "static_pages/help"
  get "static_pages/about"
  .
  .
  .
end
```
---

## [3.5 最後に（マージとデプロイ）](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-static_pages_conclusion)
ここまでの作業をコミットし、mainブランチにマージ、そしてデプロイします。

**Directory: `sample_app/`**
```bash
# すべての変更内容をコミット
$ git add -A
$ git commit -m "Finish static pages"

# mainブランチに戻り、static-pagesブランチの変更をマージ
$ git switch main
$ git merge static-pages

# リモートリポジトリ（GitHubなど）にプッシュする前にテストを実行
$ rails test
$ git push
```
