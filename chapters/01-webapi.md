# 第1章：WebAPIの基本

> 執筆者： イトゥタンジン
> 最終更新：2026-4-15

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

例：この章では、インターネット上のサービス（API）からデータを取得して、アプリ内に表示する方法を学ぶ。具体的にはiTunes Search APIを使って音楽を検索し、その結果をリスト表示するアプリを題材にする。

## 模範コードの全体像
```swift
// ============================================
// 第1章（基本）：iTunes Search APIで音楽を検索するアプリ
// ============================================
// このアプリは、iTunes Search APIを使って
// 音楽（曲）を検索し、結果をリスト表示します。
// APIキーは不要で、すぐに動かすことができます。
// ============================================

import SwiftUI

// MARK: - データモデル

struct SearchResponse: Codable {
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?

    var id: Int { trackId }
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var songs: [Song] = []
    @State private var searchText: String = ""
    @State private var isLoading: Bool = false

    var body: some View {
        NavigationStack {
            VStack {
                // 検索バー
                HStack {
                    TextField("アーティスト名を入力", text: $searchText)
                        .textFieldStyle(.roundedBorder)

                    Button("検索") {
                        Task {
                            await searchMusic()
                        }
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(searchText.isEmpty)
                }
                .padding(.horizontal)

                // 検索結果リスト
                if isLoading {
                    ProgressView("検索中...")
                        .padding()
                    Spacer()
                } else if songs.isEmpty {
                    ContentUnavailableView(
                        "曲を検索してみよう",
                        systemImage: "music.note",
                        description: Text("アーティスト名を入力して検索ボタンを押してください")
                    )
                } else {
                    List(songs) { song in
                        SongRow(song: song)
                    }
                }
            }
            .navigationTitle("Music Search")
        }
    }

    // MARK: - API通信

    func searchMusic() async {
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }

        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

        guard let url = URL(string: urlString) else { return }

        isLoading = true

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let response = try JSONDecoder().decode(SearchResponse.self, from: data)
            songs = response.results
        } catch {
            print("エラー: \(error.localizedDescription)")
            songs = []
        }

        isLoading = false
    }
}

// MARK: - 曲の行ビュー

struct SongRow: View {
    let song: Song

    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: URL(string: song.artworkUrl100)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Color.gray.opacity(0.3)
            }
            .frame(width: 60, height: 60)
            .clipShape(RoundedRectangle(cornerRadius: 8))

            VStack(alignment: .leading, spacing: 4) {
                Text(song.trackName)
                    .font(.headline)
                    .lineLimit(1)

                Text(song.artistName)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
        }
        .padding(.vertical, 4)
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**
```markdown
iTunes Search APIを使って、音楽（曲）を検索し、結果をリスト表示するアプリです。
```
## コードの詳細解説

### データモデル（Codable構造体）

```swift
struct SearchResponse: Codable {
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let previewUrl: String?
    
    var id: Int { trackId }
}
```

**何をしているか：**

APIから返ってきたJSONデータを、Swiftで使える形に変換するための設計図

つまり：

API → JSON（文字のデータ）

Swift → struct（型）

この2つをつなぐのが「データモデル」

**なぜこう書くのか：**

APIのデータの形に合わせて書いているから。

曲は「results」の中に入っている。

resultsの中に曲の配列があります。

箱の中に物が入っている

箱 → SearchResponse

中身 → results

物 → Song

理由①：JSONをそのまま使えない

理由②：JSONDecoderとセットで使う

理由③：安全にデータを扱える

APIから来る　JSONを　自動で　Swiftの型に変換するため
```swift
let response = try JSONDecoder().decode(SearchResponse.self, from: data)
```
これが成立するのは Codable があるから



**もしこう書かなかったら：**

箱を無視して中身を取ろうとする。

データが正しく読み取れない（＝アプリが動かない or エラーになる）

JSONを変換できない

つまり

```swift
JSONDecoder().decode(...)
```

がエラーになる

---

### API通信の処理

```swift
func searchMusic() async {
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }

        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

        guard let url = URL(string: urlString) else { return }

        isLoading = true

        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            let response = try JSONDecoder().decode(SearchResponse.self, from: data)
            songs = response.results
        } catch {
            print("エラー: \(error.localizedDescription)")
            songs = []
        }

        isLoading = false
    }
```

**何をしているか：**

API通信の処理は、大きく分けて次の流れで動いています。

1. ユーザーの入力をもとにリクエスト内容を作る
2. 正しい形式のURLを作る
3. サーバーにリクエストを送る
4. データが返ってくるまで待つ
5. 受け取ったデータをアプリで使える形に変換する
6. 画面に反映する

つまり、

「入力された情報をもとに外部からデータを取りに行き、結果を画面に表示する処理」です。

**全部つなげるとこうなる**

TextField入力

   ↓
   
searchText に入る

   ↓
   
URLを作る

   ↓
   
URLSessionで通信

   ↓
   
awaitで待つ

   ↓
   
JSONDecoderで変換

   ↓
   
songsに代入

   ↓
   
Listが更新される



**なぜこう書くのか：**

API通信は、通常すぐには終わりません。

ネットワークの状況やサーバーの状態によって、時間がかかることがあります。

そのため、

1. 処理が終わるまで待つ仕組み

2. エラーが発生した場合に対応する仕組み

を用意する必要があります。

また、通信中に画面が止まってしまうとユーザー体験が悪くなるため、
アプリの動作を止めないように設計する必要があります。

**エラー処理を含める理由**

インターネット通信には必ず不確実性があります。

例えば：

. ネットがつながっていない
. サーバーが応答しない
. データの形式が違う

このような問題が起きる可能性があります。

そのため、「失敗する前提」で処理を書く必要があります。


**もしこう書かなかったら：**

API通信の処理を正しく書かなかった場合、

1. データを取得できない
2. アプリがフリーズする
3. エラーでアプリが落ちる
4. データが正しく表示されない
5. ユーザーに状況が伝わらない

---

### ビューの構成

```swift
NavigationStack {
    VStack {
        // 検索バー
        HStack {
            TextField("アーティスト名を入力", text: $searchText)
                .textFieldStyle(.roundedBorder)
            
            Button("検索") {
                Task {
                    await searchMusic()
                }
            }
            .buttonStyle(.borderedProminent)
            .disabled(searchText.isEmpty)
        }
        .padding(.horizontal)
        
        // 状態に応じた表示切り替え
        if isLoading {
            ProgressView("検索中...")
                .padding()
            Spacer()
        } else if songs.isEmpty {
            ContentUnavailableView(
                "曲を検索してみよう",
                systemImage: "music.note",
                description: Text("アーティスト名を入力して検索ボタンを押してください")
            )
        } else {
            List(songs) { song in
                SongRow(song: song)
            }
        }
    }
    .navigationTitle("Music Search")
}
```

**何をしているか：**

この部分では、アプリの画面全体を構成しています。

入力欄と検索ボタンを横に並べ、その下に状態に応じた表示（読み込み中・案内・検索結果）を切り替えて表示しています。

具体的には、ユーザーが入力して検索を行う操作部分と、
その結果や状態を表示する部分を一つの画面としてまとめています。

また、現在の状態に応じて表示内容を変えることで、
ユーザーに「今何が起きているか」が分かるようにしています。

**なぜこう書くのか：**

画面は常に同じ内容を表示するのではなく、
アプリの状態に応じて変化する必要があるため、このように構成しています。

例えば、

- 入力前は何をすればいいか案内する
- 検索中は処理中であることを示す
- 検索結果があれば一覧を表示する

というように、状況に応じて適切な情報を見せることで、
ユーザーが迷わず操作できるようになります。

また、入力・操作・結果表示を一つの流れとしてまとめることで、
画面全体の構造が分かりやすくなります。

**もしこう書かなかったら：**

このように構成しなかった場合、次のような問題が起こります。

- 画面の要素が整理されず、見づらくなる
- 状態に関係なく同じ内容が表示され、状況が分からなくなる
- 検索中かどうかが分からず、ユーザーが不安になる
- 結果があるのに表示されない、または空なのに何も表示されない

結果として、ユーザーが使いにくいアプリになります。

---

## 新しく学んだSwiftの文法・API
| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`Codable` | JSONデータとSwiftの構造体を相互変換するプロトコル | `struct Song: Codable { ... }` |
| 例：`async/await` | 非同期処理を同期的に書ける構文 | `let data = try await URLSession.shared.data(from: url)` |
| 例:`NavigationStack`| 画面遷移を「スタック（積み重ね）」で管理する仕組み| `NavigationStack { ... }`|

## 自分の実験メモ

```swift
import SwiftUI

// ============================================
// データモデル
// ============================================

// APIから返ってくる全体のデータを受け取る構造体
struct SearchResponse: Codable {
    let results: [Song]
}

// 1曲分のデータを受け取る構造体
struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String

    // 【実験3で変更】
    // 元コードでは previewUrl: String? だった
    // → 今回は String に変更して、「必ず値がある」前提にした
    // → ただし実際には値がない曲もあるので、デコード失敗の可能性がある
    let previewUrl: String

    // Listで各データを区別するためのID
    var id: Int { trackId }
}

// ============================================
// メイン画面
// ============================================

struct ContentView: View {
    @State private var songs: [Song] = []
    @State private var searchText: String = ""

    // 【実験2で削除】
    // 元コードには isLoading があった
    // → 読み込み中かどうかを管理していた
    // → 今回はローディング表示の実験のため削除した
    // @State private var isLoading: Bool = false

    var body: some View {
        NavigationStack {
            VStack {

                // ============================================
                // 検索バー部分
                // ============================================
                HStack {
                    TextField("アーティスト名を入力", text: $searchText)
                        .textFieldStyle(.roundedBorder)

                    Button("検索") {
                        Task {
                            await searchMusic()
                        }
                    }
                    .buttonStyle(.borderedProminent)

                    // 【実験1で削除】
                    // 元コードでは、入力が空のとき検索ボタンを押せないようにしていた
                    // → .disabled(searchText.isEmpty)
                    // 今回は「空でも押せるようにしたらどうなるか」を試すため削除した
                    // → その結果、空検索でも通信が行われるようになる
                }
                .padding(.horizontal)

                // ============================================
                // 結果表示部分
                // ============================================

                // 【実験4で変更】
                // 元コードでは if / else if / else で状態ごとに表示を切り替えていた
                //
                // 元の流れ：
                // 1. 読み込み中 → ProgressView
                // 2. 結果が空 → ContentUnavailableView
                // 3. 結果あり → List
                //
                // 今回はその状態分岐を全部なくして、
                // 常に List だけを表示するようにした
                //
                // そのため、
                // ・読み込み中でも変化が見えない
                // ・空のときも案内表示が出ない
                // という状態になる
                List(songs) { song in
                    SongRow(song: song)
                }
            }
            .navigationTitle("Music Search")
        }
    }

    // ============================================
    // API通信処理
    // ============================================

    func searchMusic() async {
        // 検索文字をURLで使える形に変換
        guard let encodedText = searchText.addingPercentEncoding(
            withAllowedCharacters: .urlQueryAllowed
        ) else { return }

        // APIのURLを作成
        let urlString = "https://itunes.apple.com/search?term=\(encodedText)&media=music&country=jp&limit=25"

        // StringをURL型に変換
        guard let url = URL(string: urlString) else { return }

        // 【実験2で削除】
        // 元コードでは通信開始前に isLoading = true を書いていた
        // → 読み込み中の表示を出すため
        // 今回はローディング管理を削除したので、ここも削除
        // isLoading = true

        do {
            // サーバーに通信してデータを取得
            let (data, _) = try await URLSession.shared.data(from: url)

            // JSONデータを SearchResponse 型に変換
            let response = try JSONDecoder().decode(SearchResponse.self, from: data)

            // 検索結果を songs に代入
            // @State なので、値が変わると画面が更新される
            songs = response.results
        } catch {
            print("エラー: \(error.localizedDescription)")

            // エラー時は空配列にする
            songs = []
        }

        // 【実験2で削除】
        // 元コードでは通信終了後に isLoading = false を書いていた
        // → 読み込み中を解除するため
        // 今回は削除
        // isLoading = false
    }
}

// ============================================
// 1行分の表示
// ============================================

struct SongRow: View {
    let song: Song

    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: URL(string: song.artworkUrl100)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Color.gray.opacity(0.3)
            }
            .frame(width: 60, height: 60)
            .clipShape(RoundedRectangle(cornerRadius: 8))

            VStack(alignment: .leading, spacing: 4) {
                Text(song.trackName)
                    .font(.headline)
                    .lineLimit(1)

                Text(song.artistName)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
        }
        .padding(.vertical, 4)
    }
}

// ============================================
// プレビュー
// ============================================

#Preview {
    ContentView()
}
```

**実験1：**
- やったこと：　検索ボタンの無効化を外し、入力欄が空の状態でも検索できるように変更した。
- 結果：　空のままでも検索処理は実行されたが、意味のある検索結果にならなかった。また、不要な通信が発生した。
- わかったこと：　入力内容を事前に確認する処理は、無駄な通信を防ぎ、ユーザーが意図しない操作をしないようにするために重要だと分かった。

**実験2：**
- やったこと：　読み込み中を表す状態管理とローディング表示を削除し、検索中でも画面の見た目が変わらないようにした。
- 結果：　検索ボタンを押しても処理中であることが画面から分からず、アプリが止まっているように感じられた
- わかったこと：　API通信のように時間がかかる処理では、現在処理中であることをユーザーに伝える表示が必要であり、ローディング表示は使いやすさに大きく関わると分かった。
  
**実験3：**
- やったこと：　値が存在しない可能性のある項目を、必ず値が入る前提の形に変更した。
- 結果： 一部のデータで読み込み時にエラーが発生し、正常に変換できない場合があった。
- わかったこと： APIから取得するデータは常に完全とは限らないため、値が欠ける可能性を考慮した設計が必要だと分かった。

**実験4：**
- やったこと： 状態による表示の切り替えを削除し、常に一覧表示だけを行うように変更した。
- 結果： 初期状態や検索結果が空の場合でも案内表示が出なくなり、ユーザーが次に何をすればよいか分かりにくくなった。
- わかったこと： 画面は常に同じ表示にするのではなく、状態に応じて内容を切り替えることで、ユーザーにとって分かりやすい画面になると理解した。

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**  データモデル（Codable構造体）はなぜ必要なのか？
   **得られた理解：**  APIから取得したJSONデータはそのままでは扱えないため、Swiftで使える形に変換する必要があると分かった。
   データモデルはその橋渡しの役割を持ち、型として定義することで安全にデータを扱えるようになると理解した。

3. **質問：**  async / await は何をしているのか？
   **得られた理解：**  API通信のように時間がかかる処理を行うときに、処理が終わるまで待ちながらもアプリ全体は止めないための仕組みであると分かった。
   awaitによって処理の順序を保ちながら、ユーザー操作に影響を与えない設計ができると理解した。

5. **質問：**  なぜ状態（@State）によって画面が変わるのか？
   **得られた理解：**  SwiftUIではデータの状態が変わると自動で画面が更新される仕組みになっていると分かった。
   特に検索結果やローディング状態の変化によって表示が切り替わることで、コードと画面が連動していることを理解した。

## この章のまとめ

```markdown
この章では、API通信を行うアプリの基本的な仕組みについて学んだ。

特に重要だと感じたのは、データの流れと状態管理の考え方である。

アプリは、ユーザーの入力をもとに外部からデータを取得し、その結果を画面に表示するという流れで動いている。また、その過程では非同期処理やエラー処理が必要であり、安全にデータを扱うための設計が重要であると分かった。

さらに、SwiftUIではデータの状態が変わることで画面が更新されるため、状態を正しく管理することがUIの動作に直結することを理解した。

今後は、今回学んだ「データの流れ」「非同期処理」「状態による画面更新」を意識して、より実践的なアプリ開発に活かしていきたい。
```
