---
layout: post
title: Github Pagesでのテーマ上書きでハマった
date: 2018-02-13 11:01:56 UTC+9
tags: Jekyll GithubPages
---
ポストページに前・次のポストへのリンクやそのポストの編集履歴へのリンクを付けるのに、FontAwesomeのアイコンを使いたくていろいろいじっていたんですが、_includes/head.htmlを置き換えてローカルのJekyllで動かしても問題ないのに、Github側で`build failed`になって困っていました。

結論から言ってしまえば、_config.ymlで`jekyll-seo-tag`プラグインが有効化されていないのが原因でした。

使っているテーマのminimaのファイルをそのまま持ってきても動かないし、意味わからんとか思いつついろいろ試していて、head.htmlの上書きではなく、_layouts/default.htmlを上書きするようにしたら、Githubが「liquidにseoとかいうタグは使えないよー」とか言い出したので、気が付くことができました。

```diff
 theme: minima
 plugins:
   - jekyll-feed
+  - jekyll-seo-tag
```

という風に pluginsにjekyll-seo-tagを追加してあげるか、_includes/head.htmlから

```
...
  {% raw %}{% seo %}{% endraw %}
...
```

の行を削除してあげれば_includes/head.htmlを上書きできるようになります。


minimaのテーマそのままならプラグインを追加してなくてもビルドが通るので、よくわかりませんが、とりあえず解決です。

Githubのエラーメッセージが全然情報くれないのも困ります。`_includes`のエラーだとメッセージが出ないとかなんでしょうか。
