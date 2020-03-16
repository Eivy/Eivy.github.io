---
layout: post
title: Reactの勉強がてらNatureRemoのリモコンウェブアプリ作った
date: 2020-03-16T19:55:12 UTC+0900
tags: React NatureRemo TypeScript
image: https://upload.wikimedia.org/wikipedia/commons/thumb/a/a7/React-icon.svg/1200px-React-icon.svg.png
---

Reactの勉強がてら、[NatureRemo]()を操作するウェブアプリを作ってみました。

<div class="github-card" data-github="Eivy/natureremo-webapp" data-width="400" data-height="" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

去年の5月にNatureRemoを買ってからハマってしまって、物理ボタンが欲しくなって電子工作を始めてみたりしてたんですが、
この度アプリのほうにまで手を出してみました。

作ったものは [https://eivy.github.io/natureremo-webapp/](https://eivy.github.io/natureremo-webapp/ ) で動くようになっています。

今回のものを作るにあたって、ざっくり言うと以下のライブラリを使っています。

- React
- React-Redux
- React-Router
- i18next
- node-sass

Reactは本家NatureさんのアプリがReact Native製ということなので、とりあえず使ってみました。

Vue.jsは一応使ったことがあって、大まかな感じだとそんなに変わらないのかなぁという感じでした。
TypeScriptとの組み合わせはVue.jsよりも良さそうです。typescript-language-serverを使った補完とか。
Vue.jsを使ったのも2年近く前なので、今はVue.jsのほうも補完がいい感じになってるかもしれませんが。

とりあえず今回はコードがどうとかより、SVGとCSSで悩んでる時間が長かったような気がします。あとテスト。

そもそも自動テストを書く文化がある会社で働いたことないんだよ・・・。どう書いたらいいのかもよくわかってないんだよ・・・。
そのあたりのPRとか来たら嬉しいんだけど、誰かレビューとかしてくれないかな。

とりあえず、まだアイコンが無いところがあったり、照明とテレビのボタン一覧の画面がボタン並べているだけだったりするので、まだしばらくは直していくつもりです。
あとはエアコンに学習機能でボタン増やせるようにしたりとか。一応、API使えばできるっぽいし。

UIのアイディアとかあればPRお待ちしています。

正直、これを頑張って良くするより、Natureに潜り込んだほうが早い・・・?
