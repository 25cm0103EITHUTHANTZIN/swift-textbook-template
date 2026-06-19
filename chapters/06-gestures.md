# 第6章：ジェスチャー操作

> 執筆者：イトゥタンジン
> 最終更新：2026-06-19

## この章で学ぶこと

この章では、SwiftUIで使える基本的なジェスチャー操作について学びます。

タップ、長押し、ドラッグ、ピンチ（拡大縮小）、回転などの操作を実際に実装しながら、ユーザーの動きに反応するUIの作り方を理解します。

さらに、複数のジェスチャーを組み合わせて、より直感的で使いやすいアプリを作る方法も学びます。

## 模範コードの全体像

```swift

// ============================================
// 第6章（基本）：ジェスチャーで操作するカードアプリ
// ============================================
// タップ、ロングプレス、ドラッグ、ピンチ、回転の
// 各ジェスチャーを実際に体験しながら学びます。
// ============================================

import SwiftUI

// MARK: - メインビュー

struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("タップ & ロングプレス") {
                    TapDemoView()
                }
                NavigationLink("ドラッグ") {
                    DragDemoView()
                }
                NavigationLink("ピンチ（拡大縮小）") {
                    MagnifyDemoView()
                }
                NavigationLink("回転") {
                    RotateDemoView()
                }
                NavigationLink("組み合わせ") {
                    CombinedDemoView()
                }
            }
            .navigationTitle("ジェスチャー体験")
        }
    }
}

// MARK: - タップ & ロングプレス

struct TapDemoView: View {
    @State private var tapCount = 0
    @State private var backgroundColor: Color = .blue
    @State private var isPressed = false
    
    var body: some View {
        VStack(spacing: 30) {
            Text("タップ回数: \(tapCount)")
                .font(.title)
            
            // シングルタップ
            RoundedRectangle(cornerRadius: 16)
                .fill(backgroundColor)
                .frame(width: 200, height: 200)
                .overlay {
                    Text("タップしてね")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .onTapGesture {
                    tapCount += 1
                    backgroundColor = Color(
                        hue: Double.random(in: 0...1),
                        saturation: 0.7,
                        brightness: 0.9
                    )
                }
            
            // ロングプレス
            Circle()
                .fill(isPressed ? .green : .orange)
                .frame(width: 120, height: 120)
                .scaleEffect(isPressed ? 1.3 : 1.0)
                .overlay {
                    Text(isPressed ? "成功!" : "長押し")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .animation(.spring(duration: 0.3), value: isPressed)
                .onLongPressGesture(minimumDuration: 1.0) {
                    isPressed = true
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        isPressed = false
                    }
                }
        }
        .navigationTitle("タップ & ロングプレス")
    }
}

// MARK: - ドラッグ

struct DragDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    
    var body: some View {
        VStack {
            Text("カードをドラッグしてみよう")
                .font(.headline)
                .padding()
            
            Spacer()
            
            RoundedRectangle(cornerRadius: 20)
                .fill(
                    LinearGradient(
                        colors: [.purple, .blue],
                        startPoint: .topLeading,
                        endPoint: .bottomTrailing
                    )
                )
                .frame(width: 200, height: 280)
                .shadow(radius: 8)
                .overlay {
                    VStack {
                        Image(systemName: "hand.draw")
                            .font(.system(size: 40))
                        Text("ドラッグ")
                            .font(.title3)
                    }
                    .foregroundStyle(.white)
                }
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )
            
            Spacer()
            
            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ドラッグ")
    }
}

// MARK: - ピンチ（拡大縮小）

struct MagnifyDemoView: View {
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    
    var body: some View {
        VStack {
            Text("ピンチで拡大縮小")
                .font(.headline)
                .padding()
            
            Text(String(format: "倍率: %.1fx", scale))
                .font(.caption)
                .foregroundStyle(.secondary)
            
            Spacer()
            
            Image(systemName: "star.fill")
                .font(.system(size: 100))
                .foregroundStyle(.yellow)
            // タッチ判定を300×300の透明な領域に広げる
                .frame(width: 300, height: 300)
                .contentShape(Rectangle())
                .scaleEffect(scale)
                .gesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )
            
            Spacer()
            
            Button("リセット") {
                withAnimation(.spring) {
                    scale = 1.0
                    lastScale = 1.0
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ピンチ")
    }
}

// MARK: - 回転

struct RotateDemoView: View {
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero
    
    var body: some View {
        VStack {
            Text("2本指で回転")
                .font(.headline)
                .padding()
            
            Text(String(format: "角度: %.0f°", angle.degrees))
                .font(.caption)
                .foregroundStyle(.secondary)
            
            Spacer()
            
            Image(systemName: "arrow.up")
                .font(.system(size: 80))
                .foregroundStyle(.red)
            // タッチ判定を300×300の透明な領域に広げる
                .frame(width: 300, height: 300)
                .contentShape(Rectangle())
                .rotationEffect(angle)
                .gesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )
            
            Spacer()
            
            Button("リセット") {
                withAnimation(.spring) {
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("回転")
    }
}

// MARK: - 組み合わせ

struct CombinedDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero
    
    var body: some View {
        VStack {
            Text("ドラッグ・ピンチ・回転を同時に")
                .font(.headline)
                .padding()
            
            Spacer()
            
            Image(systemName: "photo.artframe")
                .font(.system(size: 120))
                .foregroundStyle(.indigo)
            // タッチ判定を300×300の透明な領域に広げる
                .frame(width: 300, height: 300)
                .contentShape(Rectangle())
                .scaleEffect(scale)
                .rotationEffect(angle)
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )
            // 複数のジェスチャーを「同時に」効かせるには
            // .gesture を重ねるのではなく .simultaneousGesture を使う
                .simultaneousGesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )
                .simultaneousGesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )
            
            Spacer()
            
            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                    scale = 1.0
                    lastScale = 1.0
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("組み合わせ")
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

このアプリは、SwiftUIの基本的なジェスチャー操作を体験しながら学習するためのサンプルアプリです。

タップ、ロングプレス、ドラッグ、ピンチによる拡大縮小、回転などの操作を実際に試し、それぞれのジェスチャーが画面上のオブジェクトにどのような影響を与えるのかを確認できます。

また、複数のジェスチャーを同時に組み合わせる方法も学ぶことができ、インタラクティブなアプリ開発の基礎を身につけられます。

## コードの詳細解説

### 基本ジェスチャー（タップ、ロングプレス）

```swift
RoundedRectangle(cornerRadius: 16)
    .fill(backgroundColor)
    .frame(width: 200, height: 200)
    .overlay {
        Text("タップしてね")
            .foregroundStyle(.white)
            .font(.headline)
    }
    .onTapGesture {
        tapCount += 1
        backgroundColor = Color(
            hue: Double.random(in: 0...1),
            saturation: 0.7,
            brightness: 0.9
        )
    }

Circle()
    .fill(isPressed ? .green : .orange)
    .frame(width: 120, height: 120)
    .scaleEffect(isPressed ? 1.3 : 1.0)
    .overlay {
        Text(isPressed ? "成功!" : "長押し")
            .foregroundStyle(.white)
            .font(.headline)
    }
    .animation(.spring(duration: 0.3), value: isPressed)
    .onLongPressGesture(minimumDuration: 1.0) {
        isPressed = true
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            isPressed = false
        }
    }
```

**何をしているか：**

この部分では、四角形をタップするとタップ回数が増え、背景色がランダムに変わります。

また、円を1秒間長押しすると、色がオレンジから緑に変わり、少し大きく表示されます。1秒後には元の状態に戻ります。

**なぜこう書くのか：**

@State を使って tapCount、backgroundColor、isPressed の値を管理しています。

SwiftUIでは、画面の見た目を変えたい場合、状態を変更すると自動で画面が更新されます。

onTapGesture はタップ処理、onLongPressGesture は長押し処理を書くための専用の書き方なので、ジェスチャーの動きが分かりやすくなります。

**もしこう書かなかったら：**

onTapGesture を書かなければ、四角形をタップしても何も起きません。

tapCount += 1 を消すと、タップ回数は増えません。

backgroundColor を変更しなければ、色も変わりません。

また、onLongPressGesture を消すと、円を長押ししても反応しません。

DispatchQueue.main.asyncAfter を消すと、一度「成功!」になった後、元の「長押し」状態に戻らなくなります。

---

### ドラッグジェスチャーとオフセット管理

```swift
@State private var offset: CGSize = .zero
@State private var lastOffset: CGSize = .zero

RoundedRectangle(cornerRadius: 20)
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = CGSize(
                    width: lastOffset.width + value.translation.width,
                    height: lastOffset.height + value.translation.height
                )
            }
            .onEnded { _ in
                lastOffset = offset
            }
    )
```

**何をしているか：**

この部分では、カードを指でドラッグしたときに、その移動量に合わせてカードを移動させています。

offset が現在の表示位置を管理し、lastOffset が前回ドラッグ終了時の位置を保存しています。

ユーザーがカードを動かすたびに value.translation から移動距離を取得し、画面上のカードの位置を更新しています。

**なぜこう書くのか：**

DragGesture() の translation は「ドラッグ開始地点からどれだけ動いたか」を表します。

そのため、ドラッグが終わるたびに現在位置を lastOffset に保存しておかないと、次のドラッグ開始時に位置情報がリセットされてしまいます。

例えば、

- 1回目のドラッグ → 右へ100移動
- 2回目のドラッグ → さらに右へ50移動 の場合、

lastOffset.width + value.translation.width とすることで、

100 + 50 = 150 となり、前回の位置を維持したまま移動できます。

**もしこう書かなかったら：**

もし lastOffset を使わず、 offset = value.translation

だけにすると、ドラッグを離した瞬間の位置が保存されません。

例えば、

- 1回目：右へ100移動
- 指を離す
- 2回目：右へ20移動

とすると、本来は120の位置に行きたいのに、 offset = value.translation

では20の位置になってしまいます。

---

### 拡大縮小と回転

```swift
@State private var scale: CGFloat = 1.0
@State private var lastScale: CGFloat = 1.0

Image(systemName: "star.fill")
    .scaleEffect(scale)
    .gesture(
        MagnifyGesture()
            .onChanged { value in
                scale = lastScale * value.magnification
            }
            .onEnded { _ in
                lastScale = scale
            }
    )

@State private var angle: Angle = .zero
@State private var lastAngle: Angle = .zero

Image(systemName: "arrow.up")
    .rotationEffect(angle)
    .gesture(
        RotateGesture()
            .onChanged { value in
                angle = lastAngle + value.rotation
            }
            .onEnded { _ in
                lastAngle = angle
            }
    )
```

**何をしているか：**

この部分では、ピンチ操作による拡大・縮小と、2本指による回転操作を実装しています。

MagnifyGesture() は指を広げたり閉じたりしたときの倍率を取得し、scaleEffect(scale) によって画像の大きさを変更しています。

また、RotateGesture() は2本指の回転角度を取得し、rotationEffect(angle) によって画像を回転させています。

ユーザーが指を動かすたびに値が更新され、リアルタイムで拡大・縮小や回転が反映されます。

**なぜこう書くのか：**

MagnifyGesture() の value.magnification は、「今回のジェスチャー開始時を基準にした倍率」を返します。

例えば、

- 最初に 1.5倍まで拡大
- 指を離す
- 次にさらに 2倍まで拡大

という操作をするとき、

scale = lastScale * value.magnification

とすることで、 1.5 × 2.0 = 3.0倍

となり、前回までの拡大状態を維持できます。

回転も同じ考え方です。

angle = lastAngle + value.rotation

とすることで、前回までの回転角度に今回の回転量を加算できます。

そのため、lastScale , lastAngle を用意して、ジェスチャー終了時の状態を保存しています。

**もしこう書かなかったら：**

もし scale = value.magnification

だけにすると、毎回1倍を基準に計算されます。

例えば、

- 1回目で2倍
- 指を離す
- 2回目で1.5倍

と操作しても、

本来 2.0 × 1.5 = 3.0倍

になるはずが、

1.5倍 に戻ってしまいます。

同様に、 angle = value.rotation

だけにすると、回転するたびに角度がリセットされます。

また、 .scaleEffect(scale)

を削除すると、scale の値は変化していても画像の大きさは変わりません。

.rotationEffect(angle)  を削除すると、angle の値が更新されても画像は回転しません。

---

### ジェスチャーの組み合わせとアニメーション

```swift
Image(systemName: "photo.artframe")
    .scaleEffect(scale)
    .rotationEffect(angle)
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = CGSize(
                    width: lastOffset.width + value.translation.width,
                    height: lastOffset.height + value.translation.height
                )
            }
            .onEnded { _ in
                lastOffset = offset
            }
    )
    .simultaneousGesture(
        MagnifyGesture()
            .onChanged { value in
                scale = lastScale * value.magnification
            }
            .onEnded { _ in
                lastScale = scale
            }
    )
    .simultaneousGesture(
        RotateGesture()
            .onChanged { value in
                angle = lastAngle + value.rotation
            }
            .onEnded { _ in
                lastAngle = angle
            }
    )

.animation(.spring(duration: 0.3), value: isPressed)
```

**何をしているか：**

この部分では、1つの画像に対して「ドラッグ」「拡大縮小」「回転」の3つのジェスチャーを同時に適用しています。

- ドラッグ → 画像を移動
- ピンチ → 画像を拡大・縮小
- 回転 → 画像を回転

を同時に行うことができます。

また、

.animation(.spring(duration: 0.3), value: isPressed)

では、ロングプレス時に円が大きくなったり小さくなったりする変化を、バネのような自然な動きでアニメーション表示しています。

**なぜこう書くのか：**

複数の .gesture() を並べると、後から書いたジェスチャーが優先され、前のジェスチャーが動作しなくなる場合があります。

そこで、

.simultaneousGesture(...)

を使うことで、

- ドラッグしながら
- 拡大しながら
- 回転する

という複数操作を同時に認識できるようにしています。

.animation(.spring(duration: 0.3), value: isPressed)

を付けることで、状態変化が急に切り替わるのではなく、自然な動きになります。

例えば、

.scaleEffect(isPressed ? 1.3 : 1.0)

だけだと、

1.0倍 → 1.3倍  が一瞬で切り替わります。

アニメーションを追加すると、ゆっくり大きくなって戻るため、ユーザーにとって分かりやすい動きになります。

**もしこう書かなかったら：**

.simultaneousGesture() を使わないと、ドラッグ・拡大縮小・回転を同時に操作できなくなります。

また、.scaleEffect()、.rotationEffect()、.offset() のどれかを削除すると、それぞれ拡大縮小・回転・移動ができなくなります。

.animation() を削除すると、ロングプレス時の変化が瞬時に切り替わり、不自然な見た目になります。

---

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`DragGesture` | ドラッグジェスチャーを認識するジェスチャーレコグナイザー | `.gesture(DragGesture().onChanged { ... })` |
| 例：`MagnificationGesture` | ピンチジェスチャーで拡大・縮小を認識 | `.gesture(MagnificationGesture().onChanged { scale in ... })` |
| | | |
| | | |
| | | |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：
- 結果：
- わかったこと：

**実験2：**
- やったこと：
- 結果：
- わかったこと：

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**
   **得られた理解：**

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
