# AI質問ログ：第4章 データの永続化

## 使用した生成AIツール

ChatGPT

## 質問と回答の記録

### Q1

**質問：**

[import SwiftUI import SwiftData]　こらはなにのためですか。どんな時、使いますか

**AIの回答の要点：**

**import SwiftUI** これは 画面(UI)を作るため に使います。

例えば：

- Text
- Button
- NavigationStack
- List
- VStack
- HStack

などを使えるようになります。
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("こんにちは")
    }
}
```
この Text や View は SwiftUI の機能です。

SwiftUIを使う場面

iPhoneアプリで：

- ボタンを置く
- 画面を作る
- リスト表示
- アニメーション
- 画面遷移

など全部UI関係で使います。

**import SwiftData** これは データ保存 のために使います。

アプリを閉じてもデータを残したい時に使います。

例えば

メモアプリで：

- メモを保存
- お気に入り保存
- タスク保存
- 日記保存
など。

```swift
@Model
class Memo {
    var title: String
    var content: String
}
```

この @Model は SwiftData の機能です。

つまり：

Memo を「保存できるデータ」にしています。

SwiftDataを使うと何ができる？

例えば：

| 機能 | 内容 | 
|------|------|
| insert| データ追加|
| delete| 削除 |
| Query| データ取得|
| 永続化| アプリ再起動後も残る|

まとめ
| import | 目的	| 使う時 |
|------ | ------ | ----- |
|import SwiftUI	| 画面を作る | UI作成|
|import SwiftData | データ保存 | メモ・Todo・DB |


**自分の理解：**

イメージすると
- SwiftUI → 「見た目」
- SwiftData → 「保存庫」
です。

つまり：
- SwiftUI → ユーザーが見る画面
- SwiftData → 裏で保存されるデータ

### Q2

**質問：**

**AIの回答の要点：**

**自分の理解：**

### Q3

**質問：**

**AIの回答の要点：**

**自分の理解：**

（質問は何個でも追加してください。多ければ多いほど良いです。）

## 今日の質問を振り返って

（どんな質問が良い質問だったか。生成AIの回答で間違いや不正確な部分はあったか。次回はどんな質問をしてみたいか。）
