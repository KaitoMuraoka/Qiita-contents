---
title: 【Xcode】Break Pointでprintデバッグする方法
tags:
  - Xcode
  - iOS
  - debug
  - Swift
private: false
updated_at: '2023-08-06T17:58:20+09:00'
id: 540aea1539fcfb25b8ea
organization_url_name: null
slide: false
---

# 概要
開発や競技プログラミングをしている際、
- その変数が任意のタイミングでどのような値を持っているのか
- この関数はどのタイミングで処理されているのか
といったことが気になる場面が多々あると思います。

そこで登場するのが`print`関数です。これを使うことにより、コンソールやターミナル上に変数や文字列を表示することができます。しかし、この関数にはいくつかの欠点があります。

今回は`print`関数を使うことのデメリットとbreakpointを使ったスマートな方法を紹介します。


# printデバッグとは
上記にも述べたとおり、我々が最も知っているデバッグ方法が`print`デバッグでしょう。

JavaScriptなら`console.log`、JavaやKotlinなら`println`、PHPなら`echo`、Rubyなら`puts`がこれに当てはまります。

iOSの場合では`print`もしくは`NSLog`です。

これらの関数はプログラムのある時点でデータの流れや状態を追跡することで、エラーの原因や特定をするのに役立ちます。

## printデバッグの長所
print デバッグの最大の長所は**使いやすさ**です。
やり方は単純で、追跡したい場所に`print`を置くだけです。
このやり方は様々なプログラミング入門書で紹介されています。

## printデバッグの短所
printデバッグの短所はテスト後に多くの**後始末**をする必要がある点です。

printデバッグは一時的なものであり、通常はバグが解消された時点で削除されます。
これを怠ると、ランタイムが遅くなる原因になります。

XcodeにはFindという機能があり、そこで"print"と検索すれば探し出すことができますが、
時々、削除するのを忘れてしまい、レビュー時に指摘される場面が多々あると思います(実体験)。

これらの解決方法として、[`#if DEBUG`](https://zenn.dev/kyome/articles/0b6a689776c9c69b4026)を使うことで`print`関数を削除する方法などがあります。

```Swift:Sample.swift
    override func viewDidLoad() {
        super.viewDidLoad()
#if DEBUG
        print("Hello World")
#endif
        // 色々な処理
    }
```

しかし、これは不要なコードを隠しているだけで、根本的な解決にはなっていません。

:::note warn
処理内容のログを取得したい場合は、[`OSLog`](https://developer.apple.com/documentation/os/oslog)のようなツールを使用してください。
この記事では、一時的なバグを修正するために`print`関数を使うことについて解説します。
:::

また、printデバッグのもう1つの問題点は、新しい`print`関数を追加したい場合、アプリを再度buildする必要があります。




# Break Pointを使ったデバッグ方法

では、ここから本題のBreak Pointを使ったデバッグ方法について解説します。

この方法を使うと、見た目がスッキリして`print`デバッグの短所を克服することができます。

## 文字列を表示する方法
Break Pointを使って任意の文字列を表示させます。
方法は以下の通りです。

1. 自分がprintデバッグしたい箇所の行番号をダブルクリックする<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/da9d7e2e-1c29-5f28-ada5-52698cf3c397.png" width=60%>
2. 以下の画像のように、Break Point Editorを表示される<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/58ed6acc-356c-fa07-a31e-62baefe91333.png" width=60%>
3. **Add Action**ボタンをクリックし、**Log Message**を選択する。出力したい文字列をtextフィールドに入力する。
4. **Automatically continue after evaluating actions**をオンにする。これにより、このBreak Pointはデバッガが停止するのを防ぎます。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/8768e1d0-3c79-4a02-5514-ef89043b82b3.png" width=60%>

これで、Break Pointの行に達すると、コンソールにログメッセージが表示されます。
画像の場合だと`Hello World`と表示されます。


以上のように、必要なステップはとても少ないです。
そのため、`print`関数のように手軽に設置することが可能だと思います。

## 変数を表示する
次に変数を表示する方法を解説します。
方法は以下の通りです。

1. 自分がprintデバッグしたい箇所の行番号をダブルクリックする<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/b532fed4-2ce3-aa9d-c7ef-da9e6d92847a.png" width=60%>
2. 以下の画像のように、Break Point Editorを表示される<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/77f71558-d931-fea0-a52e-e230fd9f82b9.png" width=60%>
3. **Add Action**ボタンをクリックし、**Debugger Command**を選択する。出力したい任意のコマンドをテキストフィールドに入力します。`po print(hoge)`のように接頭辞を必ずつけてください
4. **Automatically continue after evaluating actions**をオンにする。これにより、このBreak Pointはデバッガが停止するのを防ぎます。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/13c145fe-78df-f38a-b2e5-b4d9492b47b0.png" width=60%>

これで、ブレークポイントの行に達すると、コンソールに変数が表示されます。
この場合は、`101`が表示されます。

## 長所
以上のように、Break Pointを使うことによってより多くの利点が得られます。

- あちらこちらに`print`関数がないので、後始末が不要：ブレークポイントを無効にするか、削除すればなくなります。
- 素早く動かすことができる：ドラックすることによって行を移動することが可能です。
- アプリを再度buildする必要がなくなる：Break Pointを追加・編集・削除・移動する際にアプリを再度コンパイルする必要がなくなります。

## 参考文献


https://sarunw.com/posts/better-print-debugging-with-xcode-breakpoints/
