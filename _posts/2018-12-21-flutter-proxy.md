---
layout: post
title: FlutterでProxyを使う
tags: Flutter
date: 2018-12-21T12:55:49 UTC+9
---
Flutterでアプリを書いていて、プロキシ環境下だと通信に失敗するので、いろいろ調べました。


API叩くとかだと、

```dart
var client = new HttpClient();
client.findProxy = (Uri u) => 'PROXY proxyserver:8080';
client.getUrl(Uri.https('example.com', '/api/hoge')).then((request) {
  request.close().then((response) {
    // do something
  });
});
```

とやれば、プロキシを超えられるようになります。

ただ、HttpClientで書くと長いし面倒臭いので、[http](https://pub.dartlang.org/packages/http)使いたい。
あと、これだとImage.networkで画像が表示できません。

アプリ全体でhttp通信にプロキシを使用したい場合は `findProxyFromEnvironment`を上書きしたものをHttpOverrides.globalに設定します。

```dart
import 'dart:io';

void main() async {
  var httpOverrides = new MyHttpOverrides();
  HttpOverrides.global = httpOverrides;
  runApp(MyApp());
}

class MyHttpOverrides extends HttpOverrides {
  @override
  String findProxyFromEnvironment() => "PROXY proxyserver:8080";
}
```

これでhttp使ってスッキリ書けるし、画像も表示されるぞ。ﾔｯﾀｰ\\(^o^)/

でも、

> プロキシサーバーって固定じゃだめよね?
> AndroidにしろiOSにしろWi-Fiとかに設定してるんだから、それ使ってよ

ってことになるわけです。

ググったけど、ネットワーク設定からプロキシ設定を取ってくれるライブラリとかなくてですね。
しょうがないので自分で作りました。

<div class="github-card" data-github="Eivy/get_proxy" data-width="400" data-height="" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

[connectivity](https://pub.dartlang.org/packages/connectivity)と組み合わせて使うといい感じになります。

`pubspec.yaml`に以下を追加して、

```yaml
dependencies:
  get_proxy:
    git: https://github.com/Eivy/get_proxy.git
  connectivity: ^0.3.2
```

以下のように使うとモバイル通信とWi-Fiが切り替わってもに通信できるようになります。

```dart
import 'dart:io';
import 'dart:async';
import 'package:get_proxy/get_proxy.dart';
import 'package:connectivity/connectivity.dart';

void main() async {
  var httpOverrides = new MyHttpOverrides();
  httpOverrides.init('https://example.com/');
  HttpOverrides.global = httpOverrides;
  runApp(MyApp());
}

class MyHttpOverrides extends HttpOverrides {
  String address = "";
  String type = "";

  @override
  String findProxyFromEnvironment(Uri uri, Map<String, String> environment) {
    if (type == "DIRECT") {
      return "DIRECT";
    }
    return  'PROXY $address';
  }

  Future init(String url) async {
    type = await GetProxy.proxyType(url);
    address = await GetProxy.proxyAddress(url);
    print(type);
    print(address);
    new Connectivity().onConnectivityChanged.listen((result) async {
      type = await GetProxy.proxyType(url);
      address = await GetProxy.proxyAddress(url);
      print(type);
      print(address);
    });
  }

}
```

ただ、これだとアドレスごとにプロキシサーバーが切り替わる環境だとうまく動かないし、
`get_proxy`のほうもPACでプロキシ設定している環境でうまく動くかわからないとかあるので、
コピペしても辛い目に合うかもしれないので気をつけて下さい。
