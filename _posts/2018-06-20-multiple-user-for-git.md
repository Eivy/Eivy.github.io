---
layout: post
date: 2018-06-20T18:22:25 UTC+9
title: gitでURLごとに認証ユーザーを変える
---
会社と個人のアカウントが違うし、プライベートリポジトリがあるんだけど、どっちも同じPC内で作業したい場合なんかに。

`~/.git/config`に

```text
[url "https://CompanyAccount@github.com/Company/"]
insteadOf="github.com/Company/"

[url "https://Eivy@github.com/Eivy/"]
insteadOf="github.com/Eivy/"
```

という感じでURLのinteadOfを使うように書いてしまえばなんとかなりました。

ついでにcommitterとかもユーザーのconfigファイルで切り替えたいけど、よくわからない。
リポジトリ内の`.git/config`に書いちゃうしかないのかな?
