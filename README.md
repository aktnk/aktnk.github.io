# aktnk GitHub Pages README

[![GitHub Pages](https://github.com/aktnk/aktnk.github.io/actions/workflows/gh-pages.yml/badge.svg?branch=master&event=push)](https://github.com/aktnk/aktnk.github.io/actions/workflows/gh-pages.yml)
[![pages-build-deployment](https://github.com/aktnk/aktnk.github.io/actions/workflows/pages/pages-build-deployment/badge.svg?branch=gh-pages)](https://github.com/aktnk/aktnk.github.io/actions/workflows/pages/pages-build-deployment)

## はじめに

[hugo](https://gohugo.io/)に[Hugo Theme:Tranquilpeak](https://themes.gohugo.io/themes/hugo-tranquilpeak-theme/)を使い、["Aktnk Lab"というblog](https://aktnk.github.io/) を始めました。  

## 補足

下記のような手順で、コンテンツを公開しています。

### 最初は…

下記の手順で、公開用コンテンツをこのリポジトリに登録していました。

1. PC上でhugoを使い、markdownファイルを編集しコンテンツを作成します。
1. hudo -Dで公開用のコンテンツを生成します。
1. publicディレクトリに生成されたファイルをこのリポジトリにPushします。
1. コンテンツが公開されます。

### 今は…

1. PC上でhugoを使い、markdownファイルを編集しコンテンツを作成します。
1. 作成したmarkdownファイルをこのリポジトリにPushします。
1. このリポジトリにファイルがPushされると、
[GitHub Actions for HUGO](https://github.com/peaceiris/actions-hugo)を使い、コンテンツを生成します。
1. その後、[GitHub Actions for GitHub Pages](https://github.com/peaceiris/actions-gh-pages)を使い、gh-pagesブランチに登録します。
1. gh-gapesブランチは https://aktnk.github.io/ のGutHub Pagesと指定しているため、Blogページが自動的に更新されます。

## 謝辞

下記のOSSを使用しています。下記のOSSおよびそのcontributorsに感謝致します。

- [HUGO by @gohugoio](https://github.com/gohugoio/hugo)

- [A gorgeous responsive theme for Hugo blog framework : Tranquilpeak by @kakawait](https://github.com/kakawait/hugo-tranquilpeak-theme)

- [GitHub Actions for Hugo by @peaceiris](https://github.com/peaceiris/actions-hugo)

- [GitHub Actions for GitHub Pages by @peaceiris](https://github.com/peaceiris/actions-gh-pages)
