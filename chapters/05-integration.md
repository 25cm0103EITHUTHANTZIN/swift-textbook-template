# 第5章：機能統合の実践

> 執筆者： イトゥタンジン
> 最終更新：2026-06-05

## この章で学ぶこと

（この章で扱うトピックの概要を2〜3行で書く。自分の言葉で。）

今まで別々に学んできたカメラ・写真選択、地図、位置情報、データ保存を1つのアプリにまとめて使う方法を学ぶ。

写真を撮った場所を記録し、後から地図や一覧で確認できるアプリを作りながら、実際のアプリ開発に近い機能の組み合わせ方を理解する。

## 模範コードの全体像

```swift
// ============================================
// 第5章：写真 + 地図 + データ保存の統合アプリ
// ============================================
// 写真を選択し、選択時の現在地を地図上に記録する
// 「フォトマップ」アプリです。
// 第2〜4章で学んだ技術を組み合わせて使います。
//
// 【注意】Info.plist に以下のキーを追加してください：
//   - NSLocationWhenInUseUsageDescription
//   - NSPhotoLibraryAddUsageDescription
// ============================================

import SwiftUI
import SwiftData
import MapKit
import PhotosUI

// MARK: - データモデル

@Model
class PhotoRecord {
    var title: String
    var memo: String
    var latitude: Double
    var longitude: Double
    var imageData: Data?
    var createdAt: Date
    
    init(title: String, memo: String = "", latitude: Double, longitude: Double, imageData: Data? = nil) {
        self.title = title
        self.memo = memo
        self.latitude = latitude
        self.longitude = longitude
        self.imageData = imageData
        self.createdAt = .now
    }
    
    var coordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    }
    
    var uiImage: UIImage? {
        guard let data = imageData else { return nil }
        return UIImage(data: data)
    }
}

// MARK: - 位置情報マネージャー

@Observable
class LocationManager: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    var currentLocation: CLLocationCoordinate2D?
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
        manager.requestWhenInUseAuthorization()
        manager.startUpdatingLocation()
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        currentLocation = locations.last?.coordinate
    }
}

// MARK: - アプリエントリポイント
// ※ App ファイルに以下を記述：
//
// @main
// struct PhotoMapApp: App {
//     var body: some Scene {
//         WindowGroup {
//             ContentView()
//         }
//         .modelContainer(for: PhotoRecord.self)
//     }
// }

// MARK: - メインビュー（タブ構成）

struct ContentView: View {
    var body: some View {
        TabView {
            MapTab()
                .tabItem {
                    Label("マップ", systemImage: "map")
                }
            
            ListTab()
                .tabItem {
                    Label("一覧", systemImage: "list.bullet")
                }
        }
    }
}

// MARK: - マップタブ

struct MapTab: View {
    @Environment(\.modelContext) private var modelContext
    @Query private var records: [PhotoRecord]
    @State private var locationManager = LocationManager()
    @State private var cameraPosition: MapCameraPosition = .automatic
    @State private var isShowingAddSheet = false
    @State private var selectedRecord: PhotoRecord?
    
    var body: some View {
        NavigationStack {
            ZStack(alignment: .bottomTrailing) {
                Map(position: $cameraPosition) {
                    UserAnnotation()
                    
                    ForEach(records) { record in
                        Annotation(record.title, coordinate: record.coordinate) {
                            Button {
                                selectedRecord = record
                            } label: {
                                if let uiImage = record.uiImage {
                                    Image(uiImage: uiImage)
                                        .resizable()
                                        .aspectRatio(contentMode: .fill)
                                        .frame(width: 40, height: 40)
                                        .clipShape(Circle())
                                        .overlay(Circle().stroke(.white, lineWidth: 2))
                                        .shadow(radius: 2)
                                } else {
                                    Image(systemName: "photo.circle.fill")
                                        .font(.title)
                                        .foregroundStyle(.blue)
                                }
                            }
                        }
                    }
                }
                .mapControls {
                    MapUserLocationButton()
                }
                
                // 追加ボタン
                Button {
                    isShowingAddSheet = true
                } label: {
                    Image(systemName: "plus.circle.fill")
                        .font(.system(size: 56))
                        .foregroundStyle(.blue)
                        .background(Circle().fill(.white))
                        .shadow(radius: 4)
                }
                .padding(24)
            }
            .navigationTitle("フォトマップ")
            .sheet(isPresented: $isShowingAddSheet) {
                AddRecordView(locationManager: locationManager)
            }
            .sheet(item: $selectedRecord) { record in
                RecordDetailView(record: record)
            }
        }
    }
}

// MARK: - 一覧タブ

struct ListTab: View {
    @Environment(\.modelContext) private var modelContext
    @Query(sort: \PhotoRecord.createdAt, order: .reverse) private var records: [PhotoRecord]
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(records) { record in
                    HStack(spacing: 12) {
                        if let uiImage = record.uiImage {
                            Image(uiImage: uiImage)
                                .resizable()
                                .aspectRatio(contentMode: .fill)
                                .frame(width: 50, height: 50)
                                .clipShape(RoundedRectangle(cornerRadius: 8))
                        }
                        
                        VStack(alignment: .leading, spacing: 4) {
                            Text(record.title)
                                .font(.headline)
                            Text(record.createdAt, style: .date)
                                .font(.caption)
                                .foregroundStyle(.secondary)
                        }
                    }
                }
                .onDelete { offsets in
                    for index in offsets {
                        modelContext.delete(records[index])
                    }
                }
            }
            .navigationTitle("記録一覧")
        }
    }
}

// MARK: - 記録追加画面

struct AddRecordView: View {
    @Environment(\.modelContext) private var modelContext
    @Environment(\.dismiss) private var dismiss
    let locationManager: LocationManager
    
    @State private var title = ""
    @State private var memo = ""
    @State private var selectedItem: PhotosPickerItem?
    @State private var selectedImageData: Data?
    @State private var previewImage: Image?
    
    var body: some View {
        NavigationStack {
            Form {
                Section("写真") {
                    if let image = previewImage {
                        image
                            .resizable()
                            .aspectRatio(contentMode: .fit)
                            .frame(maxHeight: 200)
                            .clipShape(RoundedRectangle(cornerRadius: 8))
                    }
                    
                    PhotosPicker(selection: $selectedItem, matching: .images) {
                        Label("写真を選択", systemImage: "photo")
                    }
                }
                
                Section("情報") {
                    TextField("タイトル", text: $title)
                    TextField("メモ（任意）", text: $memo, axis: .vertical)
                        .lineLimit(3...6)
                }
                
                Section("位置情報") {
                    if let location = locationManager.currentLocation {
                        Text("緯度: \(location.latitude, specifier: "%.4f")")
                        Text("経度: \(location.longitude, specifier: "%.4f")")
                    } else {
                        Text("位置情報を取得中...")
                            .foregroundStyle(.secondary)
                    }
                }
            }
            .navigationTitle("新しい記録")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("キャンセル") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("保存") {
                        saveRecord()
                    }
                    .disabled(title.isEmpty || locationManager.currentLocation == nil)
                }
            }
            .onChange(of: selectedItem) { _, newItem in
                Task {
                    if let data = try? await newItem?.loadTransferable(type: Data.self) {
                        selectedImageData = data
                        if let uiImage = UIImage(data: data) {
                            previewImage = Image(uiImage: uiImage)
                        }
                    }
                }
            }
        }
    }
    
    func saveRecord() {
        guard let location = locationManager.currentLocation else { return }
        
        let record = PhotoRecord(
            title: title,
            memo: memo,
            latitude: location.latitude,
            longitude: location.longitude,
            imageData: selectedImageData
        )
        modelContext.insert(record)
        dismiss()
    }
}

// MARK: - 記録詳細画面

struct RecordDetailView: View {
    let record: PhotoRecord
    
    var body: some View {
        ScrollView {
            VStack(spacing: 16) {
                if let uiImage = record.uiImage {
                    Image(uiImage: uiImage)
                        .resizable()
                        .aspectRatio(contentMode: .fit)
                        .clipShape(RoundedRectangle(cornerRadius: 12))
                }
                
                VStack(alignment: .leading, spacing: 8) {
                    Text(record.title)
                        .font(.title2)
                        .bold()
                    
                    if !record.memo.isEmpty {
                        Text(record.memo)
                            .foregroundStyle(.secondary)
                    }
                    
                    Text(record.createdAt, style: .date)
                        .font(.caption)
                        .foregroundStyle(.tertiary)
                }
                .frame(maxWidth: .infinity, alignment: .leading)
                
                // ミニマップ
                Map {
                    Marker(record.title, coordinate: record.coordinate)
                }
                .frame(height: 200)
                .clipShape(RoundedRectangle(cornerRadius: 12))
            }
            .padding()
        }
    }
}

#Preview {
    ContentView()
        .modelContainer(for: PhotoRecord.self, inMemory: true)
}

```

**このアプリは何をするものか：**

このアプリは、選択した写真とその場所の位置情報を保存し、地図や一覧で確認できるフォトマップアプリである。

写真・メモ・位置情報をまとめて管理し、思い出や訪れた場所を記録できる。

## コードの詳細解説

### データモデルの設計

```swift
@Model
class PhotoRecord {
    var title: String
    var memo: String
    var latitude: Double
    var longitude: Double
    var imageData: Data?
    var createdAt: Date
    
    init(title: String, memo: String = "", latitude: Double, longitude: Double, imageData: Data? = nil) {
        self.title = title
        self.memo = memo
        self.latitude = latitude
        self.longitude = longitude
        self.imageData = imageData
        self.createdAt = .now
    }
    
    var coordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    }
    
    var uiImage: UIImage? {
        guard let data = imageData else { return nil }
        return UIImage(data: data)
    }
}
```

**何をしているか：**

このコードは、写真記録アプリで保存する1件分のデータを表している。

title は記録のタイトル、memo はメモ、latitude と longitude は位置情報、imageData は写真データ、createdAt は作成日時を保存するためのプロパティである。

また、coordinate では地図で使える座標に変換し、uiImage では保存した画像データを画面に表示できる画像に変換している。

**なぜこう書くのか：**

@Model を付けることで、このクラスを SwiftData で保存できるデータとして扱えるようにしている。

写真と位置情報を一緒に保存するため、タイトル・メモ・緯度・経度・画像データ・作成日時を1つのモデルにまとめている。

また、地図表示では CLLocationCoordinate2D が必要なので、latitude と longitude から coordinate を作る計算プロパティを用意している。画像も Data のままでは表示できないため、UIImage に変換する uiImage を作っている。

**もしこう書かなかったら：**

@Model を付けなければ、SwiftDataでこの記録を保存できなくなる。

latitude と longitude がなければ、写真をどこで記録したのか地図上に表示できない。

imageData がなければ、写真を保存したり一覧・詳細画面で表示したりできない。

また、coordinate や uiImage がない場合、地図や画像表示のたびに変換処理を書く必要があり、コードが長くなって分かりにくくなる。

---

### タブ構成の設計

```swift
struct ContentView: View {
    var body: some View {
        TabView {
            MapTab()
                .tabItem {
                    Label("マップ", systemImage: "map")
                }
            
            ListTab()
                .tabItem {
                    Label("一覧", systemImage: "list.bullet")
                }
        }
    }
}
```

**何をしているか：**

このコードはアプリ全体の画面構成を作っている。

TabView を使って「マップ」と「一覧」の2つの画面をタブで切り替えられるようにしている。

- MapTab() → 地図画面
- ListTab() → 保存した記録の一覧画面

tabItem では各タブの名前とアイコンを設定している。

**なぜこう書くのか：**

このアプリには、

1. 地図で記録を確認する機能
2. 一覧で記録を確認する機能

の2つの主要な機能がある。

そのため TabView を使うことで、ユーザーが画面を切り替えながら利用できる。

また、タブバーはiPhoneアプリでよく使われるUIなので、ユーザーが直感的に操作しやすい。

**もしこう書かなかったら：**

TabView を使わなければ、地図画面と一覧画面を簡単に切り替えることができなくなる。

例えば NavigationStack だけで実装すると、一覧を見るたびに別画面へ移動して戻る操作が必要になる。

また、tabItem を設定しなければ、タブに名前やアイコンが表示されず、どの画面なのか分かりにくくなってしまう。

実際に ListTab() を削除して試すと、一覧画面が表示されなくなり、保存した記録を地図からしか確認できなくなる。

---

### カメラと位置情報の連携

```swift
@State private var selectedItem: PhotosPickerItem?
@State private var selectedImageData: Data?

PhotosPicker(selection: $selectedItem, matching: .images) {
    Label("写真を選択", systemImage: "photo")
}

func saveRecord() {
    guard let location = locationManager.currentLocation else { return }

    let record = PhotoRecord(
        title: title,
        memo: memo,
        latitude: location.latitude,
        longitude: location.longitude,
        imageData: selectedImageData
    )

    modelContext.insert(record)
}
```

**何をしているか：**

このコードは、ユーザーが選択した写真と現在地の位置情報を一緒に保存している。

PhotosPicker を使ってフォトライブラリから写真を選択し、その画像データを selectedImageData に保存する。

また、LocationManager が取得した現在地の緯度・経度を利用して、写真と位置情報を1つの PhotoRecord として保存している。

その結果、後から地図上で写真を保存した場所を確認できるようになる。

**なぜこう書くのか：**

このアプリは単なる写真保存アプリではなく、「どこで撮った写真なのか」を記録するフォトマップアプリである。

そのため、写真データだけでなく位置情報も同時に保存する必要がある。

また、位置情報が取得できていることを確認するために guard let を使い、安全に保存処理を行っている。

**もしこう書かなかったら：**

位置情報を保存しなければ、写真は保存できても地図上にマーカーを表示できなくなる。

写真データを保存しなければ、位置情報だけが残り、どんな写真だったのか確認できなくなる。

```swift
guard let location = locationManager.currentLocation else { return }
```
を削除すると、位置情報が取得できていない状態でも保存処理が実行されてしまい、正しい位置情報を保存できなくなる可能性がある。

---

### SwiftDataでの画像保存

```swift
var imageData: Data?

init(
    title: String,
    memo: String = "",
    latitude: Double,
    longitude: Double,
    imageData: Data? = nil
) {
    self.title = title
    self.memo = memo
    self.latitude = latitude
    self.longitude = longitude
    self.imageData = imageData
    self.createdAt = .now
}
```
```swift
@State private var selectedImageData: Data?
```
```swift
if let data = try? await newItem?.loadTransferable(type: Data.self) {
    selectedImageData = data
}
```
```swift
let record = PhotoRecord(
    title: title,
    memo: memo,
    latitude: location.latitude,
    longitude: location.longitude,
    imageData: selectedImageData
)

modelContext.insert(record)
```

**何をしているか：**

このコードは、ユーザーが選択した写真を Data 型に変換し、SwiftDataを使って保存している。

まず PhotosPicker で選択した画像を Data として取得し、selectedImageData に保存する。

その後、PhotoRecord の imageData プロパティに渡し、modelContext.insert() を使ってSwiftDataのデータベースへ保存している。

**なぜこう書くのか：**

SwiftDataは UIImage をそのまま保存できないため、画像を保存可能な Data 型へ変換する必要がある。

画像を Data として保存しておけば、アプリを終了しても画像が消えず、次回起動時にも同じ写真を表示できる。

また、Data? としているため、写真を選択しなかった場合でもエラーにならずに記録を保存できる。

**もしこう書かなかったら：**

imageData を用意しなければ、写真を保存できなくなる。

また、画像を Data に変換せず UIImage のまま保存しようとすると、SwiftDataが保存できずエラーになる。

```swift
modelContext.insert(record)
```
を書かなければ、PhotoRecord オブジェクトは作成されてもデータベースには保存されない。そのためアプリを再起動すると記録が消えてしまう。

このコードによって、選択した写真をSwiftDataへ永続化し、いつでも表示できるようにしている。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`TabView` | 複数のビューをタブで切り替えるコンポーネント | `TabView { ... }.tabViewStyle(.page)` |
| 例：`CLLocationManager` | GPS位置情報を取得するAPIManager | `let location = manager.location?.coordinate` |
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
