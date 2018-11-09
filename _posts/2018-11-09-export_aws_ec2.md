---
layout: post
date: 2018-11-09T11:14:35 UTC+9
title: AWSのEC2をVMにExport
tags: aws ec2 virtualbox vmware
---
EC2でとりあえず動かしたインスタンスをVMにしたいと思ったことはありませんか?

私の場合は無料枠で動かしてたWindowsがいらなくなって、データは残しておきたいけど、EBSにおいたままだとお金掛かるしなーということで
ローカルに吸い出したいというのがことの始まりです。

AWSにVM Import/Exportというのがあるので、「これ使えばいいじゃん」ってなってたんですが、[ドキュメント](https://docs.aws.amazon.com/ja_jp/vm-import/latest/userguide/vmexport.html#vmexport-limits)には

> 以前に別の仮想化環境から Amazon EC2 にインポートしたインスタンスでない限り、Amazon EC2 からインスタンスをエクスポートすることはできません。

とあって、ゼロからEC2で構築したものはエクスポートできません。


しょうがないのでいろいろ調べたんですが、どうやらVirtualBox付属の`vboxmanage`でISOをvdiなりvdmxなりに変換できることがわかったので、`dd`コマンドでイメージにして、`vboxmanage convertfromraw`で変換することにしました。

エクスポートしたいEC2インスタンスについてたEBSボリュームを適当なLinuxインスタンスに付け替えて、`dd`で吸い上げます。イメージの保存先は十分サイズがあるEBSをつないだインスタンスならいいのですが、私は無料枠のデフォルトで動かしているやつに付け替えたのでサイズが足りず、ローカルからssh経由で吸い出しました。

```sh
ssh $aws_instanse 'dd if=/dev/xvdf' | dd of=aws_instance.iso
```

ちなみにこれだとEBSのサイズそのままがデータとして流れるので、通信量のほうで課金されます。私はよくわかってないので、そのままやっちゃいましたが、gzipとか挟んでデータ転送量を減らすようにしましょう。


んで、できたisoを以下のコマンドで変換します。

```sh
vboxmanage convertfromraw aws_instance.iso aws_instance.vdi # 拡張子をvmdkにすればVMWare形式にもできる
```

あとはできたVDIを仮想マシンに割り当てればローカルで動かせるようになりました。


最初からVMをインポートしてればいいんですけど、なかなか先が見通せないというか、そもそもあんまりEC2をエクスポートしたくなる場面とか想像できないというか。

Windowsをエクスポートするとライセンスの問題とかあると思うので、気をつけてください。
