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

---

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

---

## [3.3 テストから始める](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-getting_started_with_testing)
Aboutページなどを追加する前に「自動化テスト」を作成する習慣を身に着けます。

<details><summary>テスト駆動開発 (TDD) のアプローチ</summary>

1. **Red**: まず「正しいコードがないと失敗するテスト」を書く。
2. **Green**: テストがパスするための最小限のコードを書く。
3. **Refactor**: 機能を変更せずにコードを改善（リファクタリング）する。

チュートリアルではテストファイルを使ってテストを記述し、実行することでエラー（Red）を確認し、のちにパス（Green）するようにコードを修正するサイクルを回します。これにより回帰バグを防ぎ安全に開発を進めることができます。
</details>

---

## [3.4 少しだけ動的なページ](https://railstutorial.jp/chapters/static_pages?version=7.0#sec-slightly_dynamic_pages)
作成した静的ページを編集し、ページごとに `<title>` が変わるようにします。

<details><summary>タイトルの動的変更とリファクタリング</summary>

各ページの`<title>`タグの内容を編集し、「<ページ名> | Ruby on Rails Tutorial Sample App」という形式に変更します。
レイアウトファイル（`app/views/layouts/application.html.erb`）を活用して、各ビューファイルの重複するコード（HTMLの基本構造など）を一つにまとめる（リファクタリング）作業を行います。
</details>

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

<details><summary>Renderへのデプロイ操作</summary>

チュートリアルではオートデプロイをオフに設定しています。変更を本番環境へ反映させるために、手動で「Deploy latest commit」を実行し、完了後に正しく表示されるか確認します。
</details>
