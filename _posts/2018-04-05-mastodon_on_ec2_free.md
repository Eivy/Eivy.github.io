---
layout: post
date: 2018-04-05T14:58:05 UTC+9
title: AmazonLinux2上でマストドン
tags: aws mastodon AmazonLinux2
---
弊社、クラウドを積極的に利用していきたいそうです。

でもなにをやるのかは一切決まっていないという典型的なダメパターン・・・。

とりあえずAWSのアカウント作ってEC2インスタンス立ててみたんですけど、何をするのかは決まっていないので、とりあえずmastodon動かしてみるかーとやってみました。
mastodonはDocker使ってさらっと動かせたりしますが、今回はAWSの運用方法の調査もあるので、Dockerを使わず、かつ極力AWSのマネージドサービスを利用します。あと無料枠で済ませる。

## AWSのサービス

まずはmastodonを動かすのに使うAWSのサービスを有効化しましょう。

とりあえず必要なサービスは

- EC2
- RDB(PostgreSQL)
- ElasticCache(redis)
- S3

です。

今回はEC2のOSにAmazon Linux 2を使います。
実際のところ、Ubuntuのほうが楽かもしれません・・・。[mastodonのドキュメント](https://github.com/tootsuite/documentation/)もUbuntu向けに書いてあるし。

### EC2

作成から Amazon Linux 2 の無用枠対象を選びましょう。

### RDB

PostgreSQLを選んで無料枠で作成します。AuroraでもPostgreSQL互換のものが動くみたいですが、今回は無料にこだわります。

インスタンスのセキュリティーグループで、インバウンドにルールを追加して、EC2から接続できるようにしておきます。

タイプ | プロトコル | ポート範囲 | ソース
---|---|---|---
カスタムTCPルール | TCP | 5432(デフォルトの場合) | VPCのIPアドレス

### ElastiCache

t2.microを選んで、レプリケーションは0にしましょう。無料枠はt2.micro 750時間/月なので、レプリケーションでノード増やすと超えちゃう・・・。

こっちもEC2から接続できるようにします。

VPCのセキュリティーグループの中にグループ名が'default'というのがいるはずなので、以下を追加します。

タイプ | プロトコル | ポート範囲 | ソース
---|---|---|---
カスタムTCPルール | TCP | 6379(デフォルトの場合) | 0.0.0.0/0

### S3

バケット名、files.~とかするらしいです。とりあえず、ドメインすらないので、EC2に割り当てられたパブリックDNS名を使いました。

パブリックDNS名ってインスタンスの再起動ごとに新しいのが割り当てられるみたいなので、マネしないでね。

こいつはユーザーからアクセスできるように設定します。この辺のことは[Mastodon の画像などメディアデータをAmazon S3に移行する（非Docker環境）](http://takanory.hatenablog.jp/entry/2017/05/10/070000)を参考にしました。

## 環境を整える

AWSのサービスの準備ができたら、mastodonを動かすのにEC2のインスタンスに接続して環境を整えましょう。

`mastodon`ユーザーを作って作業しましょう。

```sh
$ sudo useradd mastodon
$ sudo passwd mastodon
$ su - mastodon
```

とりあえずは`yum`で取れるものは取ってきます。

```sh
$ sudo yum groups install "Development tools"
$ sudo yum install -y git ImageMagick
```

nginxは`amazon-linux-extras`でインストールします。

```sh
$ sudo amazon-linux-extras install nginx1.12
```

ffmpegも必要なので、yumリポジトリを登録してからインストール。

```sh
$ yum -y install epel-release
$ rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
$ rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
$ sudo yum -y install ffmpeg
```

yarnも入れます。(つまりNode.jsもいる)

```sh
$ curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
$ curl --silent --location https://rpm.nodesource.com/setup_6.x | sudo bash -
$ sudo yum install -y yarn
```

rubyとbundleもいります。rbenvでrubyを入れましょう。

```sh
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
$ cd ~/.rbenv && src/configure && make -C src
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ git clone https://github.com/rbenv/ruby-build ~/.rbenv/plugins/ruby-build
$ rbenv install 2.5.0
$ rbenv global 2.5.0
$ gem install bundler
```

最後に`bundle install`で必要な依存関係をインストール。

```sh
$ sudo yum install -y libicu-devel libidn-devel postgresql-devel protobuf-devel
```

## いざMastodonのインストール

Mastodnをインストールしていきましょう。

gitリポジトリの取得します。

```sh
$ git clone https://github.com/tootsuite/mastodon ~/live
```

gitリポジトリに移動して最新のタグをチェックアウトしましょう。

```sh
$ cd ~/live/
$ git tag #tagを確認
$ git checkout v2.3.3
```

チェックアウトが終わったら依存関係をインストール。

```sh
$ gem install bundler
$ bundle install -j$(getconf _NPROCESSORS_ONLN) --deployment --without test development
$ yarn install --pure-lockfile
```

あとは対話式で環境設定ファイルを作ります。いろいろ聞かれるので答えていきましょう。

```sh
$ RAILS_ENV=production bundle exec rake mastodon:setup
```

これが終わればもうMastodonが動かせるようになっています。

## Webサーバーとサービス化

ではMastodonを動かしてアクセスするために、Webサーバーを設定しましょう。

`/etc/nginx/nginx.conf`を以下のように変更します。変更するには管理者権限が必要です。

```nginx
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

server {
  listen 80;
  listen [::]:80;
  server_name example.com;
  root /home/mastodon/live/public;
  # Useful for Let's Encrypt
  location /.well-known/acme-challenge/ { allow all; }
  location / { return 301 https://$host$request_uri; }
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name example.com;

  ssl_protocols TLSv1.2;
  ssl_ciphers HIGH:!MEDIUM:!LOW:!aNULL:!NULL:!SHA;
  ssl_prefer_server_ciphers on;
  ssl_session_cache shared:SSL:10m;

  ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  keepalive_timeout    70;
  sendfile             on;
  client_max_body_size 8m;

  root /home/mastodon/live/public;

  gzip on;
  gzip_disable "msie6";
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_buffers 16 8k;
  gzip_http_version 1.1;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

  add_header Strict-Transport-Security "max-age=31536000";

  location / {
    try_files $uri @proxy;
  }

  location ~ ^/(emoji|packs|system/accounts/avatars|system/media_attachments/files) {
    add_header Cache-Control "public, max-age=31536000, immutable";
    try_files $uri @proxy;
  }

  location /sw.js {
    add_header Cache-Control "public, max-age=0";
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Proxy "";
    proxy_pass_header Server;

    proxy_pass http://127.0.0.1:3000;
    proxy_buffering off;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    tcp_nodelay on;
  }

  location /api/v1/streaming {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Proxy "";

    proxy_pass http://127.0.0.1:4000;
    proxy_buffering off;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    tcp_nodelay on;
  }

  error_page 500 501 502 503 504 /500.html;
}
```

今度はMastodonをサービスとして動かすために3つファイルを用意します。

`/etc/systemd/system/mastodon-web.service`
```ini
[Unit]
Description=mastodon-web
After=network.target

[Service]
Type=simple
User=mastodon
WorkingDirectory=/home/mastodon/live
Environment="RAILS_ENV=production"
Environment="PORT=3000"
ExecStart=/home/mastodon/.rbenv/shims/bundle exec puma -C config/puma.rb
ExecReload=/bin/kill -SIGUSR1 $MAINPID
TimeoutSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/mastodon-sidekiq.service`
```ini
[Unit]
Description=mastodon-sidekiq
After=network.target

[Service]
Type=simple
User=mastodon
WorkingDirectory=/home/mastodon/live
Environment="RAILS_ENV=production"
Environment="DB_POOL=5"
ExecStart=/home/mastodon/.rbenv/shims/bundle exec sidekiq -c 5 -q default -q mailers -q pull -q push
TimeoutSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/mastodon-streaming.service`
```ini
[Unit]
Description=mastodon-streaming
After=network.target

[Service]
Type=simple
User=mastodon
WorkingDirectory=/home/mastodon/live
Environment="NODE_ENV=production"
Environment="PORT=4000"
ExecStart=/usr/bin/npm run start
TimeoutSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

ファイルができたらsystemdに更新を伝えます。

```sh
$ sudo systemctl daemon-reload
```

これでインストールは完了です。


## 動かす

では動かしてみましょう。

```sh
$ sudo systemctl start nginx mastodon-web mastodon-sidekiq mastodon-streaming
```

ブラウザからEC2のパブリックDNSにアクセスすると、Mastodonが表示されているはずです。

![Mastodon](/images/2018-04-04-mastodon.jpg)

## 最後に

今回はAWSの無料枠でMastodonを動かしてみましたが、AWSのアカウント作成から無料枠は1年で終わるので、その後はお金がかかるようになりますので、ご注意ください。
