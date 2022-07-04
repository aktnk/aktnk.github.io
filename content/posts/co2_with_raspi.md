---
title: "Raspberry Pi で二酸化炭素濃度を測定する"
date: 2021-06-19T16:47:39+09:00
draft: false
categories: [IoT]
tags: [MH-Z19B, Python, Raspberry Pi]
thumbnailImagePosition: left
thumbnailImage: /images/co2_with_raspi/hard1.jpg
---

{{< toc >}}

# はじめに

NDIR( nondispersive infrared , 非分散型赤外線 )型センサーである Winsen の MH-Z19B を Raspberry Pi 3B+ に接続し、部屋の二酸化炭素濃度を測定してみます。

## 最終ゴール

最初に本記事の最終ゴールを掲載します。

### ハードウェア構成

今回実現するシステムのハードウェア構成を下記に示します。

- 本体: Raspberry Pi 3B+
- ケース: Smraza Raspberry Pi 3B+
- CO2 センサー: MH-Z19B+Terminal
- 電源: Cheero Canvas 3200mAh CHE-061

{{<figure src="/images/co2_with_raspi/hard1.jpg" caption="システムの全体像">}}

### 二酸化炭素濃度の測定

Rspberry Pi 上の Python に mh-z19 モジュールをインストール後、mh_z19 を実行すると MH-Z19B センサーから二酸化炭素濃度を取得し表示することができます。

{{<figure src="/images/co2_with_raspi/measured.png" caption="二酸化炭素濃度測定結果の表示">}}

# 準備

## 二酸化炭素濃度センサーの選定

[Wisen の Web サイトで CO2 Gas Sensor の製品](https://www.winsen-sensor.com/sensors/co2-sensor/)を確認すると、

- NDIR 型 CO2 センサーとして
  - MH-Z14,MH-Z14A,MH-Z14B
  - MH-Z16
  - MH-Z19B,MH-Z19C
  - MH-410D
  - MH711A
- NDIR 以外の方式のセンサーとして
  - MD62
  - MG812
  - ZPHS01B（CO2 以外のセンサーも搭載）
  - その他

といろいろあります。  
今回、Rapsberry Pi の GPIO 端子に接続して使用するため、上記の CO2 センサーの中で UART に対応しているものを選びます（Wisen の CO2 Gas Sensor 製品の Web ページの各センサーの下にある[view details]のリンクから詳細仕様を確認し、Output Signal 欄に UART が記載されていれば OK で、多くのセンサーが UART に対応していました）。その中で今回は[MH-Z19B](https://www.winsen-sensor.com/sensors/co2-sensor/mh-z19b.html)を選定しました。

## MH-Z19B センサーの購入

Google で"MH-Z19B センサー"を検索すると…  
Amazon のサイトが出てきますが、Amazon のサイトで購入すると 4500 円以上になります。
更に見ていくと AliExpress のサイトが出てきます。こちらのサイトだと$18 程度（送料別）で購入できそうです。
そこで、AliExpress のサイトで肯定的フィードバックが 98.7%の[”Your Cee”というショップ](https://ja.aliexpress.com/item/4000261110777.html)で MH-Z19B を購入することにしました。
このショップでは、MH-Z19B を

1. センサー単体
2. センサー+Pin
3. センサー+Terminal

の 3 種類のものが購入できます。
{{<figure src="/images/co2_with_raspi/yourcee_mh_z19b.png" caption="Your Ceeショップで購入可能なMH-Z19B">}}
今回、ケーブルが付いている”MH-Z19B ＋ Terminal”を選択しました（ブレッドボードを用いた試作の場合は MH-Z19B+Pin を購入すると、自分で Pin ヘッダーをはんだ付けする手間が省けます）。

#### 購入費用と納期

MH-Z19B センサー購入の費用は

- MH-Z19B+Terminal 　 2145 円
- 送料　 476 円
- 合計　 2621 円

納期は実績として 2 週間（2021/5/15 に注文し、5/29 に到着しました）でした。

## MZ-H19B 接続ケーブルの改造

Raspberry Pi の GPIO ピンは 2.54㎜ ピッチで並んでいます。
{{<figure src="/images/co2_with_raspi/raspberrypi_pin.jpg" caption="Raspberry Pi B+のGPIOピン">}}
なお、GPIO ピンの配置は[こちら](https://github.com/raspberrypi/documentation/blob/develop/documentation/asciidoc/computers/os/using-gpio.adoc)を参照して下さい。
それに対し、下記 MH-Z19B のケーブルを接続するには
{{<figure src="/images/co2_with_raspi/mz_h19b_with_terminal.jpg" caption="MH-Z19B+Terminal付属ケーブル">}}

1. ２つある白いコネクタの一方から、Raspberry Pi に接続する 4 本のケーブル（赤-5V Power 用、黒-GND 用、青-GPIO14(UART TX)用、緑-GPIO15(UART RX)用）を外します。
2. 下記パーツを使い QI コネクタに変更します。
   - QI コネクタハウジング
   - ケーブル用コネクタ

{{<figure src="/images/co2_with_raspi/qi_connecter.jpg" caption="MH-Z19B付属の接続ケーブル4本（赤、黒、緑、青）をQIコネクタに変更する">}}

### (補足)Terminal のピン配置

MH-Z19B の Terminal のピン配置の情報は[https://www.winsen-sensor.com/d/files/MH-Z19B.pdf](https://www.winsen-sensor.com/d/files/MH-Z19B.pdf)から入手可能で、該当箇所のみ抜き出すと下記のようになっています。
{{<figure src="/images/co2_with_raspi/terminal_pins.png" caption="MH-Z19BのTerminalのピン配置">}}
そのため、今回購入した MH-Z19B ＋ Terminal の接続ケーブルでは、Pin3-黒、Pin4-赤、Pin5-青、Pin6-緑ということになります。

### 改造後の MH-Z19B と接続ケーブル

最終的に下記のように改造しました。
{{<figure src="/images/co2_with_raspi/mz_h19b_with_qi_connecter.jpg" caption="改造後の接続ケーブルとMH-Z19Bセンサー">}}

## Raspberry Pi 3B+と MH-Z19B の接続

MH-Z19B 側 - Raspberry Pi 3B+側との接続を下記のように行います。

- 赤 - 5V Power 用
- 黒 - Ground
- 青 - GPIO14(UART TX)
- 緑 - GPIO15(UART RX)

{{<figure src="/images/co2_with_raspi/rpi_w_mhz19b.jpg" caption="MH-Z19BとRaspberryPiの接続">}}
{{<figure src="/images/co2_with_raspi/rpi_gpio_connecter.jpg" caption="GPIO部の拡大">}}

## Raspberry Pi 3B+の UART 有効化

Raspberry Pi のドキュメントにあるように、3B+ではデフォルトで UART0 は/dev/serial1 に割り当てられており、/dev/ttyAMA0 の Bluetooth 接続されています。そして、UART1 は/dev/serial0 で mini UART に割り当てられていますが、mini UART はデフォルトでは有効化されていません。  
そこで、

1. エディタで/boot/config.txt を開き、enable_uart=1 を追加します。
   {{<figure src="/images/co2_with_raspi/tail_boot_config.png" caption="/boot/config.txtにenable_uart=1を追加する">}}
2. Raspberry Pi を再起動します。
   {{<figure src="/images/co2_with_raspi/reboot.png" caption="shutdownコマンドで再起動する">}}
3. 再起動後、/dev/serial*を確認し、下記のようになっていれば OK！
   {{<figure src="/images/co2_with_raspi/serial_check.png" caption="再起動後に/dev/serial*を確認する">}}

## Python モジュール mh-z19 のインストール

Python で MH-Z19B を扱うため[PyPI(Python Package Index)](https://pypi.org)の[mh-z19 モジュール](https://pypi.org/project/mh-z19/)より下記のようにインストールします。なお、mh-z19 モジュールのソースコードは[GitHub 上の@UedaTakayuki 氏のリポジトリ](https://github.com/UedaTakeyuki/mh-z19)に MIT ライセンスで公開されています。

```
$ sudo pip install mh-z19
```

{{<figure src="/images/co2_with_raspi/1_install.png" caption="mh_z19モジュールのインストール">}}

# 二酸化炭素濃度の測定

動作確認も兼ねて、下記のように MH-Z19B センサーを読み取ります。

```
$ sudo python -m mh_z19
{"co2": (二酸化炭素濃度ppm)}
```

{{<figure src="/images/co2_with_raspi/2_measure.png" caption="二酸化炭素濃度の測定">}}
問題なく二酸化炭素濃度を測定できているようです。
MH-Z19B は二酸化炭素濃度以外にも気圧(UhUl)や温度(temperature)も計測できるようです。
{{<figure src="/images/co2_with_raspi/hard1.jpg" caption="今回作成したもの">}}

# 最後に

センサーから二酸化炭素濃度を取得することができたので、今後は、二酸化炭素濃度がある濃度以上になったら警告したり、二酸化炭素濃度の推移をグラフ表示したりできると良いと思いました。
