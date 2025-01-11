---
title: PencilKitをSwiftUIで使う
tags:
  - SwiftUI
  - PencilKit
private: false
updated_at: '2024-12-03T07:03:44+09:00'
id: e3ddf0e81ffb4f938df6
organization_url_name: null
slide: false
ignorePublish: false
---
この記事は、2024年アドベンドカレンダー「SwiftUI」の3日目の記事となります。

今回は、ちょうど最近、個人アプリの開発でPencilKitを使ったアプリ開発をしていることもあり、備忘録としての意味合いも込めて、SwiftUIとPencilKitについての記事を書こうと思いました。

# PencilKit
PencilKitは2018年のWWDCで発表された、手書き入力をサポートするフレームワークです。
このフレームワークを使えば、iPadOS や iOS, macOS アプリに手書きのコンテンツを簡単に組み込むことができます。PencilKitは、Apple Pencilまたはユーザーの指からの入力を受け取り、iPadOS、iOS、またはmacOSで表示する画像に変換するiOSアプリケーション用の描画環境を提供します。この環境には、線を作成、消去、選択するためのツールが付属しています。
既存のビュー階層に統合するPKCanvasViewオブジェクトを使用して、iPadアプリでコンテンツをキャプチャします。PKCanvasView オブジェクトは、Apple Pencil や指によるタッチを低レイテンシーでキャプチャできます。canvasオブジェクトは、最終結果をPKDrawingオブジェクトとして送信します。また、iOSやmacOSアプリで表示するために、描画されたコンテンツを画像に変換することもできます。

主に要素としては以下のようなものが挙げられます。

- PKCanvasView : アプリケーションに描画領域を提供します。また、PKCanvasViewは、パンやズーム可能なUIScrollViewでもあります
- PKDrawing : PencilKit のデータモデルです。共有やサムネール用の画像を生成することができます
- ![スクリーンショット 2024-11-04 11.03.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/d980ef9f-cf65-f3f0-3473-19fc8c03f69e.png)
- PKToolPicker : フロート表示のUIを提供します。CanvasViewとは別でViewを切り替えるオブジェクトです
![スクリーンショット 2024-11-04 11.06.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/707293/71d2dd02-4502-e98f-2c1c-33b84dd185c7.png)

- PKTool : CanvasViewで使用される描画ツールを採用するインターフェースです。主に以下のようなものが含まれています
    - PKInkingTool：キャンバスビューに線を描く時に使用する描画特性(線の幅、色、ペンスタイル)を定義する
    - PKEraserTool：CanvasViewで以前に描画された内容を消去するためのツール
    - PKLassonTool：描線や図形を選択するツール

これらの Class, Struct などを使ってアプリを作成していきます。

https://developer.apple.com/documentation/pencilkit

# 本題：PencilKitをSwiftUIで使う方法

SwiftUI では、[UIViewRepresentable](https://developer.apple.com/documentation/swiftui/uiviewrepresentable) プロトコルを使用して、UIKit の View を SwiftUI に統合します。

このプロトコルに準拠すると `makeUIView` と `updateUIView` の2つが必要です。
`makeUIView` は、ビューオブジェクトを作成し、その初期状態を設定します。
`updateUIView` は、SwiftUI からの新しい情報で指定されたビューの状態を更新します。

```swift: CanvasView.swift
import SwiftUI
import PencilKit

struct CanvasView: UIViewRepresentable {
    @Binding var canvasView: PKCanvasView
    private let toolPicker = PKToolPicker()
    func makeUIView(context: Context) -> some UIView {
        canvasView.drawingPolicy = PKCanvasViewDrawingPolicy.anyInput
        
        toolPicker.addObserver(canvasView)
        toolPicker.setVisible(true, forFirstResponder: canvasView)
        
        canvasView.becomeFirstResponder()
        return canvasView
    }
    
    func updateUIView(_ uiView: UIViewType, context: Context) {}
}
```
CanvasView を [`becomeFirstResponder`](https://developer.apple.com/documentation/uikit/uiresponder/1621113-becomefirstresponder) にすることで、オブジェクトをウィンドウの最初のレスポンダにするようにUIKitに依頼します。

次に、上記の CanvasView を使用して、SwiftUI のビューを作成します。

```swift ContentView.swift
import SwiftUI
import PencilKit

struct ContentView: View {
    @State private var canvasView = PKCanvasView()
    var body: some View {
        CanvasView(canvasView: $canvasView)
            .edgesIgnoringSafeArea(.all)
    }
}
```

`PKToolPicker` を使用して、ユーザーがペンや消しゴムなどのツールを選択できるようにします。上記の `updateUIView` メソッド内で、ツールピッカーを表示しています。


