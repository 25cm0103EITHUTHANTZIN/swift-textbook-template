# AI質問ログ：第1章 WebAPIの基本

## 使用した生成AIツール

ChatGPT

## 質問と回答の記録

### Q1

**質問：** Codableとは？

**AIの回答の要点：**

Codableは、JSONなどのデータをSwiftの構造体に変換するための仕組み。

「データを読み書きできるようにする能力」


Decodable（デコード）→ JSON → Swift

Encodable（エンコード）→ Swift → JSON

```swift
let response = try JSONDecoder().decode(SearchResponse.self, from: data)
```

APIのJSONデータ　→ SearchResponse（Swiftの構造体）

この変換ができるのは SearchResponse が Codable だから!

**自分の理解：**


もし Codable を付けないと

JSONをSwiftの構造体に変換できない

decode() がエラーになる

Codable = JSONとSwiftの橋渡し

API使うなら ほぼ必須


### Q2
**質問：**
```swift
let response = try JSONDecoder().decode(SearchResponse.self, from: data)

の書き方はなんでそう書くのですか？
```
**AIの回答の要点：**

① JSONDecoder()

JSONを解析するためのオブジェクト

② .decode(...)

「この型に変換してね」という命令

SearchResponse.self → この型に変換する

data → APIから取得したJSONデータ

全体の流れ
data（JSON）
↓
JSONDecoder
↓
SearchResponse型に変換
↓
response（使えるデータ）

try がついてる理由 => 変換に失敗する可能性があるから

例えば： JSONの構造が違う / 型が合ってない / その場合エラーになる


**自分の理解：** 「

JSONデータをSearchResponse型に変換して、responseに入れる」

### Q3

**質問：**
```swift
struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?

    var id: Int { trackId }
}
このコードの意味はなんですか？
```

**AIの回答の要点：**

✅ struct Song
👉 曲データの型（設計図）

✅ : Codable
👉 JSON → Swift に変換できるようにする -> iTunes APIの1曲分データをこの形に変換する

✅ : Identifiable
👉 SwiftUIの List で使うためのルール
👉 「このデータを区別するIDあるよ」っていう意味

このコードは
👉 「1曲分のデータを扱うためのモデル」

しかも
Codable → API対応

Identifiable → UI対応

👉 データ + UI 両方で使える設計


**自分の理解：**

Song は、APIから取得した「1曲の情報」を表すためのデータの型（設計図）です。

JSONデータを受け取って使いやすくし、さらにSwiftUIのリスト表示でも使えるようにしたモデルです。

### Q4

**質問：** guard let　の使い方法やなぜ　get let使うかも説明してください.

```swift
func searchMusic() async {
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }
        
        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"
        
        guard let url = URL(string: urlString) else { return }

ここでなんでguard letを使うの？

guard letを使わないとどうなるの？

guard letの使う方法や、どんな場合に使うべきかを詳しくせつめいしてください!

```

**AIの回答の要点：**

**「nil だったらその先の処理を続けないため」**です。

guard let は、Optional(値があるかもしれないし、ないかもしれない　型)に値が入っている時だけ先へ進むための書き方です。

値がある → そのまま続ける

値がない (nil) → else の中を実行して、その場を抜ける

```swift
  guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }
```
searchText を URL で使える文字列に変換しています。

例えば、検索文字が日本語や空白を含むと、そのままURLに入れると危ないことがあります。そのため、URL用に変換しています。

**guard let を使う理由**

もし nil のまま次へ進むと、URL文字列が作れません。だから、
変換できなかったらここで処理を終わるようにしています。

```swift
guard let url = URL(string: urlString) else { return }
```
文字列から URL オブジェクトを作っています。でも、文字列が正しいURL形式でなければ URL(string:) は失敗して nil になります。

URLが作れないのにこのあとを実行することはできません。だから、URLがないならその時点で止めています。

**guard let を使わないとどうなる？**

- エラーになる
- クラッシュする可能性あり
- コードが読みにくくなる

「guard let = nilだったら即終了して、安全に値を使うためのもの」「Optionalを安全にアンラップして、nilの場合は早期リターンするために使います」

### Q5

**質問：** Navigation Stackとはなんですか。

**AIの回答の要点：**

Navigation Stackは、画面遷移を「スタック（積み重ね）」で管理する仕組みです。

スタックは「本の積み重ね」と同じです。

- 一番上の本 → 今表示している画面

- 新しい画面へ → 上に積む（Push）

- 戻る → 上から取り除く（Pop）

① 画面を開く（Push）
    Home → Detail → Profile  (新しい画面がどんどん上に積まれる)

② 戻る（Pop）
    Profile → Detail → Home  (一番上から順に戻る)

**まとめ**

   Navigation Stack = 画面を積み重ねて管理する仕組み

   Push → 画面を開く

   Pop → 戻る

   SwiftUIでは NavigationStack を使う

   
### Q6

**質問：** async awaitとはなんですか。

async（アシンク）は　この処理、待つ必要あるかもよ」っていう宣言、「この関数は時間がかかる処理を含む（非同期処理）」と宣言するキーワードです。

await（アウェイト）は　終わるまで待ってから次に進む、非同期処理（時間がかかる処理）の完了を待つ」ためのキーワードです。


**どういう時に使う**

時間がかかる処理

- API通信（ネットからデータ取る）
- データベース
- ファイル読み込み
- 画像ダウンロード

こういう処理は async をつける

**async と await の関係**

- async → 作る側（関数）
- await → 使う側（呼び出し）

asyncなし（同期）→  一個ずつしかできない　→ [料理 → 食べる → 会計]

asyncあり（非同期）→ 裏で動かせる　→ [料理注文（async） → 待ってる間にスマホ → 出来たら食べる（await]

### Q7

**質問：** do catch とはなんですか。

do-catch は　エラー（例外）を安全に処理するための仕組みです。「エラーが起きてもアプリを止めないための仕組み」

```swift
do {
    try 何かの処理
} catch {
    エラーが起きたときの処理
}
```
***async/awaitとの組み合わせ***
```swift
func loadUser() async throws -> String {
    return "ユーザー"
}

do {
    let user = try await loadUser()
    print(user)
} catch {
    print("読み込み失敗")
}
```
- throws → この関数はエラーを投げる可能性あり
- try → エラーが出るかもしれない処理
- await → 終わるのを待つ
- catch → エラーが出たときの対応



（どんな質問が良い質問だったか。生成AIの回答で間違いや不正確な部分はあったか。次回はどんな質問をしてみたいか。）
