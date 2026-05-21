# AI質問ログ：第3章 カメラの利用

## 使用した生成AIツール

ChatGPT

## 質問と回答の記録

### Q1

**質問：**

Struct　とはなんですか。

**AIの回答の要点：**

struct（Structure / 構造体）とは、「データ」と「そのデータに関する機能」を1つにまとめるための型 です。

SwiftUIでは画面そのものを struct で作る。

**なぜclassではなくstructをよく使うの？**

Structは　値そのものをコピーする

Classは　同じデータを共有する

- Struct = コピー型（値型）
- Class  = 共有型（参照型）

*Struct*

```swift
struct User {
    var name: String
}
var user1 = User(name: "EI")
var user2 = user1
```
user1 と user2 は別物になる。

変更してみる　→ user2.name = "Taro"

結果： user1.name → "EI"  user2.name → "Taro"　になる。

なぜ？→ コピーされたから。

*Class*

```swift
class User {
    var name: String
    
    init(name: String) {
        self.name = name
    }
}

var user1 = User(name: "EI")
var user2 = user1
```
ここでは　→ コピーされていない同じデータを見ている。

変更してみる　→ user2.name = "Taro"

結果：　user1.name → "Taro"　　user2.name → "Taro"　になる。

***なぜSwiftUIはStructを使う？***  特にUIは、「毎回新しく作り直す」思想だから、コピー型のStructと相性がいい。

Classは継承できる。

Structはできない。


**自分の理解：**

StructとClassの一番大きな違いは、「コピーされるか」「共有されるか」だと理解した。

Structは値型なので、別の変数に代入するとデータがコピーされる。そのため、片方を変更してももう片方には影響しない。SwiftUIでは画面を安全に更新しやすいため、Viewによく使われている。

一方、Classは参照型なので、同じデータを複数の変数で共有する。どこかで値を変更すると、同じインスタンスを参照している全てに影響する。そのため、ログイン情報や音楽プレイヤーなど、状態を共有したい場合に向いている。

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
