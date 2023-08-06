---
title: [Xcode]breakpointでprintデバッグする方法
tags:
  - iOS
  - Swift
  - Xcode
  - debug
private: false
updated_at: ''
id: null
organization_url_name: null
---

# 概要
開発や競技プログラミングをしている際、
- その変数が任意のタイミングでどのような値を持っているのか
- この関数はどのタイミングで処理されているのか
といったことが気になる場面が多々あると思います。

そこで登場するのが`print`関数です。

これを使うことにより、コンソールやターミナル上に変数や文字列を表示することが可能です。

しかし、この関数にはいくつかの欠点があります。

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

```Swift
    override func viewDidLoad() {
        super.viewDidLoad()
#if DEBUG
        print("Hello World")
#endif
        // 色々な処理
    }
```

しかし、これは不要なコードを隠しているだけで、根本的な解決にはなっていません。

::note warn
処理内容のログを取得したい場合は、[`OSLog`](https://developer.apple.com/documentation/os/oslog)のようなツールを使用してください。
この記事では、一時的なバグを修正するために`print`関数を使うことについて解説します。
::

また、printデバッグのもう1つの問題点は、新しい`print`関数を追加したい場合、アプリを再度buildする必要があります。
