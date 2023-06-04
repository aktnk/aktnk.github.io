---
title: "Photogrammetry - MVE"
date: 2023-06-04T15:10:14+09:00
draft: false
categories: [Photogrammetry]
tags: [Multi-View Environment, Photogrammetry, Docker]
thumbnailImagePosition: left
thumbnailImage: /images/multi_view_env/result_ply.png
---

{{< toc >}}

# はじめに

今頃？　と思うかもしれませんが、最近 Photogrammetry に興味を持ちました。  
今回は、iPhoneやデジカメのカメラを使って撮影した画像から、PLYファイルなどの3次元データを取得することが可能なOSSの１つ、[Multi-View Environment "https://github.com/simonfuhrmann/mve"](https://github.com/simonfuhrmann/mve)を試してみようと思います。

## Photogrammetryとは

Wikipediaで調べてみると、『[写真測量法 (しゃしんそくりょうほう 英: Photogrammetry) とは、写真画像から対象物の幾何学特性を得る方法である。](https://ja.wikipedia.org/wiki/%E5%86%99%E7%9C%9F%E6%B8%AC%E9%87%8F%E6%B3%95)』と出てきます。  
そこで、他にもGoogle検索してみると、  
["https://old2-lecture.nakayasu.com/"のPhotogrammetry入門](https://old2-lecture.nakayasu.com/index_p=3568.html)には、下記のように記載されており、地形測量技術として古くからある技術だということがわかりました...

> 写真測量法（Photogrammetry）の歴史
>
> 写真測量法＝Photogrammetryは、写真画像から対象物の幾何学特性を得る手法であり、歴史的には地形測量技術として発展してきた。ある地点の位置や地点間の距離を測定することは、人間の経済活動の根幹を支える基盤技術であり、その歴史は古代エジプトのピラミッド建設まで遡る。写真測量の歴史も古く写真技術の登場と同じ19世紀に遡る。航空写真から地図を作成する技術は軍事目的としても重要視されていた。その後、人工衛星を利用したGPS（Global Positioning Satellite）測量、衛星写真等や地図を統合したGoogle Mapが登場するなど、地球規模の測量システムは進化し続けている。ここで紹介している物体の三次元形状を測定するスキャニング技術として位置付けられているPhotogrammetryは地形の測量技術の進化形の一つである。

# Multi-View Environment

今回試してみるPhotogrammetryのOSSであるMulti-View Environmentは[https://github.com/simonfuhrmann/mve](https://github.com/simonfuhrmann/mve)に、[解説は同リポジトリのWiki](https://github.com/simonfuhrmann/mve/wiki)にて公開されています。  
このリポジトリにはDockerfileも登録されていますが、ベースにしているAiplineLinuxが3.10と少々古い状態です。

また、Qiitaの記事[「Photogrammetry on Docker ～ サーバ屋さんもXRしたい ～"https://qiita.com/kotauchisunsun/items/93c037de40335bd398ab"」](https://qiita.com/kotauchisunsun/items/93c037de40335bd398ab)には、[マルチステージビルドを利用したdocker環境](https://github.com/kotauchisunsun/alpine-mve)を使い3Dデータを生成した例が紹介されています。  
このDockerfileではrootユーザのまま実行する環境になっていたり、dockerコンテナを特権モードで実行していたりします。

上記を踏まえて、下記の修正を織り込んだものを利用します。
* 今回AlpineLinuxをできる限り新しいものにする
* --privilegedオプションを指定せずに実行できるようにする
* また、docker composeを使うことでマウント指定をコマンドラインで指定せずとも実行できるようにする

なお、これらの改造を織り込んだソースファイルは[https://github.com/aktnk/mve](https://github.com/aktnk/mve)に公開しています。

## 写真の準備

何か身の回りのものを写真で撮影し、MVEで処理してみようと思います。  
そこで、建物の横に植えられている木々の周りで90枚前後iPhoneで撮影しました。  
{{< figure src="/images/multi_view_env/iphone_photos.png" link="/images/multi_view_env/iphone_photos.png" title="用意した写真" >}}

## Reconstrction Pipelineの実行

* ソースファイルの取得とディレクトリの権限変更
  ```
  $ git clone https://github.com/aktnk/mve.git
  $ cd mve
  $ chmod -R 777 data
  ```
* 写真データのコピー
  ```
  $ copy (撮影した写真path名/*.jpg) ./data/image/
  ```
* コンテナのビルドと起動
  ```
  $ docker compose up -d --build
  ```
* 3Dデータ生成
  ```
  $ docker compose exec mve ./reconst_pipe.sh
  ```

なお、上記を実行したところ、途中でファイルがないとエラー終了してしまいました。そこで、もう一度同じ`docker compose exec mve ./reconst_pipe.sh`を実行したところ、エラーにならず処理が終了しました。2時間程度掛かりました。何故だろう…？

## 生成された3Dデータの確認

今回は、PLYファイルを表示可能なMeshLabのWindows版を起動し、data/scene/surface-L2-clean.plyを[File]>[Import Mesh...]からインポートすると、下記のように表示されました。  
{{< figure src="/images/multi_view_env/result_ply.png" link="/images/multi_view_env/result_ply.png" title="生成された3Dデータ" >}}

* 写真に写った木々が、再構成された3次元で配置されており、ちょっとビックリ！
* もちろんマウスでグリグリでき、上からも下から...視点を変えて見ることができます。
* 全てが綺麗に再現されている訳ではなく、データがない部分もあるようです。

## 他にも写真を取り実行してみた...

上記で気をよくして、他にも撮って処理してみました。処理は最後まで終わりましたが、いずれも見たかった物体が表示されませんでした。
* マイカーの周りで撮影した30枚の写真
* 建物ものの入り口で撮影した50枚の写真

何故だろう…？