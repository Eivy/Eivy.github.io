---
layout: post
title: Xamarin.FormsでiOSフレームワークを参照するやり方を毎回忘れるのでメモ
date: 2022-03-15 11:40:00 UTC+9
---
表題の通り、毎回忘れてやってつまづいているのでメモ。

何はともあれ、ネイティブのライブラリを作成します。

XcodeでFrameworkプロジェクトを作成、Swift Fileを追加して以下のようなコードを書きます。

```swift
import UIKit

@objc(ClassName)
public class ClassName : NSObject {
  @objc
  public func DoSomething() -> Int {
    return 1
  }
}
```

ここでの注意としてはNSObjectがベースのクラスでないとXamarin.Formsの方からアクセスできないこと。以前にSwiftUIのコンポーネントをXamarin.Formsから使えないか試したけど、だめでした。
あと公開するメソッドをpublicにして`@objc`で修飾しておきます。

これをビルドしたら、次はVisual StudioでiOSのバインディングライブラリを作成します。

そして[Objective Sharpie](https://docs.microsoft.com/ja-jp/xamarin/cross-platform/macios/binding/objective-sharpie/get-started)(目標マジックペン？)を使って、C#のコードを作成します。
コマンドは以下の形式。

```sh
sharpie bind --output=OutputDir --namespace=csharp_name_space --sdk=iphoneos15.2 -framework ./created.framework
```

> `--sdk=`のところに何が使えるのかを確認するには以下コマンド。
> 
> ```sh
> sharpie xcode -sdks
> ```

コマンドを実行して上手くいってるとと `--output` で指定したディレクトリ以下に `ApiDefinitions.cs` が出力されているはず。場合によっては `Structs.cs` みたいなのもある。

出力された `.cs` ファイルには `[Verify]` と修飾されているものがあるので、内容を確認して問題なければ `[Verify]` 属性を削除する。

バインディングライブラリに `ApiDefinitions.cs` を追加して、ファイルのビルドアクションを `ObjcBindingApiDefinition` に変更する。( `Structs.cs` の場合は `ObjcBindingCoreSource` にする)

あとはビルドが通るように多少修正が必要かもしれない。なんか対応してないコードが吐かれたりすることがあるみたい。

---

今までに思い出せなくてつまづいたところは

* sharpieの引数で、framework用のやつとstatic library用のやつを混同してた
* ファイルのビルドアクションを変更し忘れてた
* Swiftのコードで `@objc` つけ忘れてた

というのをやらかしている・・・。
