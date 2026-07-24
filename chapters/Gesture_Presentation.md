# 第6章：ジェスチャー発表

> 執筆者：イトゥタンジン
> 最終更新：2026-07-24


## この章で学ぶこと

この章では、SwiftUIで使える基本的なジェスチャー操作について学びます。

タップ、長押し、ドラッグ、ピンチ（拡大縮小）、回転などの操作を実際に実装しながら、ユーザーの動きに反応するUIの作り方を理解します。

さらに、複数のジェスチャーを組み合わせて、より直感的で使いやすいアプリを作る方法も学びます。


## この章を選んな理由

私は「Gesture」の章を選びました。

理由は、最初は「ドラッグすると動く」ということしか分かりませんでしたが、AIへ質問を重ねることで、「なぜこのコードを書くのか」まで理解できるようになったからです。

今日は、その中でも理解が深まったポイントを紹介します。


## 最も理解できた章のソースコード解説

```swift
@State private var offset: CGSize = .zero

RoundedRectangle(cornerRadius: 16)
    .fill(.blue)
    .frame(width: 120, height: 120)
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = value.translation
            }
    )
```

**説明**

①

```swift
@State private var offset: CGSize = .zero
```

ドラッグした位置を保存する変数です。

②

```swift
.offset(offset)
```

このModifierが 保存した値だけ 表示位置を動かします。

ここで「.offsetはSwiftUIが最初から用意しているModifierです。」

③

```swift
DragGesture()
```

ドラッグを検知します。

ドラッグしている間だけ動きます。

④

```swift
.onChanged { value in
    offset = value.translation
}
```

ドラッグするたびに 指がどれだけ動いたかを 取得しています。

その値をoffsetへ入れるので 画面が動いて見えます。


## AIに良い質問

**質問**

***AIへの質問　①***

「offsetの値は分かりますが、.offsetは何ですか？」

***AIから理解したこと***

最初は、offsetという変数だけを見ていました。

AIに質問したことで、

- .offset()はSwiftUIが最初から用意しているView Modifierであること
- offset変数の値を受け取り、その分だけ表示位置を移動する機能であること
- .padding()や.background()なども同じView Modifierの仲間であること

を理解しました。

理解が変わったことは、

「offsetという変数が画面を動かしている」のではなく、

SwiftUIが用意した.offset()という機能に、自分で用意したoffsetの値を渡している

という仕組みが分かりました。

***AIへの質問***

.gesture は元々ある機能ですか

***AIから理解したこと***

AIから、

.gesture() もSwiftUIが最初から用意しているModifierであり、

その中に

- TapGesture()
- DragGesture()
- MagnifyGesture()
- RotationGesture()

などのジェスチャーを設定できることを学びました。

理解が変わったことは、

Gestureを自分で作っていると思っていましたが、

実際にはSwiftUIが用意した機能を利用しているだけだと理解できました。

***AIへの質問***

CGSize.zero は他の書き方なら？

***AIから理解したこと***

AIから

CGSize.zero　は

```swift
CGSize(width: 0, height: 0)
```
の省略形だと教えてもらいました。

***AIへの質問***

withAnimation と .animation の違い

***AIから理解したこと***

withAnimation は 状態を変更するときだけアニメーションする

一方で

.animation() は 指定した値が変化するたびに自動でアニメーションする という違いを教えてもらいました。

理解が変わったことは、

どちらも同じ機能だと思っていましたが、

実際には

- 状態を変更する側
- Viewの表示を制御する側 という役割の違いがあることを理解できました。

***AIへの質問***

.simultaneousGesture は何ですか？

***AIから理解したこと***

最初は、1つのViewには1つのGestureしか設定できないと思っていました。

AIに質問したことで、

.simultaneousGesture() は複数のGestureを同時に認識させるためのModifierだと理解しました。

理解が変わったことは、

.gesture()だけでは1つのGestureしか設定できませんが、.simultaneousGesture()を使うことで複数のGestureを同時に動かせることが分かりました。

つまり、

SwiftUIにはGesture同士を組み合わせるための仕組みが用意されていることを理解しました。

***AIへの質問***

.contentShape(Rectangle()) は何ですか？

***AIから理解したこと***

最初は、

Viewの見た目が四角なので、

どこを押してもタップできると思っていました。

しかしAIに質問すると、

.contentShape(Rectangle())は

タップやGestureが反応する範囲（ヒット判定）を指定するModifierだと教えてもらいました。

例えば、

```swift
VStack {
    Image(systemName: "star")
    Text("Favorite")
}
.contentShape(Rectangle())
```
と書くと、

画像や文字の周りの余白も含めて、

四角全体をタップできるようになります。

理解が変わったことは、

最初は見た目だけを変えるModifierだと思っていました。

しかし実際には、

見た目ではなく、「どこをタップできるか」を決めるModifierだと理解しました。

つまり、画面に表示される形と、タップできる範囲は別々に設定できることを学びました。


## 最も腹落ちしたAIからの回答

```swift
withAnimation {
    offset = value.translation
}
```

```swift
.offset(offset)
.animation(.easeInOut, value: offset)
```

withAnimation → 変更する瞬間だけアニメーション

.animation()　→ offsetが変わるたび毎回アニメーション

## 苦労したところ

.gesture()だけでは全部動くと思っていた

最初は　.gesture()　を書けば

どこを押してもGestureが反応すると思っていました。

でも実際には　Textだけ反応したり、

余白では反応しなかったりしました。

AIへ質問すると

.contentShape(Rectangle())　を追加すると　余白もタップできることを理解しました。

## 実験

**.onEnded の中を コメントアウトする**

変更前

```swift
.onEnded { _ in
    lastOffset = offset
}
```

変更後

```swift
.onEnded { _ in
    // lastOffset = offset
}
```

**予想**

1回目のドラッグは動くが、次のドラッグで前回の位置が使われない。

**実際の結果**

1回目はViewを好きな場所まで動かせます。

しかし、もう一度ドラッグを始めると、前回移動した位置が保存されていないため、Viewが急に元の基準位置へ近づくような動きをします。

**分かったこと**

lastOffset = offset

は、Viewを動かす処理ではありません。指を離した場所を、次回のドラッグ開始位置として保存する処理です。


## 最後

Gestureの章では、AIに質問しながら学習を進めたことで、コードの動きだけでなく、それぞれの文法やModifierの役割まで理解できるようになりました。

特に、.gestureや.offset、.simultaneousGestureなどがSwiftUIで用意されている機能であることを理解でき、Gestureの仕組みをより深く学ぶことができました。


