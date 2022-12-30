---
title: "INKBIRDから取得した温度・湿度データをInfluxDBに記録する"
date: 2022-12-30T10:36:13+09:00
draft: false
categories: [IoT]
tags: [influxDB, docker, INKBIRD, ITH-12S, IBS-TH2]
thumbnailImagePosition: left
thumbnailImage: /images/influxdb/chronograf.png
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

{{< highlight docker "linenos=ture" >}}

version: '3'
services:
  influxdb:
    container_name: influxdb_1810
    image: influxdb:1.8.10
    ports:
      - "8086:8086"
    volumes:
      - influxdb:/var/lib/influxdb

  chronograf:
    container_name: chronograf_1810
    image: chronograf:1.8.10
    ports:
      - "8888:8888"
    links:
      - influxdb
    volumes:
      - chronograf:/var/lib/chronograf

volumes:
  influxdb:
  chronograf:

{{< /highlight >}}

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
    {{< figure src="/images/influxdb/useradd.png" caption="ユーザ登録後の画面" height="400" >}}

## 温度湿度データ登録用データベースの作成

次に、InfluxDB上に温度湿度データを登録するデータベースとそのデーターベース用のユーザを作成します。
- データベース名:hogehoge（今回例として設定したもの）

実際の作成はpython スクリプトcreatedb.pyで実施します。  
createbd.py内の`INFLUXDB_`で始まる環境変数に先ほど設定したROOTユーザ名・パスワード、データベース名、データベースユーザ名・パスワードを記載します。
また、ホスト名、ポート番号はdocker-compose.ymlでの設定を考慮し、`localhost`、`8086`になります。  
なお、温度湿度データをデータベース内に保持する期間を2か月`60d`としています。

{{< highlight python "linenos=ture" >}}
import os
from influxdb import InfluxDBClient

def main():
    host = os.environ.get('INFLUXDB_HOST', 'localhost')
    port = os.environ.get('INFLUXDB_PORT', 8086)
    user = os.environ.get('INFLUXDB_USER', 'testuser1')
    password = os.environ.get('INFLUXDB_PASSWORD', 'ptestuser1')
    dbuser = os.environ.get('INFLUXDB_DBUSER', 'testuser2')
    dbuser_password = os.environ.get('INFLUXDB_DBUSER_PASS', 'ptestuser2')
    dbname = os.environ.get('INFLUXDB_DBNAME', 'hogehoge')
    retension = '60d'

    client = InfluxDBClient(host, port, user, password, dbname)

    print(f"Create database: {dbname}")
    client.create_database(dbname)

    print(f"Create a retention policy: {retension}")
    client.create_retention_policy(
        'raw_data_policy', retension, 1, default=True)

    print(f"Create a db user: {dbuser}")
    client.create_user(dbuser, dbuser_password)

if __name__ == '__main__':
    main()
{{< /highlight >}}

それではcreatebd.pyを実行し、InfluxDBに温度湿度登録用データベースを作成します。
```
(venv38) PS D:\projects\ble> python createdb.py
<省略>
```
実行後に、cronografのWebから、下記のようにデータベースhogehogeが作成されており、ユーザにtestuser2が追加されているはずです。
{{< figure src="/images/influxdb/createdb.png" caption="温度湿度用データベース作成後の画面" height="400" >}}
{{< figure src="/images/influxdb/after_created.png" caption="温度湿度用データベース作成後のユーザ一覧" height="400" >}}

# データの登録

## 定期実行用pythonスクリプトproc.pyの作成

以前に公開したBluetooth温度湿度計からデータを取得するpythonスクリプトをベースにinfluxdbに登録するコード（writedb()）を追加します。
なお、BluetoothLEで温度湿度データが取得できないときがあるため、取得できない場合は3回まではリトライするようにbre_read()にretryを指定しています。

{{< highlight python  "linenos=ture" >}}
import asyncio
from bleak import BleakScanner, BleakClient
from bleak.exc import BleakError
import struct
from retry import retry
import os
from datetime import datetime, timezone, timedelta
from influxdb import InfluxDBClient

SPS_12S = "49:22:03:25:03:ED"
SPS_TH2 = "49:22:08:15:10:2B"
REALTIME_DATA_UUID = "0000fff2-0000-1000-8000-00805f9b34fb"
NUM_OF_TRY = 3

async def connect_and_read(sensor):
    mac = sensor['mac']
    uuid = sensor['uuid']
    device = await BleakScanner.find_device_by_address(mac, timeout=10.0)
    if not device:
        raise BleakError(f"Device {mac} not found.")
    print(f"Device {mac} is found.")
    async with BleakClient(device) as client:
        if not client.is_connected:
            raise BleakError(f"Not connect device {mac}.")
        print("Connected! Now data reading...")
        datas = await client.read_gatt_char(uuid)
        recordtime = datetime.now(timezone(timedelta(hours=9)))
        return datas, recordtime

def get_data(raw):
    (a, b, c, d, e) = struct.unpack('<HHBBB', raw)
    temp = a/100
    humid = b/100
    unknown = ( c, d, e )
    return temp, humid

@retry(BleakError, tries=3, delay=1)
def ble_read( sensor ):
    loop = asyncio.get_event_loop()
    results, rectime = loop.run_until_complete(connect_and_read(sensor))
    return results, rectime

def writedb( temp, humid, mac ):
    # influxdb
    host = os.environ.get('INFLUXDB_HOST', 'localhost')
    port = os.environ.get('INFLUXDB_PORT', 8086)
    dbuser = os.environ.get('INFLUXDB_DBUSER', 'testuser2')
    dbuser_password = os.environ.get('INFLUXDB_DBUSER_PASS', 'testuser2')
    dbname = os.environ.get('INFLUXDB_DBNAME', 'hogehoge')
    client = InfluxDBClient(host, port, dbuser, dbuser_password, dbname)
    try:
    # データ
        data = [
            {
                "measurement": dbname,
                "tags": {
                    "mac": mac
                    },
                    "time": datetime.now(timezone(timedelta(hours=9))).isoformat(),
                    "fields": {
                        "temperature": float(temp),
                        "humidity": float(humid),
                    }
            }
        ]
        print(data)
    # 書き込み
        client.write_points(data)
    except Exception as e:
        print(f"Error: can't write data {mac} - {e}")

def main( sensors ):
    for sensor in sensors:
        #print(sensor)
        try:
            results, rectime = ble_read(sensor)
        except Exception as e:
            print(f"Error: can't read sensor {sensor['mac']} - {e}")            
        else:
            temp, humid = get_data(results)
            print(f"temp:{temp},humid:{humid}, time:{rectime} ")
            writedb(temp, humid, sensor['mac'])

if __name__ == '__main__':
    sensors = [
        {
            'uuid': REALTIME_DATA_UUID,
            'mac': SPS_12S
        },
        {
            'uuid': REALTIME_DATA_UUID,
            'mac': SPS_TH2
        }
    ]
    main( sensors )
{{< /highlight >}}

## タスクスケジューラから定期実行されるtask.batの作成

先のpythonスクリプトをWindowsのタスクスケジューラから呼び出すために、タスクスケジューラに登録するBATファイル（task.bat）を作成します。
なお、pythonの仮想環境venv38上で実行するようにtask.batを配置しています。

{{< highlight bat >}}
d:
cd %~dp0
call .\venv38\Scripts\activate
python proc.py
deactivate
{{< /highlight >}}

このbatファイルをタスクスケジューラで5分間隔で実行するように設定しました。
{{< figure src="/images/influxdb/task_scheduler.png" caption="タスクスケジューラへの登録" height="400" >}}

# 登録データのグラフ化

chronografを使い、2つの温度湿度計データをグラフ化したものを下記に掲載します。
{{< figure src="/images/influxdb/disp_graph.png" caption="温度湿度データのグラフ化" height="400" >}}

# 最後に

今回はinfluxdbとchronografにPythonスクリプトを用いて、温度湿度計の時系列データを登録し、グラフ表示できるようにしみました。
温度湿度を監視するにはグラフ化以外にも、設定した閾値を超えたらアラートのメールを出すように機能もほしくなります。いつか試してみようと思います。
