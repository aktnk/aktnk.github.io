---
title: "ネットワークWiFiカメラ Tapo で遊ぶ"
date: 2023-06-18T18:06:23+09:00
draft: false
categories: [Video Streaming]
tags: [RTSP, RTMP, HLS, nginx, Tapo C110, Tapo C200]
thumbnailImagePosition: left
thumbnailImage: /images/rtsp_camera/Tapo_C200_C110.png
---

{{< toc >}}

***

# はじめに

[TP-Link の Tapo シリーズのカメラ](https://www.tp-link.com/jp/home-networking/cloud-camera/)はTP-LinkのサポートFAQ "[Tapoを使用したRTSPライブストリーミングの利用方法](https://www.tp-link.com/jp/support/faq/2680/)" にあるように、RTSPプロトコルでのアクセスが可能です。  
そこで、今回は自宅にある[Tapo C110](https://www.tp-link.com/jp/home-networking/cloud-camera/tapo-c110/) と[Tapo C200](https://www.tp-link.com/jp/home-networking/cloud-camera/tapo-c200/)を使ってHTTP Live Streaming(HLS)を試してみます。

[![Tapo C200 C110 仕様](/images/rtsp_camera/Tapo_C200_C110.png)](https://www.tp-link.com/jp/compare/?typeId=19&productIds=49667%2C61753) 

## HTTP Live Streaming(HLS)について

HLSは[Apple Inc.](https://www.apple.com)が開発した”音声と動画をライブ配信もしくはオンデマンド配信する技術”です。この技術の詳細は[https://developer.apple.com/streaming/](https://developer.apple.com/streaming/)に掲載されています。

少々古い記事ですが、[ケータイ用語の基礎知識　第554回：HLS形式とは](https://k-tai.watch.impress.co.jp/docs/column/keyword/515059.html) によると、下記のような仕組みになっています。

> HLSでは、エンコーダーによってデジタルデータ化された映像・音声信号をファイル化する際に、10秒単位の「MPEG-2 TS」として細切れにし、暗号化して、小さな連続したファイルを作ります。ちなみにエンコーダーは、デジタルデータの作成時に、ビットレートに応じたいくつかのバリエーションを作っておきます。
> そして、この細切れになった「MPEG-2 TS」ファイルの再生順や、暗号化の鍵、コンテンツのバリエーションなどを書き込んだM3U8形式の「プレイリスト」に記載しておきます。そして、これらをWebサーバーから配信する、という仕組みになっています。
> ![HLSの仕組み](https://asset.watch.impress.co.jp/img/ktw/docs/515/059/hls01_s.jpg)

## 何故HLSなのか？

Tapoのカメラが対応しているRTSPプロトコルは[Real-Time Streaming Protocol](https://datatracker.ietf.org/doc/html/rfc7826)ですが、Webブラウザでは表示することができず、[VLC media player](https://www.videolan.org/vlc/index.ja.html)等のアプリが必要です。

HLSを用いると、WebブラウザだけでTapoカメラの映像を視聴できるのでは？と思い、HLSを試してみることにしました。

## 目標

以上より、今回の目標は
* dockerコンテナを使い、Tapo C110 & C200 の RTSP 映像をiPhoneのWebブラウザから視聴できるようにすること

です。

***

# システムの構築

今回はUbuntu 20.04 LTSをインストールしたPC上にDockerによる仮想環境を用い、必要なサーバを構築します。

**(動作確認済み環境)**
* OS : Ubuntu 20.04LTS
* CPU : Intel(R) Core(TM) i7-4790 CPU @ 3.60GHz
* コンテナ環境:docker version 24.0.2, docker compose version v2.18.1
* 使用したTapoのカメラ
    * [Tapo C110(Amazonへのリンク)](https://amzn.to/3NZj7K0)
    * [Tapo C200(Amazonへのリンク)](https://amzn.to/3NDNi7M)

## 事前準備

**(前提条件)**
* Tapoアプリ（[iPhone/iPad用](https://apps.apple.com/jp/app/tp-link-tapo/id1472718009)、[Android用](https://play.google.com/store/apps/details?id=com.tplink.iot&hl=ja&gl=US)）を使い、C110・C200の初期設定（自宅の無線WiFiへの接続が完了し、映像が視聴できることを確認済）が完了しているものとします。


事前準備としてTapo C110・C200と自宅内LANのDHCPに下記の設定を実施します。

1. Tapoアプリを使い、RTSPプロトコルでC110・C200にアクセスする際に使用する”ユーザ名”と”パスワード”を設定します。  
[高度な設定]>[デバイスアカウント]を選択し、”ユーザ名”と”パスワード”を設定しますが、詳細は[Tapoを仕様したRTSPライブストリーミングの利用方法](https://www.tp-link.com/jp/support/faq/2680/)を確認してください。  
{{< figure src="/images/rtsp_camera/device_account_half.png" link="/images/rtsp_camera/device_account_half.png" title="ユーザ名・パスワードの設定" >}}
2. 自宅の無線WiFiでC110・C200に割り当てるIPアドレスを固定します。  
これは家庭のネットワーク環境の構成に依存しますが、無線LANルータなどが持つDHCP機能により自動的に割り当てている場合は、C110・C200のMACアドレスをTapoアプリから確認した上で、そのMACアドレスに割り当てるIPアドレスを固定する設定すれば良いはずです。  

なお、Tapoアプリを使うと、カメラのIPアドレス、Macアドレスが確認できます。
{{< figure src="/images/rtsp_camera/C110_settings.png" link="/images/rtsp_camera/C110_settings.png" title="C110設定状態" >}}

### （補足情報）

先に設定したC110＆C200のユーザ名(user)・パスワード(pass)を使用し、以下の2種類の映像が取得できます。今回はstream2を使い低画質映像を使用しました。
* 高画質映像（1920x1080）- `rtsp://user:pass@(C110のipアドレス):554/stream1`
* 低画質映像（640x360）- `rtsp://user:pass@(C110のipアドレス):554/stream2`

## システムの構成

今回実現したいことは、下記構成により実現します。

* ネットワークカメラのC110・C200から音声・動画をRTSPプロトコルで受け取り、RTMPプロトコルに変換するサーバ。  
このサーバはFFmpegを実行させRTSP⇒RTMPへ変換しているだけですが、カメラ毎に用意します（今回は2台）。
* RTMPプロトコルで配信された音声・動画をHTTP Live Streaming(HLS)プロトコルで配信するWebサーバ。  
このサーバはnginxにlibnginx-rtmp-modを組み込んで使用します。RTMPプロトコルで入手した音声・映像をhls形式に変換し、HTML5とvideo.jsを用いたWebページとして公開し、ブラウザから音声・映像を視聴できるようにします。

{{< svg src="../../static/images/rtsp_camera/system_diagram.drawio.svg" title="システム構成" >}}

## HLS配信サーバ

rtmpモジュールを組み込んだnginxサーバを使い、HLS配信サーバを実現します。  
そのためには、下記の2つの機能をnginx.confにて設定する必要があります。

* RTMPサーバ機能 - 別途立ち上げたffmpegからC110の映像を1935番ポートで受け取ります。そして、受け取った映像データをhls形式に変換し、/data/hlsディレクトリに書き出します。
* WWWサーバ機能 - 80番ポートでhttpのリクエストを受け取り、/dataディレクトリ配下のhtmlおよびRTMPサーバがhls形式に変換した映像ファイルをコンテンツとして公開します。

### nginx_rtmp用Dockerfileの作成

* ubuntu:20.04のdocker公式イメージを使い、nginxとlibnginx-rtmp-modをインストールします。
* タイムゾーンをAsia/Tokyoに設定しています。

{{< gist aktnk 07f507af5c5866868ba2b76037aa5560 "nginx_rtmp_Dockerfile" >}}

### nginx_rtmp用nginx.comfの作成

* httpコンテキスト
    * ルートコンテンツは/dataディレクトリに配置します。
    * hls配信で使用するメディアタイプを定義します。
* rtmpコンテキスト
    * 1935番ポートの/livecamでRTMPプロトコルで音声・映像を受け取ります。
    * /livecamのパスで指定されたストリーム名で、/data/hlsディレクトリにhls形式ファイルを保存します。

{{< gist aktnk 07f507af5c5866868ba2b76037aa5560 "nginx_rtmp_nginx.conf" >}}

### 公開用htmlの作成

* iPhone、Androidのスマホ、iPadやWindows PCで映像が視聴できるよう、HTML5にvideo.js を使い
下記のようにhtmlファイルを作成します。
* c110.htmlの他に、c200.htmlも同様な内容で作成します。

{{< gist aktnk 07f507af5c5866868ba2b76037aa5560 "data_c110.html" >}}

## RTSPをRTMPに変換するサーバ

今回の実行環境であるubuntu20.04LTSのターミナル上で、ffmpegコマンドのパラメータを与え、実際にTapo C110＆C200のrtsp配信映像がrtmpに変換できるか事前に確認します。
その際、先に作成したnignx_rtmpコンテナを実行(`docker compose up -d nginx_rtmp`)しておきましょう。

今回使用したカメラ以外のものを使う場合には、ffmpegのパラメータを変更しないと正常に動作しないこともあるので注意してください。
なお、ffmpegの基本的な使い方は、[Qiita @yh1224さんの”FFmpeg の使い方 (基本)”](https://qiita.com/yh1224/items/38d4ef1cf768aa3368d6)が参考になります。

```
$ ffmpeg -rtsp_transport tcp -i rtsp://user:pass@(C110のipアドレス):554/stream1 \
    -c:v libx264 -b:v 512k -r 15 -x264-params keyint=30:scenecut=0 \
    -vf hue=h=10 -c:a aac -f flv rtmp://(nginx_rtmpサーバのIPアドレス)/livecam/std
```
### ffmpegサーバで実行するconver.shの作成

* 先に動作確認したffmpegのパラメータを使い、conver.shを作成します。
* TapoのC110＆C200とも、`-ar 11025`を指定しないとエラーが発生し、動作しませんでした。
* [user][pass]はTapoのカメラに設定したユーザ名、パスワードを指定します。
* [camera ip-address]はTapoのカメラに割り当てたIPアドレスを指定します。
* {RTMP_SERVER_URL}{STREAM_NAME}はcompose.yml側で定義します。

{{< gist aktnk 07f507af5c5866868ba2b76037aa5560 "camera_convert.sh" >}}

### ffmpeg用Dockerfileの作成

* ffmpegのdockerイメージは、[docker hub @jrottenberg](https://hub.docker.com/r/jrottenberg/ffmpeg/)、[GitHub @jrottenberg](https://github.com/jrottenberg/ffmpeg)を使用しました。

{{< gist aktnk 07f507af5c5866868ba2b76037aa5560 "camera_Dockerfile" >}}

***

# 動作確認

今回作成したソースファイル一式を[https://github.com/aktnk/HTTP-Live-Streaming-of-RTSP-cameras](https://github.com/aktnk/HTTP-Live-Streaming-of-RTSP-cameras)に公開しています。
動作確認はこのソースファイルを用いて実施します。
なお、一部の設定値はご自身の環境に合わせて変更する必要があります。

## コンテナのビルド＆実行

`docker compose up -d --build`を実行し、コンテナをビルドした後、起動させる。
問題がなければ、下記のように2つのコンテナが`Started`となるはず。
```
$ docker compose up -d --build
[+] Building 9.1s (25/25) FINISHED                                                                                                      

＜省略＞

[+] Running 4/4
 ✔ Network qiita_softark2_default  Created                                                                                          0.1s
 ✔ Container nginx_rtmp            Started                                                                                          3.1s
 ✔ Container camera_c110           Started                                                                                          2.1s
 ✔ Container camera_c200           Started                                                                                          2.2s
```

## コンテナのプロセスの確認

compose.ymlにて指定したhealthcheckに従い、コンテナの実行状態がチェックされる。

* コンテナを起動処理が完了していないと下記のように表示される。
```
$ docker compose ps
NAME                IMAGE                        COMMAND                  SERVICE             CREATED             STATUS                            PORTS
camera_c110         qiita_softark2-camera_c110   "/bin/sh -c 'exec /h…"   camera_c110         10 seconds ago      Up 7 seconds (health: starting)  
camera_c200         qiita_softark2-camera_c200   "/bin/sh -c 'exec /h…"   camera_c200         10 seconds ago      Up 7 seconds (health: starting)  
nginx_rtmp          qiita_softark2-nginx_rtmp    "nginx -g 'daemon of…"   nginx_rtmp          12 seconds ago      Up 8 seconds (health: starting)   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:32974->1935/tcp, :::32974->1935/tcp
```

* 未だ起動途中のコンテナがある場合
```
$ docker compose ps
NAME                IMAGE                        COMMAND                  SERVICE             CREATED             STATUS                             PORTS
camera_c110         qiita_softark2-camera_c110   "/bin/sh -c 'exec /h…"   camera_c110         30 seconds ago      Up 28 seconds (healthy)            
camera_c200         qiita_softark2-camera_c200   "/bin/sh -c 'exec /h…"   camera_c200         30 seconds ago      Up 28 seconds (healthy)            
nginx_rtmp          qiita_softark2-nginx_rtmp    "nginx -g 'daemon of…"   nginx_rtmp          32 seconds ago      Up 29 seconds (health: starting)   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:32974->1935/tcp, :::32974->1935/tcp
```

* 最終的に、全てが`(healty)`となれば問題ありません
```
$ docker compose ps
NAME                IMAGE                        COMMAND                  SERVICE             CREATED             STATUS                    PORTS
camera_c110         qiita_softark2-camera_c110   "/bin/sh -c 'exec /h…"   camera_c110         41 seconds ago      Up 39 seconds (healthy)  
camera_c200         qiita_softark2-camera_c200   "/bin/sh -c 'exec /h…"   camera_c200         41 seconds ago      Up 38 seconds (healthy)  
nginx_rtmp          qiita_softark2-nginx_rtmp    "nginx -g 'daemon of…"   nginx_rtmp          43 seconds ago      Up 39 seconds (healthy)   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:32974->1935/tcp, :::32974->1935/tcp
```

## ログの確認

* 正常に動作していれば、c110用のffmpeg、c200用のffmpegから下記のようにようなログが出力（エラーが発生していない）されているはずです。
```
$ docker compose logs
camera_c200  | ffmpeg version 4.1.10 Copyright (c) 2000-2022 the FFmpeg developers
camera_c200  |   built with gcc 10.2.1 (Alpine 10.2.1_pre1) 20201203
camera_c200  |   configuration: --disable-debug --disable-doc --disable-ffplay --enable-avresample --enable-fontconfig --enable-gpl --enable-libaom --enable-libass --enable-libbluray --enable-libfdk_aac --enable-libfreetype --enable-libkvazaar --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libopus --enable-libsrt --enable-libtheora --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxcb --enable-libxvid --enable-libzmq --enable-nonfree --enable-openssl --enable-postproc --enable-shared --enable-small --enable-version3 --extra-cflags=-I/opt/ffmpeg/include --extra-ldflags=-L/opt/ffmpeg/lib --extra-libs=-ldl --extra-libs=-lpthread --prefix=/opt/ffmpeg
camera_c200  |   libavutil      56. 22.100 / 56. 22.100
camera_c200  |   libavcodec     58. 35.100 / 58. 35.100
camera_c200  |   libavformat    58. 20.100 / 58. 20.100
camera_c200  |   libavdevice    58.  5.100 / 58.  5.100
camera_c200  |   libavfilter     7. 40.101 /  7. 40.101
camera_c200  |   libavresample   4.  0.  0 /  4.  0.  0
camera_c200  |   libswscale      5.  3.100 /  5.  3.100
camera_c200  |   libswresample   3.  3.100 /  3.  3.100
camera_c200  |   libpostproc    55.  3.100 / 55.  3.100
camera_c200  | Guessed Channel Layout for Input Stream #0.1 : mono
camera_c200  | Input #0, rtsp, from 'rtsp://lkjhgfdsa:1234567890@192.168.0.101:554/stream2':
camera_c200  |   Metadata:
camera_c200  |     title           : Session streamed by "TP-LINK RTSP Server"
camera_c200  |     comment         : stream2
camera_c200  |   Duration: N/A, start: 0.000000, bitrate: N/A
camera_c200  |     Stream #0:0: Video: h264, yuv420p(tv, bt709, progressive), 640x360, 15 fps, 16.67 tbr, 90k tbn, 30 tbc
camera_c200  |     Stream #0:1: Audio: pcm_alaw, 8000 Hz, mono, s16, 64 kb/s
camera_c200  | Stream mapping:
camera_c200  |   Stream #0:0 -> #0:0 (h264 (native) -> h264 (libx264))
camera_c200  |   Stream #0:1 -> #0:1 (pcm_alaw (native) -> aac (native))
camera_c200  | Press [q] to stop, [?] for help
camera_c200  | [aac @ 0x7fe569a48040] Too many bits 6408.707483 > 6144 per frame requested, clamping to max
camera_c200  | [libx264 @ 0x7fe569a4a040] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 AVX2 LZCNT BMI2
camera_c200  | [libx264 @ 0x7fe569a4a040] profile High, level 2.2
camera_c200  | [libx264 @ 0x7fe569a4a040] 264 - core 148 - H.264/MPEG-4 AVC codec - Copyleft 2003-2016 - http://www.videolan.org/x264.html - options: cabac=1 ref=3 deblock=1:0:0 analyse=0x3:0x113 me=hex subme=7 psy=1 psy_rd=1.00:0.00 mixed_ref=1 me_range=16 chroma_me=1 trellis=1 8x8dct=1 cqm=0 deadzone=21,11 fast_pskip=1 chroma_qp_offset=-2 threads=11 lookahead_threads=1 sliced_threads=0 nr=0 decimate=1 interlaced=0 bluray_compat=0 constrained_intra=0 bframes=3 b_pyramid=2 b_adapt=1 b_bias=0 direct=1 weightb=1 open_gop=0 weightp=2 keyint=30 keyint_min=3 scenecut=0 intra_refresh=0 rc_lookahead=30 rc=abr mbtree=1 bitrate=512 ratetol=1.0 qcomp=0.60 qpmin=0 qpmax=69 qpstep=4 ip_ratio=1.40 aq=1:1.00
camera_c200  | Output #0, flv, to 'rtmp://nginx_rtmp:1935/livecam/c200':
camera_c200  |   Metadata:
camera_c200  |     title           : Session streamed by "TP-LINK RTSP Server"
camera_c200  |     comment         : stream2
camera_c200  |     encoder         : Lavf58.20.100
camera_c200  |     Stream #0:0: Video: h264 (libx264) ([7][0][0][0] / 0x0007), yuv420p, 640x360, q=-1--1, 512 kb/s, 15 fps, 1k tbn, 15 tbc
camera_c200  |     Metadata:
camera_c200  |       encoder         : Lavc58.35.100 libx264
camera_c200  |     Side data:
camera_c200  |       cpb: bitrate max/min/avg: 0/0/512000 buffer size: 0 vbv_delay: -1
camera_c200  |     Stream #0:1: Audio: aac ([10][0][0][0] / 0x000A), 11025 Hz, mono, fltp, 66 kb/s
camera_c200  |     Metadata:
camera_c200  |       encoder         : Lavc58.35.100 aac
camera_c110  | ffmpeg version 4.1.10 Copyright (c) 2000-2022 the FFmpeg developers
camera_c110  |   built with gcc 10.2.1 (Alpine 10.2.1_pre1) 20201203
camera_c110  |   configuration: --disable-debug --disable-doc --disable-ffplay --enable-avresample --enable-fontconfig --enable-gpl --enable-libaom --enable-libass --enable-libbluray --enable-libfdk_aac --enable-libfreetype --enable-libkvazaar --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libopus --enable-libsrt --enable-libtheora --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxcb --enable-libxvid --enable-libzmq --enable-nonfree --enable-openssl --enable-postproc --enable-shared --enable-small --enable-version3 --extra-cflags=-I/opt/ffmpeg/include --extra-ldflags=-L/opt/ffmpeg/lib --extra-libs=-ldl --extra-libs=-lpthread --prefix=/opt/ffmpeg
camera_c110  |   libavutil      56. 22.100 / 56. 22.100
camera_c110  |   libavcodec     58. 35.100 / 58. 35.100
camera_c110  |   libavformat    58. 20.100 / 58. 20.100
camera_c110  |   libavdevice    58.  5.100 / 58.  5.100
camera_c110  |   libavfilter     7. 40.101 /  7. 40.101
camera_c110  |   libavresample   4.  0.  0 /  4.  0.  0
camera_c110  |   libswscale      5.  3.100 /  5.  3.100
camera_c110  |   libswresample   3.  3.100 /  3.  3.100
camera_c110  |   libpostproc    55.  3.100 / 55.  3.100
camera_c110  | Guessed Channel Layout for Input Stream #0.1 : mono
camera_c110  | Input #0, rtsp, from 'rtsp://asdfghjkl:1234567890@192.168.0.120:554/stream2':
camera_c110  |   Metadata:
camera_c110  |     title           : Session streamed by "TP-LINK RTSP Server"
camera_c110  |     comment         : stream2
camera_c110  |   Duration: N/A, start: 0.000000, bitrate: N/A
camera_c110  |     Stream #0:0: Video: h264, yuv420p(progressive), 640x360, 15 fps, 19.92 tbr, 90k tbn, 30 tbc
camera_c110  |     Stream #0:1: Audio: pcm_alaw, 8000 Hz, mono, s16, 64 kb/s
camera_c110  | Stream mapping:
camera_c110  |   Stream #0:0 -> #0:0 (h264 (native) -> h264 (libx264))
camera_c110  |   Stream #0:1 -> #0:1 (pcm_alaw (native) -> aac (native))
camera_c110  | Press [q] to stop, [?] for help
camera_c110  | [aac @ 0x7f1466bbe940] Too many bits 6408.707483 > 6144 per frame requested, clamping to max
[libx264 @ 0x7f1466bc0b40] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 AVX2 LZCNT BMI2peed=N/A    
camera_c110  | [libx264 @ 0x7f1466bc0b40] profile High, level 2.2
camera_c110  | [libx264 @ 0x7f1466bc0b40] 264 - core 148 - H.264/MPEG-4 AVC codec - Copyleft 2003-2016 - http://www.videolan.org/x264.html - options: cabac=1 ref=3 deblock=1:0:0 analyse=0x3:0x113 me=hex subme=7 psy=1 psy_rd=1.00:0.00 mixed_ref=1 me_range=16 chroma_me=1 trellis=1 8x8dct=1 cqm=0 deadzone=21,11 fast_pskip=1 chroma_qp_offset=-2 threads=11 lookahead_threads=1 sliced_threads=0 nr=0 decimate=1 interlaced=0 bluray_compat=0 constrained_intra=0 bframes=3 b_pyramid=2 b_adapt=1 b_bias=0 direct=1 weightb=1 open_gop=0 weightp=2 keyint=30 keyint_min=3 scenecut=0 intra_refresh=0 rc_lookahead=30 rc=abr mbtree=1 bitrate=512 ratetol=1.0 qcomp=0.60 qpmin=0 qpmax=69 qpstep=4 ip_ratio=1.40 aq=1:1.00
camera_c110  | Output #0, flv, to 'rtmp://nginx_rtmp:1935/livecam/c110':
camera_c110  |   Metadata:
camera_c110  |     title           : Session streamed by "TP-LINK RTSP Server"
camera_c110  |     comment         : stream2
camera_c110  |     encoder         : Lavf58.20.100
camera_c110  |     Stream #0:0: Video: h264 (libx264) ([7][0][0][0] / 0x0007), yuv420p, 640x360, q=-1--1, 512 kb/s, 15 fps, 1k tbn, 15 tbc
camera_c110  |     Metadata:
camera_c110  |       encoder         : Lavc58.35.100 libx264
camera_c110  |     Side data:
camera_c110  |       cpb: bitrate max/min/avg: 0/0/512000 buffer size: 0 vbv_delay: -1
camera_c110  |     Stream #0:1: Audio: aac ([10][0][0][0] / 0x000A), 11025 Hz, mono, fltp, 66 kb/s
camera_c110  |     Metadata:
camera_c110  |       encoder         : Lavc58.35.100 aac
```

## 実行中のリソース使用状況

* `docker stats`コマンドでリソースの使用状況を確認してみました。なお、約20時間動作させた状態です。
* PCのCPUは Intel(R) Core(TM) i7-4790 CPU @ 3.60GHz で、4コアです。
* h264, 640x360, 15 fpsの映像をflvに変換するのに、CPUを20%程度使用していました。

```
$ docker stats
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O     PIDS
dd0bed613de5   nginx_rtmp    0.40%     10.31MiB / 15.56GiB   0.06%     9.41GB / 450MB    0B / 90.5MB   10
74d70eef0d06   camera_c110   18.04%    95.66MiB / 15.56GiB   0.60%     1.91GB / 4.74GB   0B / 0B       38
cbe2c71723d8   camera_c200   19.01%    95MiB / 15.56GiB      0.60%     1.69GB / 4.82GB   0B / 0B       38
```
***

# 参考サイト

今回Qiitaの@softarkさんの記事[IPカメラの動画をウェブ・ページで公開する](https://qiita.com/softark/items/dca866616d441e0d6fd8)を参考に、dockerコンテナ化を行いました。その他にも下記のサイトを参考にさせていただきました。ありがとうございました。

## "ip camerra nginx 配信"で検索して出てくる記事

* [IPカメラの動画をウェブ・ページで公開する](https://qiita.com/softark/items/dca866616d441e0d6fd8) : Qiita [@softark](https://qiita.com/softark)
* [動画配信サーバ構築（nginx+nginx-rtmp-module）](https://centossrv.com/nginx-nginx-rtmp-module.shtml)
* [nginx + nginx-rtmp-module で RTMP(S) to HLS 配信](https://notes.yh1224.com/tech/nginx-rtmp-hls/)
* [【2022年度版】今でも動くライブ配信システムの構築（rtmp、hls対応）とブラウザ視聴対応について](https://note.com/educator/n/nea0f4663a5ba)

## nginx-rtmp-moduleとその設定について

* [NGINX-based Media Streaming Server](https://github.com/arut/nginx-rtmp-module) : GitHub [@arut](https://github.com/arut)
* [Enabling Video Streaming for Remote Learning with NGINX and NGINX Plus](https://www.nginx.com/blog/video-streaming-for-remote-learning-with-nginx/) : NGINX Blog by [Nina Forsyth of F5](https://www.nginx.com/people/nina-forsyth/)

## ブラウザでhls再生に使用したVideo.jsについて

* [VIDEO JS - Getting Started](https://videojs.com/getting-started/)

##  TP-LinkのサポートFAQ

* [Tapoを使用したRTSPライブストリーミングの利用方法](https://www.tp-link.com/jp/support/faq/2680/)


