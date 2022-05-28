---
title: "Raspberry Pi で二酸化炭素濃度を測定する"
date: 2021-06-19T16:47:39+09:00
draft: false
categories: [IoT]
tags: [MH-Z19B,Python,Raspberry Pi]
thumbnailImagePosition: left
thumbnailImage: /images/co2_with_raspi/hard1.jpg
---

# はじめに
NDIR( nondispersive infrared , 非分散型赤外線 )型センサーである Winsen の MH-Z19B を Raspberry Pi 3B+ に接続し、部屋の二酸化炭素濃度を測定してみます。

## 最終ゴール
最初に本記事の最終ゴールを掲載します。

### ハードウェア構成
今回実現するシステムのハードウェア構成を下記に示します。

* 本体: Raspberry Pi 3B+
* ケース: Smraza Raspberry Pi 3B+
* CO2センサー: MH-Z19B+Terminal
* 電源: Cheero Canvas 3200mAh CHE-061

{{<figure src="/images/co2_with_raspi/hard1.jpg" caption="システムの全体像">}}

### 二酸化炭素濃度の測定
Rspberry Pi 上の Python に mh-z19 モジュールをインストール後、mh_z19 を実行するとMH-Z19B センサーから二酸化炭素濃度を取得し表示することができます。

{{<figure src="/images/co2_with_raspi/measured.png" caption="二酸化炭素濃度測定結果の表示">}}

# 準備
## 二酸化炭素濃度センサーの選定
[WisenのWebサイトでCO2 Gas Sensorの製品](https://www.winsen-sensor.com/sensors/co2-sensor/)を確認すると、

* NDIR型CO2センサーとして
  * MH-Z14,MH-Z14A,MH-Z14B
  * MH-Z16
  * MH-Z19B,MH-Z19C
  * MH-410D
  * MH711A
* NDIR以外の方式のセンサーとして
  * MD62
  * MG812
  * ZPHS01B（CO2以外のセンサーも搭載）
  * その他

といろいろあります。  
今回、Rapsberry Pi の GPIO 端子に接続して使用するため、上記の CO2 センサーの中で UART に対応しているものを選びます（WisenのCO2 Gas Sensor製品のWebページの各センサーの下にある[view details]のリンクから詳細仕様を確認し、Output Signal欄にUARTが記載されていればOKで、多くのセンサーがUARTに対応していました）。その中で今回は[MH-Z19B](https://www.winsen-sensor.com/sensors/co2-sensor/mh-z19b.html)を選定しました。

## MH-Z19Bセンサーの購入
Googleで"MH-Z19Bセンサー"を検索すると…  
Amazonのサイトが出てきますが、Amazonのサイトで購入すると4500円以上になります。
更に見ていくとAliExpressのサイトが出てきます。こちらのサイトだと$18程度（送料別）で購入できそうです。
そこで、AliExpressのサイトで肯定的フィードバックが98.7%の[”Your Cee”というショップ](https://ja.aliexpress.com/item/4000261110777.html)でMH-Z19Bを購入することにしました。
このショップでは、MH-Z19Bを
1. センサー単体
2. センサー+Pin
3. センサー+Terminal

の3種類のものが購入できます。 
{{<figure src="/images/co2_with_raspi/yourcee_mh_z19b.png" caption="Your Ceeショップで購入可能なMH-Z19B">}}
今回、ケーブルが付いている”MH-Z19B＋Terminal”を選択しました（ブレッドボードを用いた試作の場合はMH-Z19B+Pinを購入すると、自分でPinヘッダーをはんだ付けする手間が省けます）。

#### 購入費用と納期
MH-Z19Bセンサー購入の費用は
* MH-Z19B+Terminal　2145円
* 送料　476円
* 合計　2621円

納期は実績として2週間（2021/5/15に注文し、5/29に到着しました）でした。

## MZ-H19B接続ケーブルの改造
Raspberry PiのGPIOピンは2.54㎜ピッチで並んでいます。
{{<figure src="/images/co2_with_raspi/raspberrypi_pin.jpg" caption="Raspberry Pi B+のGPIOピン">}}
なお、GPIOピンの配置は[こちら](https://github.com/raspberrypi/documentation/blob/develop/documentation/asciidoc/computers/os/using-gpio.adoc)を参照して下さい。 
それに対し、下記MH-Z19Bのケーブルを接続するには
{{<figure src="/images/co2_with_raspi/mz_h19b_with_terminal.jpg" caption="MH-Z19B+Terminal付属ケーブル">}}
1. ２つある白いコネクタの一方から、Raspberry Piに接続する4本のケーブル（赤-5V Power用、黒-GND用、青-GPIO14(UART TX)用、緑-GPIO15(UART RX)用）を外します。
2. 下記パーツを使いQIコネクタに変更します。
   * QIコネクタハウジング
   * ケーブル用コネクタ

{{<figure src="/images/co2_with_raspi/qi_connecter.jpg" caption="MH-Z19B付属の接続ケーブル4本（赤、黒、緑、青）をQIコネクタに変更する">}}

### (補足)Terminalのピン配置

MH-Z19BのTerminalのピン配置の情報は[https://www.winsen-sensor.com/d/files/MH-Z19B.pdf](https://www.winsen-sensor.com/d/files/MH-Z19B.pdf)から入手可能で、該当箇所のみ抜き出すと下記のようになっています。
{{<figure src="/images/co2_with_raspi/terminal_pins.png" caption="MH-Z19BのTerminalのピン配置">}}
そのため、今回購入したMH-Z19B＋Terminalの接続ケーブルでは、Pin3-黒、Pin4-赤、Pin5-青、Pin6-緑ということになります。

### 改造後のMH-Z19Bと接続ケーブル
最終的に下記のように改造しました。
{{<figure src="/images/co2_with_raspi/mz_h19b_with_qi_connecter.jpg" caption="改造後の接続ケーブルとMH-Z19Bセンサー">}}

## Raspberry Pi 3B+とMH-Z19Bの接続
MH-Z19B側 - Raspberry Pi 3B+側との接続を下記のように行います。

* 赤 - 5V Power用
* 黒 - Ground
* 青 - GPIO14(UART TX)
* 緑 - GPIO15(UART RX)

{{<figure src="/images/co2_with_raspi/rpi_w_mhz19b.jpg" caption="MH-Z19BとRaspberryPiの接続">}}
{{<figure src="/images/co2_with_raspi/rpi_gpio_connecter.jpg" caption="GPIO部の拡大">}}

## Raspberry Pi 3B+のUART有効化
Raspberry Piのドキュメントにあるように、3B+ではデフォルトでUART0は/dev/serial1に割り当てられており、/dev/ttyAMA0のBluetooth接続されています。そして、UART1は/dev/serial0でmini UARTに割り当てられていますが、mini UARTはデフォルトでは有効化されていません。  
そこで、
1. エディタで/boot/config.txtを開き、enable_uart=1を追加します。
{{<figure src="/images/co2_with_raspi/tail_boot_config.png" caption="/boot/config.txtにenable_uart=1を追加する">}}
2. Raspberry Piを再起動します。
{{<figure src="/images/co2_with_raspi/reboot.png" caption="shutdownコマンドで再起動する">}}
3. 再起動後、/dev/serial*を確認し、下記のようになっていればOK！
{{<figure src="/images/co2_with_raspi/serial_check.png" caption="再起動後に/dev/serial*を確認する">}}

## Pythonモジュールmh-z19のインストール
PythonでMH-Z19Bを扱うため[PyPI(Python Package Index)](https://pypi.org)の[mh-z19モジュール](https://pypi.org/project/mh-z19/)より下記のようにインストールします。なお、mh-z19モジュールのソースコードは[GitHub上の@UedaTakayuki氏のリポジトリ](https://github.com/UedaTakeyuki/mh-z19)にMITライセンスで公開されています。
```
$ sudo pip install mh-z19
```
{{<figure src="/images/co2_with_raspi/1_install.png" caption="mh_z19モジュールのインストール">}}

# 二酸化炭素濃度の測定
動作確認も兼ねて、下記のようにMH-Z19Bセンサーを読み取ります。
```
$ sudo python -m mh_z19
{"co2": (二酸化炭素濃度ppm)}
```
{{<figure src="/images/co2_with_raspi/2_measure.png" caption="二酸化炭素濃度の測定">}}
問題なく二酸化炭素濃度を測定できているようです。
MH-Z19Bは二酸化炭素濃度以外にも気圧(UhUl)や温度(temperature)も計測できるようです。
{{<figure src="/images/co2_with_raspi/hard1.jpg" caption="今回作成したもの">}}

# 最後に
センサーから二酸化炭素濃度を取得することができたので、今後は、二酸化炭素濃度がある濃度以上になったら警告したり、二酸化炭素濃度の推移をグラフ表示したりできると良いと思いました。