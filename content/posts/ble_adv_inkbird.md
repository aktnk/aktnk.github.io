---
title: "InkbirdのBluetooth温度湿度計のアドバタイズデータから温度・湿度データを読み取る"
date: 2023-01-05T22:56:05+09:00
draft: false
categories: [IoT, Python]
tags: [bleak, INKBIRD, ITH-12S, IBS-TH2, Bluetooth LE, advertise]
thumbnailImagePosition: left
thumbnailImage: /images/ble_inkbird/ITH12S.jpg
---

{{< toc >}}

# はじめに

以前、[Windows10からINKBIRD温度湿度計データをBLE通信で取得する](/2022/12/29/ble_inkbird/)ではBLEのGATT通信を使い、Inkbirdの温度湿度計からリアルタイムデータを取得しました。  
その後、Inkbirdの温度湿度計がBLEでアドバタイズしているデータから温度湿度データを取得することができましたので、その実装例を公開します。

# 使用した機器

[Windows10からINKBIRD温度湿度計データをBLE通信で取得する](/2022/12/29/ble_inkbird/#使用した機器)と同じです。

## InkbirdのBluetooth温度湿度計

- INKBIRD の Bluetooth 温度湿度計（2 機種）
  - IBS-TH2 [^1] + 単 4 電池 2 本
    {{< figure src="/images/ble_inkbird/IBSTH2.jpg" link="/images/ble_inkbird/IBSTH2.jpg" title="IBS-TH2" >}}
    [^1]: 購入先 Amazon https://www.amazon.co.jp/gp/product/B08WWPSTRS
  - ITH-12S（Bluetooth 温度湿度計）[^2]
    {{< figure src="/images/ble_inkbird/ITH12S.jpg" link="/images/ble_inkbird/ITH12S.jpg" title="ITH-12S" >}}
    [^2]: 購入先 Amazon https://www.amazon.co.jp/gp/product/B09PDMH3FQ

## PC

- Blutooth が使用可能な PC  
  事前に[Windows スタート]>[設定]>[デバイス]から Bluetooth を有効にしておきましょう。
  {{< figure src="/images/ble_inkbird/win10_ble_settings.png" link="/images/ble_inkbird/win10_ble_settings.png" title="Windows10 Bluetoothの有効化" >}}
  なお、今回使用した PC は
  - ASUS UX310U NotebookPC  
    CPU: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz  
    OS: WIndows 10 Pro 22H2  
  
  です。

## スマホ

- Android もしくは iPhone + nRF Connect for Mobile アプリ  
  事前に Nordic Semiconductor 社の[nRF Connect for Mobile https://www.nordicsemi.com/Products/Development-tools/nrf-connect-for-mobile](https://www.nordicsemi.com/Products/Development-tools/nrf-connect-for-mobile)をインストールしておきましょう。

# 準備

## Inkbird温度湿度計のアドバタイズデータを確認する

スマホにインストールしたnRf Connectアプリを使い、Inkbird温度湿度計がBLE通信でアドバタイズしているデータを確認します。
1. アプリを立ち上げSCANNERタブでデバイスをスキャンします。そして、spsと表示されたデバイスをタップします。
  {{< figure src="/images/ble_adv_inkbird/nRF_connect_scan.png" link="/images/ble_adv_inkbird/nRF_connect_scan.png" title="nRF Connectアプリでデバイスをスキャンする" >}}
1. すると詳細が表示されるので、[RAW]をクリックします。
  {{< figure src="/images/ble_adv_inkbird/Inkbird_BLE.png" link="/images/ble_adv_inkbird/Inkbird_BLE.png" title="Inkbirdデバイスspsを表示する" >}}
1. Raw data欄に表示されたデータ`0x0201060302F0FF04097370730AFF8208D71800E446E08`がアドバタイズデータです。このデータの詳細がDetailsに表示されています。この中のLEN:10の行に表示されているTYPE:`0xFF`,VALUE:`0x8208D71800E4465E08`がmanufacturerデータです。
  {{< figure src="/images/ble_adv_inkbird/ITH12S_advertisedata.png" link="/images/ble_adv_inkbird/ITH12S_advertisedata.png" title="ITH-12Sのアドバタイズデータ" >}}

# Pythonスクリプト作成

## bleakパッケージのAPI調査

Python Bleakパッケージを利用してmanufacturerデータを取得します。
- BleakScanner.discover()の引数return_advをTrueにして呼びだすことで、アドバタイズデータを取得できます。
- 今回使用しているbleakパッケージの[stable版（v.0.19.5)のドキュメント](https://bleak.readthedocs.io/en/stable/api/scanner.html#easy-methods)を確認すると、discover(return_adv=Ture)時の戻り値は検出したBLEDeviceをキー、AdvertisementDataをバリューとする辞書型データです。
- そして[同ドキュメントのAdvertisementData](https://bleak.readthedocs.io/en/stable/backends/index.html#bleak.backends.scanner.AdvertisementData.manufacturer_data)を調べるとmanufacturer_dataが[int,bytes]の辞書型データになるようです。

## 作成したPythonスクリプト

以上から、実際にスクリプトを作成し試した結果、下記のような実装をすることで温度・湿度のデータを取得することができました。

{{< gist aktnk 882f3b412acda9f3584ada19bc07170d "inkbird_ble_adv.py" >}}

## 実行結果

```
(venv38) PS D:\projects\ble> python .\ble_adv_inkbird.py
49:22:08:15:10:2B
rssi:-57, temp:21.85, humid:55.45, now:2023-01-06 00:23:16.288976
-----------------
49:22:03:25:03:ED
rssi:-65, temp:21.59, humid:58.76, now:2023-01-06 00:23:16.288976
-----------------
(venv38) PS D:\projects\ble>
```

# 最後に

今回BLE通信のアドバタイズデータをスマホnRF Connectのように取得できれば、そのバイトデータを処理すればよいと思い調査しました。しかし、bleakパッケージでは見つけることができませんでした。そこで、イマイチですが今回のような形の実装になりました。  
もしかしたら、Inkbirdのアドバタイズデータが標準に基づいていない部分があるのでしょうか？
BLE通信についての知識と経験が浅いので、今後も精進しようと思います。
