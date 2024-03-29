---
title: "Arduino nano + CCS811で二酸化炭素濃度を測定する"
date: 2021-07-11T23:20:45+09:00
draft: false
categories: [IoT]
tags: [CCS811, Arduino nano]
thumbnailImagePosition: left
thumbnailImage: /images/co2_with_arduino/result.jpg
---

{{< toc >}}

# はじめに

今回は、Arduino nano に CCS811(空気品質センサー) を接続し、部屋の二酸化炭素濃度を測定してみます。

# 使用したパーツ

- Arduino Nano：COODENKEY Mini ATmega328P Nano V3.0 互換 マイクロコントローラーボード[^1]
  {{< figure src="/images/co2_with_arduino/nano1.png" link="/images/co2_with_arduino/nano1.png" title="Arduino Nano V3.0互換機" >}}
  [^1]: 購入先 Amazon https://www.amazon.co.jp/gp/product/B089ZXTWFK
- ディスプレイモジュール： OLED 1.3 インチ ７ Pin SPI SH1106[^2]
  {{< figure src="/images/co2_with_arduino/oled1.png" link="/images/co2_with_arduino/oled1.png" title="1.3インチOLEDディスプレイ" >}}
  [^2]: 購入先 AliExpress https://ja.aliexpress.com/item/1005001950055514.html
- 空気品質センサ：CJMCU-811[^3]
  {{< figure src="/images/co2_with_arduino/ccs8111.png" link="/images/co2_with_arduino/ccs8111.png" title="CCS811ガスセンサー" >}}
  [^3]: 購入先 AliExpress https://ja.aliexpress.com/item/32901973013.html

## （補足）CCS811 センサー、CJMCU-811 について

http://www.ne.jp/asahi/shared/o-family/ElecRoom/AVRMCOM/CCS811/CCS811_test.html の情報を読むと、CCS811 センサーは Cambridge CMOS Sensors Ltd という会社が開発したものと思われます。同社の Web サイトとして、www.ccmoss.comという記載もありますが、現在このURLにアクセスしてもWebサイトが表示されません。

一方、CCS811 センサーについては[ScioSense の商品紹介ページ](https://www.sciosense.com/products/environmental-sensors/ccs811-gas-sensor-solution/)に詳細に情報がでていました。
このページの[Dashboard Software]のリンクをクリックすると
https://downloads.sciosense.com/ccs811/
のページが表示されます。そこには、C++用ドライバーが[sciosense の Github リポジトリ](https://github.com/sciosense/CCS811_driver)に[MIT ライセンスとして公開](https://github.com/sciosense/CCS811_driver/blob/97abe9b45b5793f030f39bd06f0d07f166800377/LICENSE.md)されていることが記述されていました。

# 接続準備

## ピンヘッダのハンダ付け

先の機材を購入すると OLED ディスプレイモジュールを除き、Pin ヘッダーが同封されてきます。
今回、ブレッドボードを使って配線しますので、Pin ヘッダーをハンダ付けします。

- Arduino Nano 互換ボードに Pin ヘッダーをハンダ付け
  {{< figure src="/images/co2_with_arduino/nano3.png" link="/images/co2_with_arduino/nano3.png" title="互換ボード(横から)w/ピンヘッダ" >}}
  {{< figure src="/images/co2_with_arduino/nano2.png" link="/images/co2_with_arduino/nano2.png" title="互換ボード(下から) w/ピンヘッダ" >}}
- CJMCU-811(CCS811 センサー)に Pin ヘッダーをハンダ付け
  {{< figure src="/images/co2_with_arduino/ccs8113.png" link="/images/co2_with_arduino/ccs8113.png" title="CJMCU-811(横から) w/ピンヘッダ" >}}
  {{< figure src="/images/co2_with_arduino/ccs8112.png" link="/images/co2_with_arduino/ccs8112.png" title="CJMCU-811(下から) w/ピンヘッダ" >}}

## 配線

ハンダ付けした Pin ヘッダーをブレッドボードに差し、下記のように配線します。
{{< figure src="/images/co2_with_arduino/wiring.png" link="/images/co2_with_arduino/wiring.png" title="配線検討図">}}
各パーツ間は下記のように接続します。

- Arduino Nano - SH1103 間
  - 5V - VDO
  - GND - GND
  - D8 - RES
  - D9 - DC
  - D10 - CS
  - D11 - SDA
  - D13 - SCK
- Arduino Nano - CJMCU-811 間
  - 5V - VCC
  - GND - GND
  - GND - WAK(WAK を GND に接続しないと動作せず)
  - A3 - SCL
  - A4 - SDA

実際に配線したものを下記に掲載します。
{{< figure src="/images/co2_with_arduino/wiring2.png" link="/images/co2_with_arduino/wiring2.png" title="実際の配線(5VとGNDは上側に持っていきました)">}}

# eCO2 値の取得

## 利用する CCS811 センサー用ライブラリ

今回 AliExpress で購入した CJMCU-811 は使用するライブラリーに関する情報が提供されていませんでした。そこで、Google 検索をして、CCS811 センサーを扱うライブラリーとして MIT ライセンスで公開されている SparkFun CCS811 Arduino Library[^4] を使用することにしました。
[^4]: 参照 URL https://github.com/sparkfun/SparkFun_CCS811_Arduino_Library

### 実装例を用い eCO2 値が取得できるか確認する

同 GitHub リポジトリには Arduino で CCS811 のセンサー値を読み取り、シリアル通信でその値を確認できる実装例：Example1_BasicReadings.ino が公開されています。
そこで、まずこの実装例を利用し、今回利用する Arduino Nano ＋ CJMCU-811 で動作するように変更しました。

尚、実装例のコメントに下記が記載されていました。実際に CCS811 を利用する際は留意する必要があります。

- A new sensor requires at 48-burn in. - 新しいセンサーは 48 時間の「慣らし」が必要。
- Once burned in a sensor requires 20 minutes of run in before readings are considered good. - 一度慣らしを終えたセンサーは、良好な値を得るため使用前に 20 分間の暖気が必要。

{{< gist aktnk 580fb53f0559bf1a5bb9259110d0e09b co2_serial.ino >}}

### 実行結果

上記コードを Arduino IDE に張り付け、コンパイルエラー等発生しないことを「検証・コンパイル」で確認し、「マイコンボードに書き込む」を実行しました。

その後、シリアルポートの出力を確認すると、下記のようになり、無事 CCS811 センサーより eCO2 値を取得できていることが確認できました。

```
12:26:56.013 -> CCS811 Basic Example
12:27:00.012 -> CO2[400] tVOC[0] millis[3997]
12:27:01.008 -> CO2[400] tVOC[0] millis[4994]
12:27:02.009 -> CO2[400] tVOC[0] millis[5989]
12:27:03.012 -> CO2[400] tVOC[0] millis[6975]
12:27:04.011 -> CO2[400] tVOC[0] millis[7970]
12:27:04.965 -> CO2[400] tVOC[0] millis[8967]
12:27:05.968 -> CO2[400] tVOC[0] millis[9962]
12:27:06.971 -> CO2[400] tVOC[0] millis[10958]
12:27:07.969 -> CO2[400] tVOC[0] millis[11943]
12:27:08.968 -> CO2[400] tVOC[0] millis[12940]
12:27:09.970 -> CO2[400] tVOC[0] millis[13936]
```

## SH1106 OLED への表示

### 利用する SH1106 ドライバー

今回購入した AliExpress で購入した SH1106 OLED ディスプレイについても、ライブラリーに関する情報が提供されていませんでした。
そこで、こちらも Google 検索したところ、Arduino-er の Hello World 1.3 inch IIC/SPI 128x64 OLED x Arduino, using u8glib library という記事 で u8glib ライブラリを使用した解説がされていました。
u8glib は BSD ライセンスでhttps://github.com/olikraus/u8glib にて公開されていますが、既に開発は止まっているようです。
u8g2 ライブラリの開発へ移行しているのでそちらを使うようにとの記載がありますが、今回は u8glib を使いました。

u8glib の GitHub リポジトリには使い方の説明もドキュメント化されていますので、https://github.com/olikraus/u8glib/wiki/thelloworld を参考に、先ほどの CCS811 のコードに実装を追加しました。

{{< gist aktnk 580fb53f0559bf1a5bb9259110d0e09b co2_oled.ino >}}

尚、20 分間の暖気が必要ということで、起動後 20 分間は OLED の 1 行目の表示を  
"= CO2 Monitor \*="  
としています。20 分経過後すると、  
"= CO2 Monitor ="  
と表示するようにしています。

### 実行結果

上記コードを Arduino IDE に張り付け、コンパイルエラー等発生しないことを「検証・コンパイル」で確認し、「マイコンボードに書き込む」を実行しました。  
その結果、下記のように SH1106 OLED ディスプレイに eCO2 値が表示されました。
{{< figure src="/images/co2_with_arduino/result.jpg" link="/images/co2_with_arduino/result.jpg" title="CCS811センサー値をSH1106 OLEDディスプレイに表示" >}}

# 最後に

二酸化炭素濃度を測定できる MH-Z19B と CCS811 が入手できましたので、どこかでこれらの比較ができるとよいですね。
