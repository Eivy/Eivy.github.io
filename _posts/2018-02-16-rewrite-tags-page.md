---
layout: post
date: 2018-02-16 12:00:00 UTC+9
title: JekyllでTagsページを実装
tags: Jekyll
---
[タグページ]({{ "/tags.html" | relative_url }})を書き換えました。

タイトルで実装とか言いながら書き換えとか言ってますが、書き換えました。
元々用意はしていたんですが、タグの一覧としか機能していないのが気に入らなくて、やっぱり指定したタグのポストだけが出てくるページが欲しいなーと。

ただ、このサイトはGithub PagesのJekyllで出力されているので、指定されたタグページを動的に出力するとか、それぞれのタグ用のページを自動で作っておくとかいうことができません。
なので今回は[Vue.js](https://jp.vuejs.org/)を使って、ブラウザ側で動的にページを作ることにしました。

やっていることは単純で、

1. Jekyllでタグごとのポスト情報をJSONオブジェクトでHTMLに埋め込む
2. 1のJSONをVueに渡してレンダリング

です。

1のJSONを吐き出すところが以下。
{% raw %}
```liquid
var data = {
  {%- assign last_tag = site.tags | last -%}
  {% for tag in site.tags %}
    "{{ tag[0] }}": {
      "page": [
        {%- for page in tag[1] -%}
        {"url": "{{ page.url }}", "title": "{{ page.title }}"}
        {%- unless forloop.last -%},{%- endunless -%}
        {%- endfor -%}
      ]
    }
  {%- unless forloop.last -%},{%- endunless -%}
  {%- endfor -%}
};
```
{% endraw %}

最初は
{% raw %} `{{ site.tags | jsonify }}` {% endraw %}
で渡せばいいかなーと思ってたんですが、HTMLが含まれてるし、いらない情報も多いしで、上のようになりました。

2の部分は本当に単純でVue入門って感じ。
展開元のノード指定とデータ、展開するテンプレートを渡してあげるだけ。

{% raw %}
```js
    new Vue({
      el: '#tags',
      data,
      template: '<div><div v-for="(t, v) in this.$data"><h3><i class="fas fa-tag"></i> {{v}}</h3><div v-for="p in t.page"><a :href=p.url>{{p.title}}</a></div></div></div>'
    });
```
{% endraw %}

ちなみにテンプレート部分はこうなってます。

{% raw %}
```html
<div>
  <div v-for="(t, v) in this.$data">
    <h3><i class="fas fa-tag"></i> {{v}}</h3>
    <div v-for="p in t.page"><a :href=p.url>{{p.title}}</a></div>
  </div>
</div>
```
{% endraw %}

HTMLにタグ関連の情報が全部入っているので、ポストが増えていったときにタグページが重い、なんてことになりそうなので、あんまりいい方法ではないんですけど、コード書いたので満足です。

一番いいのは、タグごとのページを出力するようなプラグイン書いてCI回してビルド、デプロイという形じゃないかなーと思います。
それならポストの原稿をコミット・プッシュすればサイトが更新されるって状況も維持できるし。

まあ、追々やろうかな。
