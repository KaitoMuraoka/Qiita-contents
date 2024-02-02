---
title: 【Swift】private(set) と fileprivate(set)
tags:
  - ''
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
`private(set)` について簡単にまとめました。

# private(set)
private(set) は値が初期化中にのみ設定することができ、いつでも読み取ることができる「読み取り専用のプロパティ」を作成することができます。

```swift
class MyClass {
    private (set) var myProperty: String
    
    init(myProperty: String) {
        self.myProperty = myProperty
    }
}

let myInstance = MyClass(myProperty: "Hello World") // 初期化なのでOK
print(myInstance.myProperty) // 出力：Hello World
myInstance.myProperty = "Goodbey World" // Error すでに初期化されているのでOUT
```
この例にある MyClass の myProperty は、初期化時のみ値を設定することができる読み取り専用のプロパティです。
実際に、初期化時に`"Hello World` という値を設定します。
「初期化中にのみ設定できる」なので問題なく出力できます。
しかし、`"Goodbey World` を代入するとエラーになります。すでに初期化して、読み取り専用プロパティとなっているのでOUTです。

# fileprivate(set)
private(set) に似たものとして、fileprivate(set) というものがあります。
これは、現在のファイル内で変更できます。
以下は、Swift で fileprivate(set) を使用する方法の例です。
 
```swift
class MyFileClass {
    fileprivate(set) var myProperty: String
    
    init(myProperty: String) {
        self.myProperty = myProperty
    }
}

extension MyFileClass {
    func changeMyProperty() {
        myProperty = "Goodbye World"
    }
}

let myFileInstance = MyFileClass(myProperty: "Hello World")
print(myFileInstance.myProperty) // 出力：Hello World
myFileInstance.changeMyProperty()
print(myFileInstance.myProperty) // 出力："Goodbye World"
```

# 参考文献
https://qiita.com/mototaji/items/d1ae68457a118df71458

[https://swift.tecc0.com/?p=631](https://egg-is-world.com/2016/02/09/swift-property-access/)https://egg-is-world.com/2016/02/09/swift-property-access/

https://qiita.com/codelynx/items/43e1c4f176730d952d13
