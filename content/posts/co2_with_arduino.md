---
title: "Arduino nano + CCS811で二酸化炭素濃度を測定する"
date: 2021-07-11T23:20:45+09:00
draft: false
categories: [IoT]
tags: [CCS811,Arduino nano]
thumbnailImagePosition: left
thumbnailImage: /images/co2_with_arduino/result.jpg
---

# はじめに
今回は、Arduino nano に CCS811(空気品質センサー) を接続し、部屋の二酸化炭素濃度を測定してみます。

## 使用したパーツ
* Arduino Nano：COODENKEY Mini ATmega328P Nano V3.0互換 マイクロコントローラーボード[^1]
{{<figure src="/images/co2_with_arduino/nano1.png" height="120" caption="Arduino Nano V3.0互換機">}}
[^1]: 購入先 Amazon https://www.amazon.co.jp/gp/product/B089ZXTWFK
* ディスプレイモジュール： OLED 1.3インチ ７Pin SPI SH1106[^2]
{{<figure src="/images/co2_with_arduino/oled1.png" caption="1.3インチOLEDディスプレイ">}}
[^2]: 購入先 AliExpress https://ja.aliexpress.com/item/1005001950055514.html
* 空気品質センサ：CJMCU-811[^3]
{{<figure src="/images/co2_with_arduino/ccs8111.png" caption="CCS811ガスセンサー">}}
[^3]: 購入先 AliExpress https://ja.aliexpress.com/item/32901973013.html

#### （補足）CCS811センサー、CJMCU-811について
http://www.ne.jp/asahi/shared/o-family/ElecRoom/AVRMCOM/CCS811/CCS811_test.html の情報を読むと、CCS811センサーは Cambridge CMOS Sensors Ltdという会社が開発したものと思われます。同社のWebサイトとして、www.ccmoss.comという記載もありますが、現在このURLにアクセスしてもWebサイトが表示されません。

一方、CCS811センサーについては[ScioSenseの商品紹介ページ](https://www.sciosense.com/products/environmental-sensors/ccs811-gas-sensor-solution/)に詳細に情報がでていました。
このページの[Dashboard Software]のリンクをクリックすると
https://downloads.sciosense.com/ccs811/
のページが表示されます。そこには、C++用ドライバーが[sciosenseのGithubリポジトリ](https://github.com/sciosense/CCS811_driver)に[MITライセンスとして公開](https://github.com/sciosense/CCS811_driver/blob/97abe9b45b5793f030f39bd06f0d07f166800377/LICENSE.md)されていることが記述されていました。

# 接続準備
## ピンヘッダのハンダ付け
先の機材を購入するとOLEDディスプレイモジュールを除き、Pinヘッダーが同封されてきます。
今回、ブレッドボードを使って配線しますので、Pinヘッダーをハンダ付けします。
* Arduino Nano互換ボードにPinヘッダーをハンダ付け
{{<figure src="/images/co2_with_arduino/nano3.png" caption="互換ボード(横から)w/ピンヘッダ">}}
{{<figure src="/images/co2_with_arduino/nano2.png" caption="互換ボード(下から) w/ピンヘッダ">}}
* CJMCU-811(CCS811センサー)にPinヘッダーをハンダ付け
{{<figure src="/images/co2_with_arduino/ccs8113.png" caption="CJMCU-811(横から) w/ピンヘッダ">}}
{{<figure src="/images/co2_with_arduino/ccs8112.png" caption="CJMCU-811(下から) w/ピンヘッダ">}}

## 配線
ハンダ付けしたPinヘッダーをブレッドボードに差し、下記のように配線します。
{{<figure src="/images/co2_with_arduino/wiring.png" caption="配線検討図">}}
各パーツ間は下記のように接続します。
* Arduino Nano - SH1103 間
  * 5V - VDO
  * GND - GND
  * D8 - RES
  * D9 - DC
  * D10 - CS
  * D11 - SDA
  * D13 - SCK
* Arduino Nano - CJMCU-811 間
  * 5V - VCC
  * GND - GND
  * GND - WAK(WAKをGNDに接続しないと動作せず)
  * A3 - SCL
  * A4 - SDA

実際に配線したものを下記に掲載します。
{{<figure src="/images/co2_with_arduino/wiring2.png" caption="実際の配線(5VとGNDは上側に持っていきました)">}}

# eCO2値の取得
## 利用するCCS811センサー用ライブラリ
今回AliExpressで購入したCJMCU-811は使用するライブラリーに関する情報が提供されていませんでした。そこで、Google検索をして、CCS811センサーを扱うライブラリーとしてMITライセンスで公開されているSparkFun CCS811 Arduino Library[^4] を使用することにしました。
[^4]: 参照URL https://github.com/sparkfun/SparkFun_CCS811_Arduino_Library

### 実装例を用いeCO2値が取得できるか確認する
同GitHubリポジトリにはArduinoでCCS811のセンサー値を読み取り、シリアル通信でその値を確認できる実装例：Example1_BasicReadings.ino が公開されています。
そこで、まずこの実装例を利用し、今回利用するArduino Nano＋CJMCU-811で動作するように変更しました。

尚、実装例のコメントに下記が記載されていました。実際にCCS811を利用する際は留意する必要があります。
* A new sensor requires at 48-burn in. - 新しいセンサーは48時間の「慣らし」が必要。
* Once burned in a sensor requires 20 minutes of run in before readings are considered good. - 一度慣らしを終えたセンサーは、良好な値を得るため使用前に20分間の暖気が必要。

{{< highlight CPP "linenos=true">}}
#include <wire.h>
#include "SparkFunCCS811.h" //Click here to get the library: http://librarymanager/All#SparkFun_CCS811
 
//#define CCS811_ADDR 0x5B //Default I2C Address
#define CCS811_ADDR 0x5A //Alternate I2C Address
 
CCS811 mySensor(CCS811_ADDR);
 
void setup()
{
  Serial.begin(115200);
  Serial.println("CCS811 Basic Example");
 
  Wire.begin(); //Inialize I2C Hardware
 
  if (mySensor.begin() == false)
  {
    Serial.print("CCS811 error. Please check wiring. Freezing...");
    while (1)
      ;
  }
}
 
void loop()
{
  //Check to see if data is ready with .dataAvailable()
  if (mySensor.dataAvailable())
  {
    //If so, have the sensor read and calculate the results.
    //Get them later
    mySensor.readAlgorithmResults();
 
    Serial.print("CO2[");
    //Returns calculated CO2 reading
    Serial.print(mySensor.getCO2());
    Serial.print("] tVOC[");
    //Returns calculated TVOC reading
    Serial.print(mySensor.getTVOC());
    Serial.print("] millis[");
    //Display the time since program start
    Serial.print(millis());
    Serial.print("]");
    Serial.println();
  }
 
  delay(10); //Don't spam the I2C bus
}
{{< /highlight >}}

### 実行結果
上記コードをArduino IDEに張り付け、コンパイルエラー等発生しないことを「検証・コンパイル」で確認し、「マイコンボードに書き込む」を実行しました。

その後、シリアルポートの出力を確認すると、下記のようになり、無事CCS811センサーよりeCO2値を取得できていることが確認できました。
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

## SH1106 OLEDへの表示
### 利用するSH1106ドライバー
今回購入したAliExpressで購入したSH1106 OLEDディスプレイについても、ライブラリーに関する情報が提供されていませんでした。
そこで、こちらもGoogle検索したところ、Arduino-er の Hello World 1.3 inch IIC/SPI 128x64 OLED x Arduino, using u8glib library という記事 でu8glibライブラリを使用した解説がされていました。
u8glibはBSDライセンスでhttps://github.com/olikraus/u8glib にて公開されていますが、既に開発は止まっているようです。
u8g2ライブラリの開発へ移行しているのでそちらを使うようにとの記載がありますが、今回はu8glibを使いました。

u8glibのGitHubリポジトリには使い方の説明もドキュメント化されていますので、https://github.com/olikraus/u8glib/wiki/thelloworld を参考に、先ほどのCCS811のコードに実装を追加しました。

{{< highlight CPP "linenos=ture" >}}
#include <wire.h>
#include <u8glib.h>         // https://github.com/olikraus/u8glib
#include <sparkfunccs811.h> //Click here to get the library: http://librarymanager/All#SparkFun_CCS811
 
// CCS811 sensor
//#define CCS811_ADDR 0x5B        //Default I2C Address
#define CCS811_ADDR 0x5A          //Alternate I2C Address
#define WARMUP_TIME 1200000ul   // センサーウォームアップ時間 20min*60s*1000msec
CCS811 mySensor(CCS811_ADDR);
 
//OLEDモニタ使用のワイヤ定義
U8GLIB_SH1106_128X64 u8g(13, 11, 10, 9, 8);
#define UPDATE_INTERVAL 1000ul    // 表示更新間隔 1s*1000msec
unsigned long previous_time = 0ul;
 
void setup()
{
  Serial.begin(115200);
  Serial.println("CCS811 Basic Example");
 
  Wire.begin(); //Inialize I2C Hardware
 
  if (mySensor.begin() == false)
  {
    Serial.print("CCS811 error. Please check wiring. Freezing...");
    while (1)
      ;
  }
}
 
//画面表示
void draw(void) {
 //１列目：CO2 Meter
  u8g.setFont(u8g_font_unifont);
  u8g.setPrintPos(2,14);
  if (millis() <= WARMUP_TIME) { // センサWARM UP中
    u8g.print("= CO2 Monitor *=");
  }
  else {
    u8g.print("= CO2 Monitor =");
  }
  
 //２列目：二酸化炭素PPM実数表示
  u8g.setPrintPos(2,28);
  u8g.print("CO2:");
  u8g.print(mySensor.getCO2());
  u8g.print(" ppm");
  
 //３列目：総揮発性有機化合物PPB実数表示
  u8g.setPrintPos(2,42);
  u8g.print("TVOC:");
  u8g.print(mySensor.getTVOC());
  u8g.print(" ppb");
 
}
 
void loop()
{ 
  //Check to see if data is ready with .dataAvailable()
  if (mySensor.dataAvailable())
  {
    //If so, have the sensor read and calculate the results.
    //Get them later
    mySensor.readAlgorithmResults();
 
    // 表示更新間隔を過ぎていたらOLEDの表示を更新する
    if (millis() - previous_time > UPDATE_INTERVAL) {
      Serial.print("eCO2[");
      //Returns calculated CO2 reading
      Serial.print(mySensor.getCO2());
      Serial.print("] tVOC[");
      //Returns calculated TVOC reading
      Serial.print(mySensor.getTVOC());
      Serial.print("] millis[");
      //Display the time since program start
      Serial.print(millis());
      Serial.print("]");
      Serial.println();
      
      u8g.firstPage();
      do {
        draw();
      } while( u8g.nextPage() );
    }
  }
 
  delay(10); //Don't spam the I2C bus
}
{{< /highlight >}}

尚、20分間の暖気が必要ということで、起動後20分間はOLEDの1行目の表示を  
"= CO2 Monitor *="  
としています。20分経過後すると、  
"= CO2 Monitor ="  
と表示するようにしています。

### 実行結果
上記コードをArduino IDEに張り付け、コンパイルエラー等発生しないことを「検証・コンパイル」で確認し、「マイコンボードに書き込む」を実行しました。  
その結果、下記のようにSH1106 OLEDディスプレイにeCO2値が表示されました。
{{< figure src="/images/co2_with_arduino/result.jpg" caption="CCS811センサー値をSH1106 OLEDディスプレイに表示" >}}

# 最後に
二酸化炭素濃度を測定できるMH-Z19BとCCS811が入手できましたので、どこかでこれらの比較ができるとよいですね。
