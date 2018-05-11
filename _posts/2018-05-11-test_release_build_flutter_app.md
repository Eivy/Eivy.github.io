---
layout: post
date: 2018-05-11T17:31:46 UTC+9
title: Flutterアプリのリリースビルドはシミュレータで動かない
tags: flutter android
---
本当は動かなくもないです。

`flutter build apk` で作ったapkはx86では動かないらしいです。

私は実機持ってないので、Webでダウンロードしてインストールできるのか試したかったんですが、できなくて1日悩んでしまいました。

`INSTALL_FAILED_NO_MACHING_ABIS` とか出てたのにね。

というわけで、シミュレータにリリースビルドのapkを入れて動かしたいときはARMのイメージを動かしましょう。
すごく遅いですけど。

やっぱり実機がいる・・・。
