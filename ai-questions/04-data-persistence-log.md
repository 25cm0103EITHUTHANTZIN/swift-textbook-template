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

@Model　をなんでつけているんですか？

**AIの回答の要点：**

@Model を付けることで、SwiftDataが　「このクラスはデータベースに保存するデータですよ」　と理解します。

**init は何のため？** 
```swift
init(title: String, content: String, createdAt: Date = .now, isFavorite: Bool = false)
```
init は、新しいMemoを作る時に最初の値を入れるため です。

**なぜ createdAt: Date = .now？**  これは、作成日を自動で「今の時間」にするためです。 だから、Memoを作る時に毎回日付を書かなくてもいいです。

**自分の理解：**　

「タイトル・内容・作成日・お気に入り状態を持つメモを、SwiftDataに保存できる形で作っている」という意味です。

### Q3

**質問：**

```swift
struct ContentView: View {
    @Environment(\.modelContext) private var modelContext
    @Query(sort: \Memo.createdAt, order: .reverse) private var memos: [Memo]
    @AppStorage("sortByFavorite") private var sortByFavorite: Bool = false
    @AppStorage("userName") private var userName: String = ""
    @State private var isShowingAddSheet = false
    @State private var isShowingSettings = false
```
この部分も詳しく説明お願いします.

**AIの回答の要点：**

1. @Environment(\.modelContext)
   ```swift
   @Environment(\.modelContext) private var modelContext
   ```
これは SwiftDataに保存・削除するための道具 を取得しています。

つまり modelContext は、SwiftDataのデータを操作する入口です。

2. @Query
   ```swift
   @Query(sort: \Memo.createdAt, order: .reverse) private var memos: [Memo]
   ```
これは SwiftDataに保存されているMemo一覧を自動で取得する ものです。

*sort: \Memo.createdAt*  これは、メモの createdAt を使って並び替えるという意味です。

*createdAt*  は作成日でしたね。

*var createdAt: Date*  つまり、作成日順で表示します。

*order: .reverse* これは 逆順 という意味です。

作成日を逆順にすると、 「新しいメモ → 古いメモ」　の順番になります。だから、新しく追加したメモが上に表示されます。

3. @AppStorage("sortByFavorite")
   ```swift
   @AppStorage("sortByFavorite") private var sortByFavorite: Bool = false
   ```
これは お気に入りを上に表示する設定を保存する ためです。

Bool なので、
- true  → お気に入りを上に表示する
- false → 普通の順番で表示する です。

@AppStorage はアプリを閉じても保存されます。

この "sortByFavorite" は保存する時の名前、つまりキーです。

イメージ： sortByFavorite という名前で true / false を保存する です。

4. @AppStorage("userName")
   ```swift
   @AppStorage("userName") private var userName: String = ""
   ```
これは ユーザー名を保存する ためです。

例えば設定画面で名前を入れると、イさん みたいに保存できます。

そして画面タイトルで使えます。 userName.isEmpty ? "メモ帳" : "\(userName)のメモ帳"

**自分の理解：**

- @Environment → SwiftDataを操作する道具
- @Query       → SwiftDataからデータを読む
- @AppStorage  → 小さい設定を保存する
- @State       → 今だけの画面状態を管理する

## 今日の質問を振り返って

（どんな質問が良い質問だったか。生成AIの回答で間違いや不正確な部分はあったか。次回はどんな質問をしてみたいか。）
