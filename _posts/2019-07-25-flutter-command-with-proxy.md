---
layout: post
title: Flutterコマンドのほうでproxyを通す
date: 2019-07-25T13:07:50 UTC+9
tags: flutter proxy
---

[FlutterアプリでProxyを使う](./2018-12-21-flutter-proxy.md) のほうにアクセスがちょっとあるようなのですが、もしかしたら`flutter`コマンドでproxyを突破したいのではないかなーということで書き記しておきます。

環境変数`http_proxy`と`https_proxy`を設定したら`flutter`コマンドがそれを読み取ってよしなにやってくれます。

Unix系のターミナルから実行するなら

```sh
http_proxy=http://proxy-server.co.jp:8080 https_proxy=http://proxy-server.co.jp flutter packages get
```

みたいな。

Windowsのコマンドプロンプトでやるなら

```cmd
SET http_proxy=http://proxy-server.co.jp
SET https_proxy=http://proxy-server.co.jp
flutter packages get
```

ですかね?  
Windowsのほうは環境変数なくてもネットワーク設定みてくれてる?

---

いつのまにか、FlutterでWebアプリやらDesktopアプリやら更には組み込み向けのものまで作れるようになってるんですね。すごいなー。
