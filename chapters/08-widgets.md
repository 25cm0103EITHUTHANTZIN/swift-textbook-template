# 第8章：ウィジェット

> 執筆者：　イトゥタンジン

> 最終更新：2026-07-17

## この章で学ぶこと

この章では、WidgetKitを使ってiPhoneのホーム画面に表示できるウィジェットの作成方法を学びます。ウィジェットに表示するデータを管理するTimelineEntryやTimelineProviderの役割を理解し、日付に応じて表示内容を更新する仕組みを学習します。また、CalendarやDateを利用して「毎日0時に更新する」スケジュールを設定する方法や、WidgetFamilyを使って小サイズ・中サイズなど、ウィジェットのサイズごとに異なるレイアウトを表示する方法も学びます。さらに、StaticConfigurationによるウィジェットの設定、containerBackgroundによる背景デザイン、#Previewを使ったプレビュー機能など、WidgetKitの基本的な使い方を一通り習得し、ホーム画面で動作する日替わり名言ウィジェットを完成させます。

## 模範コードの全体像

```swift
//
//  QuoteWidget.swift
//  QuoteWidget
//
//  Created by cmStudent on 2026/07/01.
//


// ============================================
// ■ ウィジェット側のコード（自動生成された QuoteWidget.swift を "全置換"）
// ============================================
// ※ 下の /* ... */ を外し、自動生成ファイルの中身を全部消してから貼り付けます。
// ※ Quote と QuoteStore は手順3で QuoteStore.swift に移し、両ターゲットの
//    Target Membership に入れてあるので、ここでは再定義しません。
// ============================================


 import WidgetKit
 import SwiftUI
 
 // MARK: - タイムラインエントリ
 
 struct QuoteEntry: TimelineEntry {
 let date: Date
 let quote: Quote
 }
 
 // MARK: - タイムラインプロバイダ
 
 struct QuoteProvider: TimelineProvider {
 // プレースホルダー（読み込み中の仮表示）
 func placeholder(in context: Context) -> QuoteEntry {
 QuoteEntry(
 date: Date(),
 quote: Quote(id: 0, text: "読み込み中...", author: "")
 )
 }
 
 // スナップショット（ウィジェットギャラリーでのプレビュー）
 func getSnapshot(in context: Context, completion: @escaping (QuoteEntry) -> Void) {
 let entry = QuoteEntry(
 date: Date(),
 quote: QuoteStore.todaysQuote()
 )
 completion(entry)
 }
 
 // タイムライン（実際のウィジェット更新スケジュール）
 func getTimeline(in context: Context, completion: @escaping (Timeline<QuoteEntry>) -> Void) {
 let currentDate = Date()
 let quote = QuoteStore.todaysQuote()
 let entry = QuoteEntry(date: currentDate, quote: quote)
 
 // 次の日の0時にウィジェットを更新
 let tomorrow = Calendar.current.startOfDay(
 for: Calendar.current.date(byAdding: .day, value: 1, to: currentDate)!
 )
 
 let timeline = Timeline(entries: [entry], policy: .after(tomorrow))
 completion(timeline)
 }
 }
 
 // MARK: - ウィジェットのビュー
 
 struct QuoteWidgetEntryView: View {
 var entry: QuoteProvider.Entry
 @Environment(\.widgetFamily) var family
 
 var body: some View {
 switch family {
 case .systemSmall:
 smallWidget
 case .systemMedium:
 mediumWidget
 default:
 mediumWidget
 }
 }
 
 // 小サイズ
 var smallWidget: some View {
 VStack(spacing: 4) {
 Image(systemName: "quote.opening")
 .font(.caption)
 .foregroundStyle(.blue)
 
 Text(entry.quote.text)
 .font(.caption)
 .bold()
 .multilineTextAlignment(.center)
 .lineLimit(3)
 
 Text(entry.quote.author)
 .font(.caption2)
 .foregroundStyle(.secondary)
 }
 .padding(12)
 }
 
 // 中サイズ
 var mediumWidget: some View {
 HStack(spacing: 16) {
 Image(systemName: "quote.opening")
 .font(.title)
 .foregroundStyle(.blue)
 
 VStack(alignment: .leading, spacing: 4) {
 Text("今日の名言")
 .font(.caption2)
 .foregroundStyle(.secondary)
 
 Text(entry.quote.text)
 .font(.subheadline)
 .bold()
 
 Text("— \(entry.quote.author)")
 .font(.caption)
 .foregroundStyle(.secondary)
 }
 
 Spacer()
 }
 .padding()
 }
 }
 
 // MARK: - ウィジェット定義
 
 @main
 struct QuoteWidget: Widget {
 let kind: String = "QuoteWidget"
 
 var body: some WidgetConfiguration {
 StaticConfiguration(kind: kind, provider: QuoteProvider()) { entry in
 QuoteWidgetEntryView(entry: entry)
 .containerBackground(.fill.tertiary, for: .widget)
 }
 .configurationDisplayName("今日の名言")
 .description("日替わりで名言を表示します")
 .supportedFamilies([.systemSmall, .systemMedium])
 }
 }
 
 // MARK: - プレビュー
 
 #Preview(as: .systemMedium) {
 QuoteWidget()
 } timeline: {
 QuoteEntry(date: .now, quote: QuoteStore.todaysQuote())
 }


```

**このアプリは何をするものか：**

このアプリは、毎日異なる名言をホーム画面のウィジェットに表示するアプリです。

あらかじめ登録されている名言の中から、その日の名言を自動で選び、作者名と一緒に表示します。

ウィジェットは毎日0時に自動更新されるため、ホーム画面を見るだけで毎日新しい名言を楽しむことができます。

また、ウィジェットのサイズ（小・中）に応じて表示レイアウトが切り替わり、見やすいデザインで名言を確認できるようになっています。

## コードの詳細解説

### TimelineProviderの仕組み

```swift
struct QuoteProvider: TimelineProvider {
 // プレースホルダー（読み込み中の仮表示）
 func placeholder(in context: Context) -> QuoteEntry {
 QuoteEntry(
 date: Date(),
 quote: Quote(id: 0, text: "読み込み中...", author: "")
 )
 }
 
 // スナップショット（ウィジェットギャラリーでのプレビュー）
 func getSnapshot(in context: Context, completion: @escaping (QuoteEntry) -> Void) {
 let entry = QuoteEntry(
 date: Date(),
 quote: QuoteStore.todaysQuote()
 )
 completion(entry)
 }
 
 // タイムライン（実際のウィジェット更新スケジュール）
 func getTimeline(in context: Context, completion: @escaping (Timeline<QuoteEntry>) -> Void) {
 let currentDate = Date()
 let quote = QuoteStore.todaysQuote()
 let entry = QuoteEntry(date: currentDate, quote: quote)
 
 // 次の日の0時にウィジェットを更新
 let tomorrow = Calendar.current.startOfDay(
 for: Calendar.current.date(byAdding: .day, value: 1, to: currentDate)!
 )
 
 let timeline = Timeline(entries: [entry], policy: .after(tomorrow))
 completion(timeline)
 }
 }
```

**何をしているか：**

TimelineProviderは、ウィジェットに「どのデータを表示するか」と「いつ表示内容を更新するか」を管理する仕組みです。

このコードでは、QuoteProviderという構造体がTimelineProviderに準拠しています。TimelineProviderを使用する場合は、基本的にplaceholder、getSnapshot、getTimelineの3つのメソッドを実装します。

placeholderは、ウィジェットのデータがまだ準備できていないときに表示する仮の内容を返します。このアプリでは、「読み込み中...」という文字を表示するQuoteEntryを作成しています。

getSnapshotは、ウィジェットを追加するときのギャラリーやプレビュー画面で使用するデータを返します。QuoteStore.todaysQuote()を呼び出し、その日の名言を取得してQuoteEntryに入れています。作成したエントリは、completion(entry)によってWidgetKit側に渡されます。

getTimelineは、実際にホーム画面へ表示するデータと、次にウィジェットを更新する日時を設定します。現在日時を取得し、その日の名言をQuoteEntryに入れています。その後、Calendarを使って明日の0時を計算し、.after(tomorrow)を指定しています。これにより、明日の0時以降にWidgetKitが新しいタイムラインを取得し、次の日の名言に更新します。

**なぜこう書くのか：**

通常のSwiftUIアプリは、ユーザーが操作したり、データが変更されたりすると画面がすぐに更新されます。しかし、ウィジェットはアプリのように常に動き続けているわけではありません。

そのため、ウィジェットではTimelineProviderを使って、あらかじめ表示データと更新予定をWidgetKitに渡す必要があります。

このアプリでは、名言を1日ごとに変更したいため、更新タイミングを明日の0時に設定しています。

policy: .after(tomorrow)

と書くことで、「明日の0時を過ぎたら、新しいタイムラインを要求してください」という予定をWidgetKitに伝えています。

また、completionを使って結果を返すのは、WidgetKitが必要なタイミングで処理を呼び出し、データの準備が完了した後に結果を受け取る仕組みになっているためです。

getTimelineが直接Timelineをreturnするのではなく、次のように書く必要があります。

completion(timeline)

この書き方によって、将来Web APIなどから非同期でデータを取得する場合にも対応できます.

**もしこう書かなかったら：**

TimelineProviderを実装しなかった場合、WidgetKitはウィジェットに表示するデータや更新タイミングを判断できません。そのため、ウィジェットを正しく作成できず、コンパイルエラーになる可能性があります。

placeholderを書かなかった場合、TimelineProviderに必要なメソッドが不足するため、プロトコルに準拠していないというエラーが発生します。また、データの読み込み中に表示する内容がなくなります。

getSnapshotを書かなかった場合も、TimelineProviderへの準拠に必要な処理が不足します。ウィジェットギャラリーやプレビューで、サンプルの表示内容を作れなくなります。

getTimelineを書かなかった場合、実際のウィジェットに表示するデータと更新予定を渡せないため、ウィジェットを正常に動作させることができません。

次の処理を書かなかった場合、

completion(timeline)

作成したタイムラインがWidgetKit側に渡されません。そのため、処理が完了せず、ウィジェットにデータが表示されない可能性があります。

また、更新ポリシーを次のように変更すると、

policy: .never

WidgetKitによる自動更新が行われなくなります。そのため、日付が変わっても名言が自動的に切り替わらず、アプリ側から更新を要求するまで同じ名言が表示され続ける可能性があります。

反対に、更新時間を現在時刻に近すぎる時間へ設定しても、ウィジェットは指定した時刻どおりに必ず更新されるとは限りません。WidgetKitがバッテリー消費などを考慮し、実際の更新タイミングを調整するためです。

このコードでは、毎日内容が変わる名言アプリに合わせて、明日の0時を次の更新予定として設定しています。

---

### TimelineEntryとウィジェットビュー

```swift
// MARK: - タイムラインエントリ

struct QuoteEntry: TimelineEntry {
    let date: Date
    let quote: Quote
}

// MARK: - ウィジェットのビュー

struct QuoteWidgetEntryView: View {
    var entry: QuoteProvider.Entry

    @Environment(\.widgetFamily) var family

    var body: some View {
        switch family {
        case .systemSmall:
            smallWidget

        case .systemMedium:
            mediumWidget

        default:
            mediumWidget
        }
    }

    // 小サイズ
    var smallWidget: some View {
        VStack(spacing: 4) {
            Image(systemName: "quote.opening")
                .font(.caption)
                .foregroundStyle(.blue)

            Text(entry.quote.text)
                .font(.caption)
                .bold()
                .multilineTextAlignment(.center)
                .lineLimit(3)

            Text(entry.quote.author)
                .font(.caption2)
                .foregroundStyle(.secondary)
        }
        .padding(12)
    }

    // 中サイズ
    var mediumWidget: some View {
        HStack(spacing: 16) {
            Image(systemName: "quote.opening")
                .font(.title)
                .foregroundStyle(.blue)

            VStack(alignment: .leading, spacing: 4) {
                Text("今日の名言")
                    .font(.caption2)
                    .foregroundStyle(.secondary)

                Text(entry.quote.text)
                    .font(.subheadline)
                    .bold()

                Text("— \(entry.quote.author)")
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }

            Spacer()
        }
        .padding()
    }
}
```

**何をしているか：**

QuoteEntryは、ウィジェットに表示する1回分のデータをまとめる構造体です。

struct QuoteEntry: TimelineEntry

と書くことで、QuoteEntryがWidgetKitのTimelineEntryプロトコルに準拠することを表しています。

この構造体には、次の2つのデータがあります。

let date: Date
let quote: Quote

dateは、そのデータをウィジェットに表示する日時です。quoteには、表示する名言の文章や作者名が入っています。

例えば、次のようなデータが1つのQuoteEntryになります。

```swift
QuoteEntry(
    date: Date(),
    quote: Quote(
        id: 1,
        text: "失敗は成功のもと",
        author: "ことわざ"
    )
)
```

QuoteWidgetEntryViewは、QuoteEntryに入っているデータを実際のウィジェット画面に表示するためのビューです。

var entry: QuoteProvider.Entry

このentryには、QuoteProviderが作成したQuoteEntryが渡されます。

画面には、次のようにして名言の文章を表示しています。

Text(entry.quote.text)

作者名は、次のように表示しています。

Text(entry.quote.author)

@Environment(\.widgetFamily) var family

によって、現在表示されているウィジェットのサイズを取得しています。

その後、switch文を使って、小サイズの場合はsmallWidget、中サイズの場合はmediumWidgetを表示しています。

```swift
switch family {
case .systemSmall:
    smallWidget

case .systemMedium:
    mediumWidget

default:
    mediumWidget
}
```

小サイズでは縦方向に配置するため、VStackを使用しています。中サイズでは、アイコンと名言を横方向に並べるため、HStackを使用しています。

**なぜこう書くのか：**

TimelineEntryとウィジェットビューを分けて作ることで、表示するデータと画面の見た目をそれぞれ独立して管理できます。

QuoteEntryは「いつ・どの名言を表示するか」というデータだけを持ち、QuoteWidgetEntryViewはそのデータをどのようなレイアウトで表示するかを担当しています。

このように役割を分けることで、表示する名言が変わってもビューのレイアウトを変更する必要がなく、保守しやすいコードになります。

また、@Environment(\.widgetFamily)を使用することで、現在のウィジェットのサイズを取得し、小サイズと中サイズでそれぞれ最適なレイアウトを表示できます。

サイズごとに表示方法を切り替えることで、文字が見切れたり、余白が不自然になったりすることを防ぎ、見やすいウィジェットを作ることができます。

**もしこう書かなかったら：**

QuoteEntryをTimelineEntryとして作成しなかった場合、WidgetKitは表示するデータとして認識できず、ウィジェットを正常に動作させることができません。

また、QuoteWidgetEntryViewがなければ、QuoteEntryにデータが入っていても画面に表示する方法がないため、ウィジェットには何も表示されません。

さらに、@Environment(\.widgetFamily)やswitch familyを書かなかった場合、小サイズと中サイズで同じレイアウトを使用することになります。

その結果、小サイズでは文字が途中で切れたり、中サイズでは余白が多くなったりして、見た目や使いやすさが悪くなる可能性があります。

---

### ウィジェットサイズごとのレイアウト

```swift
@Environment(\.widgetFamily) var family

var body: some View {
    switch family {
    case .systemSmall:
        smallWidget

    case .systemMedium:
        mediumWidget

    default:
        mediumWidget
    }
}

// 小サイズ
var smallWidget: some View {
    VStack(spacing: 4) {
        Image(systemName: "quote.opening")
            .font(.caption)
            .foregroundStyle(.blue)

        Text(entry.quote.text)
            .font(.caption)
            .bold()
            .multilineTextAlignment(.center)
            .lineLimit(3)

        Text(entry.quote.author)
            .font(.caption2)
            .foregroundStyle(.secondary)
    }
    .padding(12)
}

// 中サイズ
var mediumWidget: some View {
    HStack(spacing: 16) {
        Image(systemName: "quote.opening")
            .font(.title)
            .foregroundStyle(.blue)

        VStack(alignment: .leading, spacing: 4) {
            Text("今日の名言")
                .font(.caption2)
                .foregroundStyle(.secondary)

            Text(entry.quote.text)
                .font(.subheadline)
                .bold()

            Text("— \(entry.quote.author)")
                .font(.caption)
                .foregroundStyle(.secondary)
        }

        Spacer()
    }
    .padding()
}
```

**何をしているか：**

このコードは、ウィジェットのサイズに合わせて表示レイアウトを切り替えています。

@Environment(\.widgetFamily)を使って、現在のウィジェットが小サイズか中サイズかを取得しています。

その後、switch familyでサイズを確認し、小サイズの場合はsmallWidget、中サイズの場合はmediumWidgetを表示しています。

小サイズでは、VStackを使ってアイコン、名言、作者名を縦方向に並べています。中サイズでは、HStackを使ってアイコンと名言の部分を横方向に並べています。

**なぜこう書くのか：**

ウィジェットはサイズによって表示できる範囲が異なるため、それぞれのサイズに合ったレイアウトを作る必要があります。

小サイズは横幅が狭いため、内容を縦に並べることで見やすくしています。また、名言が長すぎないように.lineLimit(3)を設定しています。

中サイズは横幅が広いため、アイコンと文章を横に並べることで、スペースを有効に使っています。

また、smallWidgetとmediumWidgetを別々のプロパティに分けることで、コードが読みやすくなり、サイズごとのデザインも修正しやすくなります。

**もしこう書かなかったら：**

ウィジェットサイズごとのレイアウトを分けなかった場合、すべてのサイズで同じ表示になります。

例えば、中サイズ用の横長レイアウトを小サイズで表示すると、文字が入りきらなかったり、作者名が見えなくなったりする可能性があります。

反対に、小サイズ用の縦並びレイアウトを中サイズで表示すると、横のスペースが余ってしまい、見た目のバランスが悪くなる可能性があります。

また、.lineLimit(3)を書かなかった場合、長い名言が多くの行に表示され、作者名が画面外に押し出されることがあります。

---

### メインアプリとの連携

```swift
// 今日の名言を取得
let quote = QuoteStore.todaysQuote()

// タイムラインエントリを作成
let entry = QuoteEntry(
    date: Date(),
    quote: quote
)
```

**何をしているか：**

このコードでは、メインアプリとウィジェットで共通のQuoteStoreを利用して、その日に表示する名言を取得しています。

QuoteStore.todaysQuote()を呼び出すことで、その日の名言を取得し、それをQuoteEntryに保存しています。QuoteEntryに保存されたデータは、QuoteWidgetEntryViewへ渡され、ホーム画面のウィジェットに表示されます。

また、QuoteとQuoteStoreをメインアプリとウィジェットの両方で共有しているため、同じデータを使用できます。

**なぜこう書くのか：**

メインアプリとウィジェットで別々に名言のデータを作成すると、同じ内容を2か所で管理しなければならず、修正するときに両方を書き換える必要があります。

そのため、QuoteとQuoteStoreを共通ファイルとして管理し、メインアプリとウィジェットの両方から利用できるようにしています。

こうすることで、名言を追加・修正するときはQuoteStoreだけを変更すればよくなり、コードの重複を防ぐことができます。

**もしこう書かなかったら：**

QuoteStore.todaysQuote()を使わなかった場合、ウィジェットは表示する名言を取得できず、毎日の名言を表示できません。

また、QuoteやQuoteStoreをウィジェット側だけ、またはメインアプリ側だけに置いた場合、もう一方のターゲットからそのコードを利用できず、**「Cannot find 'QuoteStore' in scope」**などのコンパイルエラーが発生します。

さらに、メインアプリ用とウィジェット用で別々に名言データを管理すると、片方だけ内容が更新されてしまう可能性があり、アプリとウィジェットで異なる名言が表示されるなど、不整合が起こる原因になります。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`TimelineProvider` | ウィジェットを更新するタイミングとコンテンツを定義 | `struct QuoteProvider: TimelineProvider { ... }` |
| 例：`@main` + `WidgetConfiguration` | ウィジェットのエントリーポイント | `@main struct QuoteWidget: Widget { ... }` |
| 例：`TimelineEntry`|　ウィジェットに表示する1回分のデータを表す | `struct QuoteEntry: TimelineEntry`　|
| 例：`Timeline`| 表示するデータと更新スケジュールを管理する| `Timeline(entries: [entry], policy: .after(tomorrow))` |
| 例：`placeholder()`| データ読み込み中に表示する仮の内容を設定する | `func placeholder(in: Context)`|
| 例：`getSnapshot()`| ウィジェットギャラリー用の表示データを返す | `func getSnapshot(in:completion:)`|
| 例：`getTimeline()`| 実際に表示するデータと更新日時を設定する | `func getTimeline(in:completion:)`|
| 例：`@Environment(\.widgetFamily)`| 現在のウィジェットサイズを取得する | `@Environment(\.widgetFamily) var family` |
| 例：`switch`| サイズによって表示内容を切り替える| `switch family { ... }`|
| 例：`Calendar.current`| 日付を計算するためのカレンダーを取得する | `Calendar.current.startOfDay(for: tomorrow)`|
| 例：`StaticConfiguration`| ウィジェットの基本設定を行う| `StaticConfiguration(kind: kind, provider: QuoteProvider())` |
| 例：`@main + Widget`| ウィジェットの開始位置を定義する | @main struct QuoteWidget: Widget |
| 例：`.configurationDisplayName()`| ウィジェットの名前を設定する |`.configurationDisplayName("今日の名言")`|
| 例：`.description()`| ウィジェットの説明文を設定する | `.description("日替わりで名言を表示します")` |
| 例：`.supportedFamilies()`| 対応するウィジェットサイズを指定する | `.supportedFamilies([.systemSmall, .systemMedium])`|
| 例：`.containerBackground()`| ウィジェット専用の背景を設定する | `.containerBackground(.fill.tertiary, for: .widget)`|
| 例：``| | |
| 例：``| | |
| 例：``| | |
| 例：``| | |

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
