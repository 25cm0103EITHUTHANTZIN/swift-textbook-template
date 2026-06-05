# 第4章：データの永続化

> 執筆者：イトゥタンジン
> 最終更新：2026-05-22

## この章で学ぶこと

この章で学ぶことは、大きく分けて 「データを保存する方法」 です。
特に、SwiftUIでよく使う AppStorage と SwiftData を使って、「アプリを閉じてもデータを残す方法」を学びます。


## 模範コードの全体像

```swift
// ============================================
// 第4章：データの永続化（AppStorage + SwiftData）
// ============================================
// シンプルなメモアプリで、2つの永続化方法を学びます。
// - AppStorage：アプリ設定の保存
// - SwiftData：構造化データの保存
// ============================================

import SwiftUI
import SwiftData

// MARK: - SwiftDataモデル

@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool
    
    init(title: String, content: String, createdAt: Date = .now, isFavorite: Bool = false) {
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isFavorite = isFavorite
    }
}

// MARK: - アプリのエントリポイント
// ※ @main のあるAppファイルに以下を記述してください：
//
// @main
// struct MemoApp: App {
//     var body: some Scene {
//         WindowGroup {
//             ContentView()
//         }
//         .modelContainer(for: Memo.self)
//     }
// }

// MARK: - メインビュー

struct ContentView: View {
    @Environment(\.modelContext) private var modelContext
    @Query(sort: \Memo.createdAt, order: .reverse) private var memos: [Memo]
    @AppStorage("sortByFavorite") private var sortByFavorite: Bool = false
    @AppStorage("userName") private var userName: String = ""
    @State private var isShowingAddSheet = false
    @State private var isShowingSettings = false
    
    var displayedMemos: [Memo] {
        if sortByFavorite {
            return memos.sorted { $0.isFavorite && !$1.isFavorite }
        }
        return memos
    }
    
    var body: some View {
        NavigationStack {
            Group {
                if memos.isEmpty {
                    ContentUnavailableView(
                        "メモがありません",
                        systemImage: "note.text",
                        description: Text("右上の＋ボタンからメモを追加してください")
                    )
                } else {
                    List {
                        ForEach(displayedMemos) { memo in
                            NavigationLink(destination: MemoEditView(memo: memo)) {
                                MemoRow(memo: memo)
                            }
                        }
                        .onDelete(perform: deleteMemos)
                    }
                }
            }
            .navigationTitle(userName.isEmpty ? "メモ帳" : "\(userName)のメモ帳")
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    Button {
                        isShowingSettings = true
                    } label: {
                        Image(systemName: "gear")
                    }
                }
                ToolbarItem(placement: .topBarTrailing) {
                    Button {
                        isShowingAddSheet = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
            .sheet(isPresented: $isShowingAddSheet) {
                MemoAddView()
            }
            .sheet(isPresented: $isShowingSettings) {
                SettingsView(userName: $userName, sortByFavorite: $sortByFavorite)
            }
        }
    }
    
    func deleteMemos(at offsets: IndexSet) {
        for index in offsets {
            let memo = displayedMemos[index]
            modelContext.delete(memo)
        }
    }
}

// MARK: - メモの行

struct MemoRow: View {
    let memo: Memo
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(memo.title)
                    .font(.headline)
                
                Text(memo.content)
                    .font(.caption)
                    .foregroundStyle(.secondary)
                    .lineLimit(2)
                
                Text(memo.createdAt, style: .date)
                    .font(.caption2)
                    .foregroundStyle(.tertiary)
            }
            
            Spacer()
            
            if memo.isFavorite {
                Image(systemName: "star.fill")
                    .foregroundStyle(.yellow)
            }
        }
        .padding(.vertical, 2)
    }
}

// MARK: - メモ追加画面

struct MemoAddView: View {
    @Environment(\.modelContext) private var modelContext
    @Environment(\.dismiss) private var dismiss
    @State private var title = ""
    @State private var content = ""
    
    var body: some View {
        NavigationStack {
            Form {
                Section("タイトル") {
                    TextField("メモのタイトル", text: $title)
                }
                Section("内容") {
                    TextEditor(text: $content)
                        .frame(minHeight: 200)
                }
            }
            .navigationTitle("新しいメモ")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("キャンセル") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("保存") {
                        let memo = Memo(title: title, content: content)
                        modelContext.insert(memo)
                        dismiss()
                    }
                    .disabled(title.isEmpty)
                }
            }
        }
    }
}

// MARK: - メモ編集画面

struct MemoEditView: View {
    @Bindable var memo: Memo
    
    var body: some View {
        Form {
            Section("タイトル") {
                TextField("タイトル", text: $memo.title)
            }
            Section("内容") {
                TextEditor(text: $memo.content)
                    .frame(minHeight: 200)
            }
            Section {
                Toggle("お気に入り", isOn: $memo.isFavorite)
            }
        }
        .navigationTitle("メモを編集")
        .navigationBarTitleDisplayMode(.inline)
    }
}

// MARK: - 設定画面（AppStorageの活用）

struct SettingsView: View {
    @Binding var userName: String
    @Binding var sortByFavorite: Bool
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            Form {
                Section("ユーザー設定") {
                    TextField("あなたの名前", text: $userName)
                }
                Section("表示設定") {
                    Toggle("お気に入りを上に表示", isOn: $sortByFavorite)
                }
                Section {
                    Text("設定はアプリを閉じても保存されます")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .navigationTitle("設定")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .confirmationAction) {
                    Button("完了") { dismiss() }
                }
            }
        }
    }
}

#Preview {
    ContentView()
        .modelContainer(for: Memo.self, inMemory: true)
}

```

**このアプリは何をするものか：**

このアプリは、メモを保存・管理するためのメモ帳アプリです。

ユーザーはメモの追加・編集・削除ができ、お気に入り登録や並び替えも行えます。

また、AppStorage を使ってユーザー設定を保存し、SwiftData を使ってメモデータを永続化することで、アプリを閉じてもデータが保持される仕組みを学ぶアプリです。

## コードの詳細解説

### SwiftDataモデル（@Model）

```swift
@Model
class Memo {
    var title: String
    var content: String
    var createdAt: Date
    var isFavorite: Bool
    
    init(title: String, content: String, createdAt: Date = .now, isFavorite: Bool = false) {
        self.title = title
        self.content = content
        self.createdAt = createdAt
        self.isFavorite = isFavorite
    }
}
```

**何をしているか：**

Memo は、1件分のメモデータを表すモデルです。

タイトル、内容、作成日、お気に入り状態を保存します。

@Model を付けることで、SwiftDataがこのクラスをデータベースに保存できる形として扱います。

**なぜこう書くのか：**

SwiftDataでは、保存したいデータの形を @Model 付きのクラスで定義します。

これにより、Memo のデータを追加・取得・編集・削除できるようになります。

また、createdAt や isFavorite に初期値を入れておくことで、新しいメモを作るときに毎回すべての値を書かなくてもよくなります。

**もしこう書かなかったら：**

@Model を付けないと、SwiftDataの保存対象として認識されません。

そのため、@Query でメモ一覧を取得したり、modelContext.insert() で保存したりできなくなります。

また、init がないと、メモを作成するときに必要な値を正しくセットできず、追加処理が書きにくくなります。

---

### データの追加・削除（modelContext）

```swift
@Environment(\.modelContext) private var modelContext

let memo = Memo(title: title, content: content)
modelContext.insert(memo)

func deleteMemos(at offsets: IndexSet) {
    for index in offsets {
        let memo = displayedMemos[index]
        modelContext.delete(memo)
    }
}
```

**何をしているか：**

modelContext を使って、SwiftDataのデータを追加・削除しています。

- insert(memo)
  
  → 新しいメモを保存する
- delete(memo)
  
  → 保存済みのメモを削除する

という役割があります。

このコードでは、ユーザーが入力したタイトルと内容から Memo オブジェクトを作成し、SwiftDataに保存しています。

また、Listでスワイプ削除したメモをデータベースから消しています。

**なぜこう書くのか：**

SwiftDataでは、データ管理を modelContext が担当しているためです。

```swift
@Environment(\.modelContext)
```
を使うことで、SwiftUI環境からSwiftDataの管理機能を受け取っています。

そして、
```swift
modelContext.insert(memo)
```
を書くことで、「このメモを保存してください」 とSwiftDataへ指示しています。

削除も同じで、

```swift
modelContext.delete(memo)
```
によって、「このデータを削除してください」 と伝えています。

**もしこう書かなかったら：**

insert() を書かなかった場合、メモオブジェクトは作成されても保存されません。

そのため：

- 一覧に表示されない
- アプリ再起動後に消える

状態になります。

また、

```swift
@Environment(\.modelContext)
```
を書かなければ、modelContext 自体が使えないため、 modelContext.insert() や modelContext.delete() でエラーになります。

さらに、delete() を使わない場合は、画面から消えてもSwiftData内にはデータが残り続けます。

---

### @Queryによるデータ取得

```swift
@Query(sort: \Memo.createdAt, order: .reverse)
private var memos: [Memo]
```

**何をしているか：**

@Query を使って、SwiftDataに保存されている Memo データを取得しています。

このコードでは：

- Memo を取得する
- createdAt（作成日時）で並び替える
- .reverse によって新しい順に表示する

という処理をしています。

取得したデータは memos 配列に入り、List表示などで使われます。

**なぜこう書くのか：**

@Query を使うことで、SwiftDataのデータを自動で監視できるためです。

例えば：

- メモを追加
- メモを削除
- メモを編集

すると、memos が自動更新され、画面も自動で再描画されます。

つまり、「データ変更 → 画面更新」 を自動化できます。

また、 sort: \Memo.createdAt  を書くことで、作成日時順に並び替えています。

さらに、order: .reverse によって、新しいメモを上に表示しています。

**もしこう書かなかったら：**

@Query を使わない場合、SwiftDataからデータを取得できません。

そのため：

- Listに何も表示されない
- 保存済みメモを読み込めない

状態になります。

また、並び替えを書かなかった場合は、表示順が不安定になることがあります。

さらに、@Query の自動更新が無いと、

- 手動で再読み込みを書く
- 画面更新処理を書く

必要が出てきて、コードがかなり複雑になります。

---

### @AppStorageによる設定保存

```swift
@AppStorage("sortByFavorite") private var sortByFavorite: Bool = false
@AppStorage("userName") private var userName: String = ""

Toggle("お気に入りを上に表示", isOn: $sortByFavorite)

TextField("あなたの名前", text: $userName)
```

**何をしているか：**

@AppStorage を使って、アプリの設定値を保存しています。

このコードでは：

- userName → ユーザー名を保存
- sortByFavorite → お気に入り優先表示のON/OFFを保存

しています。

設定した値は、アプリを閉じても消えず、次回起動時にも保持されます。

**なぜこう書くのか：**

@AppStorage は、簡単な設定値を保存するのに便利だからです。

例えば：

- 名前
- ダークモード設定
- ON/OFF設定
- 音量設定

などの小さいデータ保存に向いています。

また、 @AppStorage("userName") の "userName" は保存キーになっています。

これによって、「この名前でデータを保存してください」とシステムへ伝えています。

さらに、@State と似た書き方で使えるため、TextField(..., text: $userName) のようにUIと簡単に連携できます。

**もしこう書かなかったら：**

@AppStorage を使わずに @State だけで書いた場合、

@State private var userName = ""

アプリを終了した瞬間にデータが消えます。

つまり：

- 毎回名前を入力し直す
- 設定が保存されない

状態になります。

また、保存機能を自分で実装する必要があり、コードが複雑になります。

さらに、保存キー（"userName" など）が重複すると、別データが上書きされる可能性もあります。

---

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`@Model` | SwiftDataでオブジェクトを永続化するためのマクロ | `@Model final class Memo { ... }` |
| 例：`@Query` | データベースからデータを取得し、変更を自動で反映するプロパティラッパー | `@Query var memos: [Memo]` |
| 例：`@Environment`| SwiftUI環境から値を取得する | `@Environment(\.modelContext)`|
| 例：`@AppStorage`| ユーザー設定を保存する | `@AppStorage("userName") var userName`|
| 例：`@State`| View内で状態を管理する | `@State private var title = ""`|
| 例：`@Binding`| 親Viewと子Viewでデータを共有する | `@Binding var userName: String`|
| 例：`@Bindable`| SwiftDataモデルを直接編集可能にする | `@Bindable var memo: Memo`|
| 例：`.sheet()`| モーダル画面を表示する | `.sheet(isPresented: $isShowingAddSheet)`|
| 例：`.onDelete()`| Listのスワイプ削除を有効にする | `.onDelete(perform: deleteMemos)`|
| 例：`.modelContainer()`| SwiftDataの保存領域を作る | `.modelContainer(for: Memo.self)`|

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
