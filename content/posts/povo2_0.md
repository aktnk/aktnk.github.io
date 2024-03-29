---
title: "KYF38のかけ放題をauからpovo2.0に変更し、月3575円から月1650円に"
date: 2022-07-04T17:49:06+09:00
draft: false
categories: [lifehack]
tags: [povo2.0, MNO, MVNO, KYF38, au]
thumbnailImagePosition: left
thumbnailImage: /images/povo2_0/povo2app.png
---

{{< toc >}}

# はじめに

povo2.0 のかけ放題プランにするとケータイ代を月 2000 円近く安くできるようなので、私もやってみました。

## au での利用状況

- 実家の母は au の 4G ケータイ KYF38（京セラ GRANTINA） をかけ放題で利用中
- これまでの月額料金は SMS を除いて、[au ケータイプラン](https://www.au.com/mobile/charge/4glte-featurephone/plan/k-tai/)にあるように 1265 円+通話定額 2 の 1980 円、合計 3575 円
- 回線契約は私で、利用者として母を登録

# povo2.0 かけ放題への道のり

## 対応機種の調査・確認

- [povo の Web サイトの対応端末](https://povo.jp/product/)の一覧から、現在使用している京セラ GRANTINA KYF38 が povo に対応しているか確認します。
  ⇒ KYF37 は掲載されていますが、KYF38 は対応端末として掲載されていませんでした。  
   {{< figure title="対応端末" src="/images/povo2_0/supported_devices.png" link="/images/povo2_0/supported_devices.png" >}}

- Google で"povo2.0 KFY38"と検索すると、実用太郎のブログに[親父のガラホ kyf38 を povo2.0 運用に変更　毎月の支払いが激安に](https://cryhug949.blogspot.com/2021/10/kyf38povo20.html)の記事を見つけました。  
  ⇒ ダメなら Android 端末をシニア向けに変更して使えばよいと考え、今回 povo2.0 化をしてみることにしました。

## Android スマホに povo2.0 アプリをインストール

- 最初に調査した対応端末で KYF37 の注意点として「\*4：SIM カードの有効化やトッピングの購入等には、別途 povo2.0 アプリがご利用可能な端末をご用意していただく必要があります。」となっていました。  
  ⇒ 対応端末の SIM フリー端末に掲載されていた Android スマホを所有していましたので、KFY38 が使えなかったときのことを踏まえ、初期化し povo2.0 アプリをインストールしました。  
  {{< figure title="povo2.0アプリをインストールしたAndroid端末" src="/images/povo2_0/povo2app_android.png" link="/images/povo2_0/povo2app_android.png" >}}

## povo2.0 アプリを起動し「他社/UQ mobile から povo2.0 へお乗り換え」の手順に従い手続きを実施

### 手続きに必要なもの・こと

- 本人確認書類：マイナンバーカードでも構いませんが、契約する私の運転免許証を用意
- クレジットカード情報：契約者のクレジットカード情報
- 利用端末：KYF38（povo2.0 でかけ放題に変更する 4G ケータイ）
- 手続き端末：povo2.0 アプリをインストールした Android 端末

### 手続き

1. MNP 予約の実施  
   KYF38 から au の MNP 予約先 0077-75470 に電話し、MNP 予約を実施しました。  
   ⇒ SMS で MNP 予約番号が送られてきました。
   {{< figure title="MNP予約結果" src="/images/povo2_0/mnp_reserved.png" link="/images/povo2_0/mnp_reserved.png" >}}

1. povo2.0 アプリでの手続き

   1. メールアドレスの登録
      入力したメールアドレス宛に認証コードが送付されるので、認証コードを入力しメールアドレス登録＝アカウント登録を完了させる。
   1. 契約手続きの実施
      - SIM/契約タイプの指定
      - 重要事項説明の確認
      - 契約者である私のクレジットカード情報の入力
   1. eKYC（電子本人認証）の実施
      SIM の送付先等指示に従い情報を入力するとともに、povo2.0 アプリをインストールした Android 端末のカメラを使い、契約者の運転免許証、契約者の顔写真を指示に従い撮影しアップロードします。  
      ⇒ アップロードが完了すると、本人確認書類を受付た旨のメールが届きます。  
      ⇒ その後、本人確認が無事完了すると本人確認完了の旨のメールが届きます。  
      ⇒ povo2.0 アプリでは下記のように表示されます。  
      {{< figure title="本人確認完了" src="/images/povo2_0/povo2proc1.png" link="/images/povo2_0/povo2proc1.png" >}}

      **(補足)** なお、私が実施したとき 2 度本人確認がエラーとなり、"「povo2.0 へのご確認/SIM 再発行」を受け付けることができませんでした…"とのメールが来ました。povo2.0 アプリのチャットからサポートへ連絡し、エラーの原因を確認していただきました。その結果、暗い場所で撮影したため、写真がブレたり、ノイズが乗って鮮明でないことが原因のようでした。

1. povo2.0 の SIM 到着後の回線開通の手続き  
   povo2.0のSIMが到着したら、下記手順に従い、SIMの有効化後、apn設定、通話テストを行います。

   1. povo2.0アプリを起動し、「SIMカードを有効化する」ボタンをタップすると、バーコードをスキャンする旨メッセージが表示されます。表示に従いpovo MULTI IC CARDの裏面下部にあるバーコードをスキャンします。
      {{< figure title="SIM有効化" src="/images/povo2_0/sim_activate.png" link="/images/povo2_0/sim_activate.png" >}}
   1. 有効化が完了した旨のメッセージが表示されるので、KYF38の電源をOFFし、裏蓋を開けます。バッテリーを外して、povo2.0のSIMを挿入します。
      {{< figure title="SIMの挿入" src="/images/povo2_0/sim_insert.png" link="/images/povo2_0/sim_insert.png" >}}
   1. KYF38にバッテリーを取付け、裏蓋を閉じます。そして、KYF38の電源を入れます。
   1. povo2.0アプリに表示される apn 設定内容に従い、KYF38 の APN を設定します。
      {{< figure title="APN設定情報" src="/images/povo2_0/settings_info.png" link="/images/povo2_0/settings_info.png" >}}
      1. KYF38 の「メインメニュー」を開きから「設定を行う」を選択します。  
         {{< figure title="メインメニュー" src="/images/povo2_0/setting2.png" link="/images/povo2_0/setting2.png" >}}
      1. 表示された「設定」から「その他の設定を行う」を選択します。
         {{< figure title="その他の設定" src="/images/povo2_0/setting3.png" link="/images/povo2_0/setting3.png" >}}
      1. 表示された「その他の設定」から「開発者向けオプション」を選択します。  
         {{< figure title="開発者向けオプション" src="/images/povo2_0/setting4.png" link="/images/povo2_0/setting4.png" >}}  
         もし「開発者向けオプション」が表示されない場合は、「端末の情報を表示する」を選択し表示された「ビルド番号」を 7 回選択すると「開発者向けオプション」が表示されます。
      1. 表示された「開発者向けオプション」から「モバイルネットワーク」を選択します。
         {{< figure title="モバイルネットワーク" src="/images/povo2_0/setting5.png" link="/images/povo2_0/setting5.png" >}}
      1. 表示された「モバイルネットワーク」から「アクセスポイント名」を選択します。
         {{< figure title="アクセスポイント名" src="/images/povo2_0/setting6.png" link="/images/povo2_0/setting6.png" >}}
      1. povo2.0 アプリに表示された APN 設定情報を下記のように入力し、「決定」ボタンを押します。
         - 名前:povo2.0
         - APN:povo.jp
         - APN プロトコル:IPv4/IPv6
           {{< figure title="APN設定１" src="/images/povo2_0/setting8.png" link="/images/povo2_0/setting8.png" >}}
           {{< figure title="APN設定２" src="/images/povo2_0/setting9.png" link="/images/povo2_0/setting9.png" >}}
   1. 発信テスト用番号`111`に電話をかけます。ガイダンスを最後まで確認して、通話を切断します。

## povo2.0 アプリで「かけ放題」オプションを有効化

1. povo2.0 アプリを起動し、「かけ放題」オプションを有効化します。
   {{< figure title="かけ放題の設定" src="/images/povo2_0/activate.png" link="/images/povo2_0/activate.png" >}}
1. povo2.0 アプリの「詳細な内訳」を選択して、下記のように表示されれば「かけ放題」が有効化されています。  
   {{< figure title="かけ放題の確認" src="/images/povo2_0/povo2app_now.png" link="/images/povo2_0/povo2app_now.png" >}}

# まとめ

- '22/6/12 に povo2.0 に切り替えてから、KYF38 で今のところ問題なく使用できています。
- しかしながら、povo2.0 として正式に対応端末として公開されていませんので、今後も問題なく使用できるかはわかりません。
- 本記事を参考にされ、KYF38 で povo2.0 に乗り換える方は、当方では責任を追うことができません。あくまでもご自身の責任で行ってください。
