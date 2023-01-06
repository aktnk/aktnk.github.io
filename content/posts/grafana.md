---
title: "INKBIRDから取得した温度・湿度データをGrafanaでモニターする"
date: 2023-01-05T09:39:32+09:00
draft: false
categories: [IoT, server]
tags: [grafana, influxDB, docker, INKBIRD, ITH-12S, IBS-TH2]
thumbnailImagePosition: left
thumbnailImage: /images/grafana/monitor.png
---
{{< toc >}}

# はじめに

今回は["INKBIRDから取得した温度・湿度データをInfluxDBに記録する"](/2022/12/30/influxdb/)でInfluxDBに取り込んだ温度・湿度データをGrafanaでモニターしてみます。なお、INKBIRDのセンサー値をInfluxDBに保存する部分については["INKBIRDから取得した温度・湿度データをInfluxDBに記録する"](/2022/12/30/influxdb/)で記載していますので、ここでは説明を省きます。

# 全体構成

システム全体の構成を下記に記載します。
{{< svg src="../../static/images/grafana/system_diagram.drawio.svg" title="システム全体構成" >}}
- 今回は Windows PCのUbuntu20.04 にインストールしたdockerを使い、grafanaとinfluxdbとchronografコンテナを起動します。
- Windows PC上のpythonで作成したスクリプトをWindowsのタスクスケジューラから定期的に起動し、INKBIRDのBluetooth温度湿度計からデータを取得し、influxdbにデータを登録します。
- grafanaコンテナでinfluxdbに登録されたデータをグラフ表示させます。

# コンテナの作成

## docker-compose.ymlの作成

- influxdb,chronografはdocker公式イメージ[https://hub.docker.com/_/influxdb](https://hub.docker.com/_/influxdb) 、[https://hub.docker.com/_/chronograf](https://hub.docker.com/_/chronograf) のv.1.8.10を使用します。
- grafanaはGrafana Labsが公開しているgrafana-oss https://hub.docker.com/r/grafana/grafana-oss からv.8.5の最新版であるv.8.5.15  を使用します。なお、Grafana Labsが公開しているドキュメント[Migrate from previous Docker containers versions](https://grafana.com/docs/grafana/v8.5/installation/docker/#migrate-from-previous-docker-containers-versions) に従い、grafana-ossのuser IDを"104"に指定しています。
- 各コンテナとも設定した情報がコンテナを削除した際に失われないように、ボリュームを指定しています。

{{< gist aktnk 1f3a82d49fcc2bdd4ccf6611e1c0982f "docker-compose.yml" >}}

## コンテナの起動

コンテナを起動するため`docker-compose up`を実行します。するとgrafana-oss、influxdb、chronografコンテナのログが表示されます。ここでエラーが発生してコンテナが停止してしまう場合は、エラーメッセージを確認し対応します。
```
$ docker-compose up
<省略>

```
そして、ブラウザで`http://localhost:3000/` にアクセスして、grafanaのログイン画面が表示されることを確認します。確認できれば、<kbd>Ctrl</kbd>+<kbd>C</kbd>でコンテナを停止した後、`docker-compose up -d`として起動しなおします。
```
$ docker-compose up -d
<省略>

```

# GrafanaにInfluxDBのデータを表示させる

## Grafanaのadminパスワードを設定する

再度ブラウザで`http://localhost:3000/`にアクセスするとログイン画面が表示されるので、ユーザ名`admin`、パスワード`admin`でログインします。ログインするとパスワードの変更画面が表示されるので、パスワードを変更します。  

## GrafanaのデータソースにInfluxDBを登録する

1. 左側メニュから[Configure]>[Data sources]を選択します。
{{< figure src="/images/grafana/datasource_menu.png" link="/images/grafana/datasource_menu.png" title="データソースの設定" >}}
1. 表示されたページの右上にある[Add data source]ボタンを押し、表示されたメニューからInfulxDBを選択します。
{{< figure src="/images/grafana/select_influxdb.png" link="/images/grafana/select_influxdb.png" title="InfluxDBの選択" >}}
1. InfluxDBの設定メニューが表示されるので、IfluxDBに設定した情報を入力します。
{{< figure src="/images/grafana/setting_influxdb_1.png" link="/images/grafana/setting_influxdb_1.png" title="InfluxDBの設定(1)" >}}
{{< figure src="/images/grafana/setting_influxdb_2.png" link="/images/grafana/setting_influxdb_2.png" title="InfluxDBの設定(2)" >}}
1. [Save & test]ボタンを押します。"Data source is workind"と表示されることを確認します。表示されない場合、設定内容に間違いがないか確認します。
{{< figure src="/images/grafana/save_and_test.png" link="/images/grafana/save_and_test.png" title="設定の保存とテスト" >}}

## Dashboardを作成する

1. [Create]>[Dashboard]を選択します。
{{< figure src="/images/grafana/create_dashboard.png" link="/images/grafana/create_dashboard.png" title="ダッシュボードの作成" >}}
1. [Add a new panel]をクリックします。
{{< figure src="/images/grafana/add_panel.png" link="/images/grafana/add_panel.png" title="パネルの追加" >}}
1. Query欄でInfuxDBから表示するデータを指定します。そして、右側の各グラフ設定メニューを設定し、[Apply]ボタンを押します。
{{< figure src="/images/grafana/edit_panel.png" link="/images/grafana/edit_panel.png" title="パネルの設定" >}}
1. 上記を繰り返し最終的に下記のようなダッシュボードを作成しました。
{{< figure src="/images/grafana/monitor.png" link="/images/grafana/monitor.png" title="設定後のダッシュボード" >}}

# 最後に

今回はGrafanaにInfluxDBを連携させ、Inkbirdの温度・湿度データを表示するダッシュボードを作成しました。
Grafanaはアラートの設定ができます。アラートの一例としてGmailにメールを送信する設定をしてみようと思います。