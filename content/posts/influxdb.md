---
title: "INKBIRDから取得した温度・湿度データをInfluxDBに記録する"
date: 2022-12-30T10:36:13+09:00
draft: false
categories: [IoT, server]
tags: [influxDB, docker, INKBIRD, ITH-12S, IBS-TH2]
thumbnailImagePosition: left
thumbnailImage: /images/influxdb/disp_graph.png
---

{{< toc >}}

# はじめに

今回はINKBIRDのBluetooth温度湿度計から取得したデータを時系列データベースInfluxDBに登録してみます。

# 全体構成

システム全体の構成を下記に記載します。
{{< svg src="../../static/images/influxdb/system_diagram.drawio.svg" title="システム全体構成" >}}
- 今回は Windows PCのUbuntu20.04 にインストールしたdockerを使い、influxdbとchronografコンテナを起動します。
- Windows PC上のpythonで作成したスクリプトをWindowsのタスクスケジューラから定期的に起動し、INKBIRDのBluetooth温度湿度計からデータを取得し、influxdbにデータを登録します。
- chronografコンテナでinfluxdbに登録されたデータをグラフ表示させます。

# コンテナの作成

## docker-compose.ymlの作成

- influxdbはdocker公式イメージhttps://hub.docker.com/_/influxdb を使います。influxdbはver.2が公開されていますが、扱う上で参考となる情報が多いver.1の最新版ではv.1.8.10を使いました。
- chronografもdokcer公式イメージhttps://hub.docker.com/_/chronograf を使います。influxdb、chronografとも今回初めて使用します。chronografとinfluxdbのバージョンを合わせる必要があるかどうか調べましたが情報が見つけられませんでした。そこで、chronografのバージョンもinfluxdbと合わせてv.1.8.10を使用することしました。
- chronografのWebUIで設定した情報がコンテナを削除しても失われないように、ボリュームを指定しています。

{{< gist aktnk c635eb7d81f93222468d61c2eee61def "docker-compose.yml" >}}

## コンテナの起動

コンテナを起動するため`docker-compose up`を実行します。するとinfluxdbとchronografコンテナのログが表示されます。ここでエラーが発生してコンテナが停止してしまう場合は、エラーメッセージを確認し対応します。
```
$ docker-compose up
<省略>

```

# 温度湿度データ登録用データベースの準備

## InfluxDBのROOTユーザの登録

エラー終了することなくinfluxdbとchronografが動作していることを確認したら、ブラウザで`http://localhost:8888`にアクセスします。  
1. 左側に並んでいるアイコンの中から[InfuxDB Admin]の王冠アイコン`http://localhost:8888/sources/1/admin-influxdb/databases`をクリックします。
1. 次に、画面の左側タブ[Users]`http://localhost:8888/sources/1/admin-influxdb/users`をクリックします。
1. 右上の[+ Create User]ボタンを押し、ROOTユーザのユーザ名とパスワードを設定します。  
    - ROOTユーザ名:`testuser1`、パスワード:`ptestuser1`（今回例として設定したもの）
    {{< figure src="/images/influxdb/useradd.png" link="/images/influxdb/useradd.png" title="ユーザ登録後の画面" >}}

## 温度湿度データ登録用データベースの作成

次に、InfluxDB上に温度湿度データを登録するデータベースとそのデーターベース用のユーザを作成します。
- データベース名:hogehoge（今回例として設定したもの）

実際の作成はpython スクリプトcreatedb.pyで実施します。  
createbd.py内の`INFLUXDB_`で始まる環境変数に先ほど設定したROOTユーザ名・パスワード、データベース名、データベースユーザ名・パスワードを記載します。
また、ホスト名、ポート番号はdocker-compose.ymlでの設定を考慮し、`localhost`、`8086`になります。  
なお、温度湿度データをデータベース内に保持する期間を2か月`60d`としています。

{{< gist aktnk c635eb7d81f93222468d61c2eee61def "createdb.py" >}}

それではcreatebd.pyを実行し、InfluxDBに温度湿度登録用データベースを作成します。
```
(venv38) PS D:\projects\ble> python createdb.py
<省略>
```
実行後に、cronografのWebから、下記のようにデータベースhogehogeが作成されており、ユーザにtestuser2が追加されているはずです。
{{< figure src="/images/influxdb/createdb.png" link="/images/influxdb/createdb.png" title="温度湿度用データベース作成後の画面" >}}
{{< figure src="/images/influxdb/after_created.png" link="/images/influxdb/after_created.png" title="温度湿度用データベース作成後のユーザ一覧" >}}

# データの登録

## 定期実行用pythonスクリプトproc.pyの作成

以前に公開したBluetooth温度湿度計からデータを取得するpythonスクリプトをベースにinfluxdbに登録するコード（writedb()）を追加します。
なお、BluetoothLEで温度湿度データが取得できないときがあるため、取得できない場合は3回まではリトライするようにbre_read()にretryを指定しています。

{{< gist aktnk c635eb7d81f93222468d61c2eee61def "proc.py" >}}

## タスクスケジューラから定期実行されるtask.batの作成

先のpythonスクリプトをWindowsのタスクスケジューラから呼び出すために、タスクスケジューラに登録するBATファイル（task.bat）を作成します。
なお、pythonの仮想環境venv38上で実行するようにtask.batを配置しています。

{{< gist aktnk c635eb7d81f93222468d61c2eee61def "task.bat" >}}

このbatファイルをタスクスケジューラで5分間隔で実行するように設定しました。
{{< figure src="/images/influxdb/task_scheduler.png" link="/images/influxdb/task_scheduler.png" title="タスクスケジューラへの登録" >}}

# 登録データのグラフ化

chronografを使い、2つの温度湿度計データをグラフ化したものを下記に掲載します。
{{< figure src="/images/influxdb/disp_graph.png" link="/images/influxdb/disp_graph.png" title="温度湿度データのグラフ化" >}}

# 最後に

今回はinfluxdbとchronografにPythonスクリプトを用いて、温度湿度計の時系列データを登録し、グラフ表示できるようにしみました。
温度湿度を監視するにはグラフ化以外にも、設定した閾値を超えたらアラートのメールを出すように機能もほしくなります。いつか試してみようと思います。
