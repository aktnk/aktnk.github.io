# aktnk GitHub Pages README

[![GitHub Pages](https://github.com/aktnk/aktnk.github.io/actions/workflows/gh-pages.yml/badge.svg?branch=master&event=push)](https://github.com/aktnk/aktnk.github.io/actions/workflows/gh-pages.yml)
[![pages-build-deployment](https://github.com/aktnk/aktnk.github.io/actions/workflows/pages/pages-build-deployment/badge.svg?branch=gh-pages)](https://github.com/aktnk/aktnk.github.io/actions/workflows/pages/pages-build-deployment)

## はじめに

[hugo](https://gohugo.io/)に[Hugo Theme:Tranquilpeak](https://themes.gohugo.io/themes/hugo-tranquilpeak-theme/)を使い、["Aktnk Lab"という blog](https://aktnk.github.io/) を始めました。

## コンテンツ作成の手順

### 前提条件：開発環境

記事を作成している環境

- Windows 10 Pro.
- wsl2 + Ubuntu 20.04
- hugo : snap を使いインストール
- distrod + snapd : wsl2 環境では snap install hugo でエラーとなるため

### ０．GitHub リポジトリを作成

GitHub にまだリポジトリを登録していないときは下記の手順でコンテンツを作成し、GitHub に登録する

1. github にリポジトリを作成
1. `hugo new site (sitename)`を実行
   ```
   $ hugo new site (sitename)
   ```
1. hugo のテーマ tranguilpeak を導入
   ```
   $ cd (sitename)
   $ git init
   $ git submodule add https://github.com/kakawait/hugo-tranquilpeak-theme themes/tranquilpeak
   $ echo theme = \"tranquilpeak\" >> config.toml
   ```
1. 新しく作成する Web サイト用に config.toml を編集しカスタマイズ
1. `.gitignore`に`/public`と`.hugo_build.lock`を追加
1. `.github/workflows/gh-pages.yml`を作成
1. github リポジトリに登録
   ```
   $ git add .
   $ git commit -m "add base file"
   $ git remote add origin (githubリポジトリのURL)
   $ git push -u origin master
   ```

### １．GitHub リポジトリからクローンする

「０．GitHub リポジトリを作成」を行ったディレクトリで継続して「２．コンテンツ作成・編集」する場合、「１．GitHub リポジトリからクローンする」の作業は不要

1. `git clone --recursive (クローンするリポジトリのURL)`を実行
   ```
   $ git clone --recursive git@github.com:aktnk/aktnk.github.io.git
   ```
   - 本リポジトリは hugo themes を git submodule 指定しているため、`--recursive`オプションをつけて clone する。
   - もし、`--recursive`をつけ忘れた clone した場合は、clone して生成されたディレクトリ配下に移動し、`git submodule update --init --recursive`を実行すればよい
     ```
     $ cd aktnk.github.io
     $ git submodule update --init --recursive
     ```

### ２．コンテンツの作成・編集

1. `hugo new posts/(markdownファイル名)`を実行
   ```
   $ hugo new posts/(content_name.md)
   ```
1. 別ターミナルを起動し、`hugo server -D`で hugo サーバを起動
1. ブラウザーを起動し`http://localhost:1313/`を開く
1. ブラウザーのプレビューを確認しながら、posts/(content_name.md)を編集
1. コンテンツの編集が完了したら、別ターミナルで起動した hugo サーバを Ctrl+C で終了
1. `hugo -D`コマンドで Web ページを生成し、エラーが発生しないことを確認

### ３．github に登録

1. git add (追加したファイル)
1. git commit -m "(コミットメッセージ)"
1. git push origin master
1. github actions が正常に終了すると、gh-pages ブランチに web コンテンツが生成される

### 参考サイト

- [hugo 公式ドキュメント](https://gohugo.io/documentation/)
- [hugo-tranquilpeak-thema 公式ドキュメント](https://github.com/kakawait/hugo-tranquilpeak-theme/blob/master/docs/user.md)
- [hugo を使って爆速でブログを作成する](https://zenn.dev/harachan/articles/a043e9a756cae4)
- [hugo を使った爆速ブログをカスタマイズする](https://zenn.dev/harachan/articles/21d8f3a9f2ca4e)

## 補足：GitHub Actions による GitHub Pages の自動生成について

ローカル PC で markdown ファイルを作成すると、下記の流れで[https://aktnk.github.io/](https://aktnk.github.io/)は自動生成される

1. PC 上で hugo を使い、markdown ファイルを編集しコンテンツを作成
1. 作成した markdown ファイルをこのリポジトリに Push
1. このリポジトリにファイルが Push されると、
   [GitHub Actions for HUGO](https://github.com/peaceiris/actions-hugo)を使い、コンテンツが生成される
1. その後、[GitHub Actions for GitHub Pages](https://github.com/peaceiris/actions-gh-pages)を使い、gh-pages ブランチに登録される
1. gh-gapes ブランチは https://aktnk.github.io/ の GutHub Pages と指定しているため、Blog ページが自動的に更新される

### デプロイ自動化について参考にした情報

- [GitHub Actions による GitHub Pages への自動デプロイ](https://qiita.com/peaceiris/items/d401f2e5724fdcb0759d)
- [Hugo + GitHub Pages / Actions でブログを公開する](https://zenn.dev/bryutus/articles/hugo-github-pages-actions)
- [Hugo と Github Pages でブログを作る](https://sat8bit.github.io/posts/hugo-with-github-pages/)

## 謝辞

下記の OSS を使用しています。下記の OSS およびその contributors に感謝致します。

- [HUGO by @gohugoio](https://github.com/gohugoio/hugo)
- [A gorgeous responsive theme for Hugo blog framework : Tranquilpeak by @kakawait](https://github.com/kakawait/hugo-tranquilpeak-theme)
- [GitHub Actions for Hugo by @peaceiris](https://github.com/peaceiris/actions-hugo)
- [GitHub Actions for GitHub Pages by @peaceiris](https://github.com/peaceiris/actions-gh-pages)
