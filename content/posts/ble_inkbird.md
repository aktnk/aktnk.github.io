---
title: "BLE_inkbird"
date: 2022-12-29T20:17:55+09:00
draft: false
categories: [IoT]
tags: [INKBIRD, ITH-12S, IBS-TH2, Bluetooth LE]
thumbnailImagePosition: left
thumbnailImage: /images/ble_inkbird/cover.png
---

{{< toc >}}

# はじめに

今回は Bluetooth 機能がついた温度・湿度計を Bluetooth 経由で PC と接続し、温度・湿度データを取得します。

これまで Bluetooth と言えば、iPhone を車載オーディオに接続し音楽を聞いたり、ワイヤレスヘッドフォンをノート PC と接続したり・・・、音楽用に使うイメージしかありませんでした。
いつものように Amazon のサイトを見ていると、INKBIRD という会社から Bluetooth 通信機能がついた温度・湿度計が発売されていることに気付きました。

そこで、Windows10 のノート PC の Bluetooth を使い、INKBIRD 温度・湿度計から温度・湿度データを取得してみます。

# 使用した機器

## 購入したもの

- INKBIRD の Bluetooth 温度湿度計（2 機種）
  {{<figure src="/images/ble_inkbird/inkbird_sensors.png" caption="INKBIRDのBluetooth温度湿度計" height="350">}}
  - IBS-TH2 [^1] + 単 4 電池 2 本
    {{<figure src="/images/ble_inkbird/IBSTH2.jpg" caption="IBS-TH2" height="300">}}
    [^1]: 購入先 Amazon https://www.amazon.co.jp/gp/product/B08WWPSTRS
  - ITH-12S（Bluetooth 温度湿度計）[^2]
    {{<figure src="/images/ble_inkbird/ITH12S.jpg" caption="ITH-12S" height="150">}}
    [^2]: 購入先 Amazon https://www.amazon.co.jp/gp/product/B09PDMH3FQ

## PC

- Blutooth が使用可能な PC  
  事前に[Windows スタート]>[設定]>[デバイス]から Bliutooth を有効にしておきましょう。
  {{<figure src="/images/ble_inkbird/win10_ble_settings.png" caption="Windows10 Bluetoothの有効化" height="400">}}
  なお、今回使用した PC は
  - ASUS UX310U NotebookPC  
    CPU: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz  
    OS: WIndows 10 Pro 22H2  
    です。

## スマホ

- Android もしくは iPhone + nRF Connect for Mobile アプリ  
  事前に Nordic Semiconductor 社の[nRF Connect for Mobile https://www.nordicsemi.com/Products/Development-tools/nrf-connect-for-mobile](https://www.nordicsemi.com/Products/Development-tools/nrf-connect-for-mobile)をインストールしておきましょう。

# 準備

## Python 仮想環境の構築

Windows10 上にインストールした python3.8 を使い、仮想環境を作成し、bleak パッケージ [https://pypi.org/project/bleak/](https://pypi.org/project/bleak/) をインストールします。なお、私がインストールした bleak のバージョンは 0.19.5 でした。

```
PS D:\projects\ble> python -m venv venv38

PS D:\projects\ble> .\venv38\Script\activate

(vnev28) PS D:\projects\ble> pip install bleak
省略
(venv38) PS D:\projects\ble> pip show bleak
Name: bleak
Version: 0.19.5
Summary: Bluetooth Low Energy platform Agnostic Klient
Home-page: https://github.com/hbldh/bleak
Author: Henrik Blidh
Author-email: henrik.blidh@nedomkull.com
License: MIT
Location: d:\projects\ble\venv38\lib\site-packages
Requires: async-timeout, bleak-winrt
Required-by:
(venv38) PS D:\projects\ble>

```

## INKBIRD 温度湿度計を稼働させる

Amazon から INKBIRD の温度湿度計が届いたら、電池を入れて稼働させます。

- ITH-12S はボタン電池 CR2032 が付属していますので、上下（＋－）を間違えないように取り付けます。
  {{<figure src="/images/ble_inkbird/ITH12S_battery.jpg" caption="ITH-12Sバッテリ取付" height="150">}}
  なお、電池を取り付けたらパネル右上の Bluetooth 通信が ON になっていることを確認します。もし ON になっていない場合は、Bluetooth のボタンを何度か押して ON にします。
  {{<figure src="/images/ble_inkbird/ITH12S_ble.jpg" caption="ITH-12S Bleutooth通信ON" height="250">}}
- IBS-TH2 は単 4 電池が 2 本必要です。付属のドライバーを使い、IBS-TH2 の裏蓋を開け単 4 電池を装着します。
  {{<figure src="/images/ble_inkbird/IBSTH2_battery.jpg" caption="IBS-TH2バッテリ取付" height="300">}}

## INKBIRD 温湿度計の Bluetooth 通信内容調査

### MAC アドレス調査

Android または iPhone にインストールした nRF Connect を起動し、バッテリを入れた温度湿度計の MAC アドレスを確認します。下図の sps と表示されている欄に記載されている"49:22:\*\*:\*\*:\*\*:\*\*"の 16 進の数値が MAC アドレスです。このように 2 つ同時に稼働させると 2 つのどちら INKBIRD の MAC アドレスかわかりませんので、ITH-12S は Bluetooth 通信をボタンで OFF にし、MAC アドレスを確認します。
{{<figure src="/images/ble_inkbird/mac_address.png" caption="nRF ConnectでMACアドレス確認" height="600">}}

### データ取得用 UUID 調査

- UUID 共通部分の調査

  - nRF Connect に表示された sps をクリックし、情報を表示させます。そして、[MORE]のリンクをクリックします。
    {{<figure src="/images/ble_inkbird/more.png" caption="nRF ConnectでUUIDの調査(1)" height="600">}}
  - 表示された画面にて[FLAGS & SERVICES]タブをクリックし、サービスクラス UUID を確認します。ここには 3 つ UUID が表示されており、下記全てが共通していることがわかります。  
    `0000fff0-0000-1000-8000-00805f9b34fb`
    {{<figure src="/images/ble_inkbird/uuid.png" caption="nRF ConnectでUUIDの調査(2)" height="600">}}

- 温度・湿度データ取得用コードの調査
  - 元も表示に戻り、今度は[CONNECT]ボタンを押します。
    {{<figure src="/images/ble_inkbird/connect.png" caption="nRF Connectでデータ取得コードの調査(1)" height="600">}}
  - 表示された画面にて[Unknown Service]をクリックします。
    {{<figure src="/images/ble_inkbird/service.png" caption="nRF Connectでデータ取得コードの調査(2)" height="600">}}
  - 表示された画面にて上から順に、ダウンロードボタンをクリックし、Unknouwn Characteristic の内容、Characterristic User Descrption の内容を取得し、表示させます。すると、Realtime data 取得コードが UUID 0xFFF2 だと推察できます。
    {{<figure src="/images/ble_inkbird/realtimedata.png" caption="nRF Connectでデータ取得コードの調査(3)" height="600">}}

## 温度湿度データ取得のトライ

[GitHub の bleak リポジトリ https://github.com/hbldh/bleak](https://github.com/hbldh/bleak)に[実装例 https://github.com/hbldh/bleak/tree/develop/examples](https://github.com/hbldh/bleak/tree/develop/examples)が公開されています。
これらを参考にしながら、前節で調査した MAC アドレスおよび UUID 共通部分の先頭4バイトをデータ取得用 UUID に置き換えて、温度・湿度データの取得できるかトライします。

### トライ用に作成した Python スクリプト

{{< highlight PYTHON "linenos=ture" >}}
import asyncio
from bleak import BleakClient
import struct

MACS = ["49:22:03:25:03:ED","49:22:08:15:10:2B"]
REALTIME_DATA_UUID = "0000fff2-0000-1000-8000-00805f9b34fb"

async def run(address, loop):
    async with BleakClient(address, loop=loop) as client:
    x = await client.is_connected()
    print("Connected: {0}".format(x))
    y = await client.read_gatt_char(REALTIME_DATA_UUID)
    print(y)
    (temp, humid, unknown)=real_time_data(y)
    print(temp)
    print(humid)
    print(f"{unknown}")

def real_time_data(y):
    (a, b, c, d, e) = struct.unpack('<HHBBB', y)
    temp = a/100
    humid = b/100
    unknown = ( c, d, e )
    return temp, humid, unknown

if __name___=="__main__":
    loop = asyncio.get_event_loop()
    for address in MACS:
        loop.run_until_complete(run(address, loop))
{{< /highlight >}}

### 実行結果

実行すると、下記のように2つのINKBIRDから温度・湿度が取得できました。
```
(venv38) PS D:\projects\ble> python .\test1.py
.\test1.py:10: FutureWarning: is_connected has been changed to a property. Calling it as an async method will be removed in a future version
x = await client.is_connected()
Connected: True
bytearray(b'\x0f\t\xb5\x15\x00l\xeb')
23.19
55.57
(0, 108, 235)
Connected: True
bytearray(b'\x84\x08j\x17\x00\xf9\x92')
21.8
59.94
(0, 249, 146)
(venv38) PS D:\projects\ble>
```

# 最後に

Google で inkbird pythonと検索すると、INS-TH1の温度湿度データをLinuxのbluepyを使って取得記事がいくつか出てきます。
- https://qiita.com/c60evaporator/items/e7c3b2c65ad1664c832f
- https://qiita.com/junara/items/f396c1c4c15c78cde89f
- https://zenn.dev/yh1224/articles/ld2arlkd2v6azz4xg

今回はこれらを参考にWindows10でbluepyを試しましたが、そもそもbluepyがインストールすることができませんでした。

そんな中で見つけたのが、下記の記事でした。
- https://atatat.hatenablog.com/entry/2020/07/09/003000

ここではWindows10でbleakを使い、BLE通信でデータを取得する方法が記載されていました。今回のBLE通信の部分は全く同じものです。

その際に苦労したのは、実測データを取得するためのUUIDを調べることでした。
この記事にはあっさりと書いていますが、実際には上から順に全て試しながら調べた結果です。

しかしながら、この記事を書きながら再度Google検索をしていたところ、なっなんとINKBIRDのアドバタイズパケットに温度・湿度データの他にもバッテリーレベルを取得している実装[^3]がGitHub上の[CUSTOM_COMPONENTS/BLE Monitorのリポジトリhttps://github.com/custom-components/ble_monitor](https://github.com/custom-components/ble_monitor)にて公開されていることがわかりました。

ここまで書いてから見つけてしまったので、この記事はそのまま公開します。

[^3]: アドバタイズパケットから温度湿度等を取得する実装は下記公開されています。[https://github.com/custom-components/ble_monitor/blob/master/custom_components/ble_monitor/ble_parser/inkbird.py](https://github.com/custom-components/ble_monitor/blob/master/custom_components/ble_monitor/ble_parser/inkbird.py)

