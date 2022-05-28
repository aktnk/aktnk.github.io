# aktnk GitHub Pages README

[![GitHub Pages](https://github.com/aktnk/aktnk.github.io/actions/workflows/gh-pages.yml/badge.svg?branch=master&event=push)](https://github.com/aktnk/aktnk.github.io/actions/workflows/gh-pages.yml)
[![pages-build-deployment](https://github.com/aktnk/aktnk.github.io/actions/workflows/pages/pages-build-deployment/badge.svg?branch=gh-pages)](https://github.com/aktnk/aktnk.github.io/actions/workflows/pages/pages-build-deployment)

## はじめに

[hugo](https://gohugo.io/)に[Hugo Theme:Tranquilpeak](https://themes.gohugo.io/themes/hugo-tranquilpeak-theme/)を使い、["Aktnk Lab"というblog](https://aktnk.github.io/) を始めました。  

## コンテンツ作成の手順

### 前提条件：開発環境

記事を作成している環境

- Windows 10 Pro.
- wsl2 + Ubuntu 20.04
- hugo : snapを使いインストール
- distrod + snapd : wsl2環境ではsnap install hugoでエラーとなるため

### １．事前準備

基本的には最初に一度実施

1. githubにリポジトリを作成
1. `hugo new site (sitename)`を実行
    ```
    $ hugo new site (sitename)
    ```
1. hugoのテーマtranguilpeakを導入
    ```
    $ cd (sitename)
    $ git init
    $ git submodule add https://github.com/kakawait/hugo-tranquilpeak-theme themes/tranquilpeak
    $ echo theme = \"tranquilpeak\" >> config.toml
    ```
1. 新しく作成するWebサイト用にconfig.tomlを編集しカスタマイズ
1. `.gitignore`に`/public`と`.hugo_build.lock`を追加
1. `.github/workflows/gh-pages.yml`を作成
1. githubリポジトリに登録
    ```
    $ git add .
    $ git commit -m "add base file"
    $ git remote add origin (githubリポジトリのURL)
    $ git push -u origin master
    ```

### ２．コンテンツの作成・編集

1. `hugo new posts/(markdownファイル名)`を実行
    ```
    $ hugo new posts/(content_name.md)
    ```
1. 別ターミナルを起動し、`hugo server -D`でhugoサーバを起動
1. ブラウザーを起動し`http://localhost:1313/`を開く
1. ブラウザーのプレビューを確認しながら、posts/(content_name.md)を編集
1. コンテンツの編集が完了したら、別ターミナルで起動したhugoサーバをCtrl+Cで終了
1. `hugo -D`コマンドでWebページを生成し、エラーが発生しないことを確認

### ３．githubに登録

1. git add (追加したファイル)
1. git commit -m "(コミットメッセージ)"
1. git push origin master
1. github actionsが正常に終了すると、gh-pagesブランチにwebコンテンツが生成される

### 参考サイト

- [hugo公式ドキュメント](https://gohugo.io/documentation/)
- [hugo-tranquilpeak-thema公式ドキュメント](https://github.com/kakawait/hugo-tranquilpeak-theme/blob/master/docs/user.md)
- [hugoを使って爆速でブログを作成する](https://zenn.dev/harachan/articles/a043e9a756cae4)
- [hugoを使った爆速ブログをカスタマイズする](https://zenn.dev/harachan/articles/21d8f3a9f2ca4e)

## 補足：GitHub ActionsによるGitHub Pagesの自動生成について

ローカルPCでmarkdownファイルを作成すると、下記の流れで[https://aktnk.github.io/](https://aktnk.github.io/)は自動生成される

1. PC上でhugoを使い、markdownファイルを編集しコンテンツを作成
1. 作成したmarkdownファイルをこのリポジトリにPush
1. このリポジトリにファイルがPushされると、
[GitHub Actions for HUGO](https://github.com/peaceiris/actions-hugo)を使い、コンテンツが生成される
1. その後、[GitHub Actions for GitHub Pages](https://github.com/peaceiris/actions-gh-pages)を使い、gh-pagesブランチに登録される
1. gh-gapesブランチは https://aktnk.github.io/ のGutHub Pagesと指定しているため、Blogページが自動的に更新される

### デプロイ自動化について参考にした情報 

- [GitHub Actions による GitHub Pages への自動デプロイ](https://qiita.com/peaceiris/items/d401f2e5724fdcb0759d)
- [Hugo + GitHub Pages / Actionsでブログを公開する](https://zenn.dev/bryutus/articles/hugo-github-pages-actions)
- [Hugo と Github Pages でブログを作る](https://sat8bit.github.io/posts/hugo-with-github-pages/)

## 謝辞

下記のOSSを使用しています。下記のOSSおよびそのcontributorsに感謝致します。

- [HUGO by @gohugoio](https://github.com/gohugoio/hugo)
- [A gorgeous responsive theme for Hugo blog framework : Tranquilpeak by @kakawait](https://github.com/kakawait/hugo-tranquilpeak-theme)
- [GitHub Actions for Hugo by @peaceiris](https://github.com/peaceiris/actions-hugo)
- [GitHub Actions for GitHub Pages by @peaceiris](https://github.com/peaceiris/actions-gh-pages)
