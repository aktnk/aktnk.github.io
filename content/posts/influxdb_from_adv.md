---
title: "Inkbird BLE通信のアドバタイズデータで取得失敗を無くす"
date: 2023-01-06T00:59:25+09:00
draft: false
categories: [IoT, server]
tags: [influxDB, bleak, advertise, INKBIRD, ITH-12S, IBS-TH2]
thumbnailImagePosition: left
thumbnailImage: /images/influxdb_from_adv/missed_data_detail.png
---

{{< toc >}}

# はじめに

以前に公開したPythonスクリプトではInkbird温度湿度計とのGATT通信で温度・湿度データを取得していましたが、ときどき取得できないケースが発生していました。
{{< figure src="/images/influxdb_from_adv/missed_data.png" link="/images/influxdb_from_adv/missed_data.png" title="データの取りこぼし" >}}
{{< figure src="/images/influxdb_from_adv/missed_data_detail.png" link="/images/influxdb_from_adv/missed_data_detail.png" title="データの取りこぼし例" >}}
アドバタイズデータからデータを取得するこで、温度湿度データの取りこぼしが発生しないようにすることができないか試してみました。

# 調査&試行

## 元の実装で実施していたこと

取りこぼしを防ぐ狙いで下記を実施していました。
- InkbirdのBLEデバイスが見つからないとき、見つかって接続できないとき。GATT通信が体うアウトしたときは全部で3回までやり直しを行う

{{< gist aktnk c635eb7d81f93222468d61c2eee61def "proc.py" >}}

## 今回試したこと

- GATT通信せずにアドバタイズデータから温度・湿度データを取得する
- BLEデバイスが見つからないときは3回までやり直しを行うが、試行回数が増えるにしたがって試行までの間隔を増やしながら試行を行う
- 最終的に温度湿度計のデータがInfluxDBに書き込めなかった場合は、アドバタイズデータの取得からやり直す。その際、試行回数が増えるにしたがってアドバタイズデータ取得までの時間を増やしながら取得を行う
- どこで失敗したかわかるようにloggerを利用でログを残す
- PCのBLEをOFFにしたり、InkbirdのBLE通信をOFFにしたり、Inkbirdを別の部屋に以て行ったりしながら、テストを行う

最終的に実験を行った実装を下記に掲載します。

{{< gist aktnk 2b4ab98ceabd769a58d1c5a5a44f4e40 "ble_adv_to_influxdb.py" >}}

### 試行条件

- Inkbirdの2つの温度湿度計（IBS-TH2、ITH-12S）を使用
- 1分間隔でアドバタイズデータをWindows10 PCで取得し、InfluxDBに記録
- PCと温度湿度計との距離は2m
- 温度湿度計のバッテリは12/28に新品に交換し1週間経過
- 24時間連続して計測し、取得できなかった回数を調べる

### 試行結果

残念ながら、各デバイスで各1回/24Hデータの取りこぼしが発生し、24時間データの取りこぼし無しは達成できませんでした。


- ITH-12SでAM5:12に取りこぼしが発生した際のログ
  ```
  2023-01-06 05:12:01 [DEBUG] test7.py, lines 71. main called : 1
  2023-01-06 05:12:01 [DEBUG] test7.py, lines 63. read_BLEdevice() called
  2023-01-06 05:12:06 [WARNING] test7.py, lines 96. not found device 49:22:03:25:03:ED
  2023-01-06 05:12:07 [WARNING] test7.py, lines 99. retry 2 device(49:22:03:25:03:ED)
  2023-01-06 05:12:07 [DEBUG] test7.py, lines 71. main called : 2
  2023-01-06 05:12:07 [DEBUG] test7.py, lines 63. read_BLEdevice() called
  2023-01-06 05:12:12 [DEBUG] test7.py, lines 84. found device (49:22:08:15:10:2B)
  2023-01-06 05:12:12 [INFO] test7.py, lines 89. rssi:-63, temp:20.3, humid:56.8, now:2023-01-06T05:12:12.901596+09:00
  2023-01-06 05:12:12 [INFO] test7.py, lines 44. wrote data: [{'measurement': 'hogehoge', 'tags': {'mac': '49:22:08:15:10:2B'}, 'time': '2023-01-06T05:12:12.901596+09:00', 'fields': {'temperature': 20.3, 'humidity': 56.8}}]
  2023-01-06 05:12:12 [WARNING] test7.py, lines 96. not found device 49:22:03:25:03:ED
  2023-01-06 05:12:14 [WARNING] test7.py, lines 99. retry 3 device(49:22:03:25:03:ED)
  2023-01-06 05:12:14 [DEBUG] test7.py, lines 71. main called : 3
  2023-01-06 05:12:14 [DEBUG] test7.py, lines 63. read_BLEdevice() called
  2023-01-06 05:12:19 [WARNING] test7.py, lines 96. not found device 49:22:03:25:03:ED
  2023-01-06 05:12:22 [WARNING] test7.py, lines 99. retry 4 device(49:22:03:25:03:ED)
  2023-01-06 05:12:22 [DEBUG] test7.py, lines 71. main called : 4
  2023-01-06 05:12:22 [DEBUG] test7.py, lines 63. read_BLEdevice() called
  2023-01-06 05:12:27 [WARNING] test7.py, lines 96. not found device 49:22:03:25:03:ED
  2023-01-06 05:12:30 [WARNING] test7.py, lines 99. retry 5 device(49:22:03:25:03:ED)
  2023-01-06 05:12:30 [DEBUG] test7.py, lines 71. main called : 5
  2023-01-06 05:12:30 [DEBUG] test7.py, lines 63. read_BLEdevice() called
  2023-01-06 05:12:35 [WARNING] test7.py, lines 96. not found device 49:22:03:25:03:ED
  2023-01-06 05:12:35 [ERROR] test7.py, lines 102. retry failed device(49:22:03:25:03:ED)
  ```
  5回再取得を試みましたが、取得できずERRORとなっています。
  {{< figure src="/images/influxdb_from_adv/result_ITH_12S.png" link="/images/influxdb_from_adv/result_ITH_12S.png" target="_blank" title="ITH-12Sでの取りこぼし" >}}
  {{< figure src="/images/influxdb_from_adv/result_IBS_TH2.png" link="/images/influxdb_from_adv/result_IBS_TH2.png" title="IBS-TH2での取りこぼし" >}}

# 最後に

データの取りこぼしが発生することが許されない環境で計測を実施する場合、無線でなく有線を利用した方がよいのかもしれません。
また、BLE通信するデバイスとPCとの距離が変わったり、デバイスのバッテリ残量が減少すると取りこぼしの確率は増えるかもしれません。  
これらを踏まえると、実際に使用する環境で事前に試行を行い、対策する必要がありそうです。現地現物は大切だと感じた実験でした。