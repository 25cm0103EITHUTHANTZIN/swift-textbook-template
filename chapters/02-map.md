# 第2章：地図アプリの基本

> 執筆者：イトゥタンジン
> 最終更新：2026-05-13

## この章で学ぶこと

この章では、SwiftUIとMapKitを使って地図アプリを作成し、東京の観光スポットを地図上に表示する方法を学ぶ。

また、マーカー表示やカテゴリフィルター機能を実装し、ユーザーが場所を見つけやすくする方法について理解を深める。

## 模範コードの全体像

```swift
// ============================================
// 第2章（基本）：MapKitで地図を表示するアプリ
// ============================================
// 東京の観光スポットを地図上にマーカーで表示します。
// マーカーをタップすると詳細情報が表示されます。
// ============================================

import SwiftUI
import MapKit

// MARK: - データモデル

struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let description: String
    let coordinate: CLLocationCoordinate2D
    let category: Category
    
    enum Category: String, CaseIterable {
        case temple = "寺社"
        case tower = "タワー"
        case park = "公園"
        
        var iconName: String {
            switch self {
            case .temple: return "building.columns"
            case .tower: return "antenna.radiowaves.left.and.right"
            case .park: return "leaf"
            }
        }
        
        var color: Color {
            switch self {
            case .temple: return .red
            case .tower: return .blue
            case .park: return .green
            }
        }
    }
}

// MARK: - サンプルデータ

extension Landmark {
    static let sampleData: [Landmark] = [
        Landmark(
            name: "浅草寺",
            description: "東京都内最古の寺院。雷門が有名。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7148, longitude: 139.7967),
            category: .temple
        ),
        Landmark(
            name: "東京タワー",
            description: "1958年に完成した高さ333mの電波塔。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6586, longitude: 139.7454),
            category: .tower
        ),
        Landmark(
            name: "東京スカイツリー",
            description: "高さ634mの世界一高い自立式電波塔。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7101, longitude: 139.8107),
            category: .tower
        ),
        Landmark(
            name: "明治神宮",
            description: "明治天皇と昭憲皇太后を祀る神社。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6764, longitude: 139.6993),
            category: .temple
        ),
        Landmark(
            name: "上野恩賜公園",
            description: "美術館や動物園がある広大な公園。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7146, longitude: 139.7732),
            category: .park
        ),
        Landmark(
            name: "新宿御苑",
            description: "都心にある広さ58.3ヘクタールの庭園。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6852, longitude: 139.7100),
            category: .park
        ),
    ]
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var cameraPosition: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
            span: MKCoordinateSpan(latitudeDelta: 0.08, longitudeDelta: 0.08)
        )
    )
    @State private var selectedLandmark: Landmark?
    @State private var selectedCategories: Set<Landmark.Category> = Set(Landmark.Category.allCases)
    
    var filteredLandmarks: [Landmark] {
        Landmark.sampleData.filter { selectedCategories.contains($0.category) }
    }
    
    var body: some View {
        ZStack(alignment: .bottom) {
            // 地図
            Map(position: $cameraPosition) {
                ForEach(filteredLandmarks) { landmark in
                    Marker(
                        landmark.name,
                        systemImage: landmark.category.iconName,
                        coordinate: landmark.coordinate
                    )
                    .tint(landmark.category.color)
                }
            }
            .mapStyle(.standard(elevation: .realistic))
            
            // カテゴリフィルター
            VStack(spacing: 8) {
                if let landmark = selectedLandmark {
                    LandmarkCard(landmark: landmark)
                        .transition(.move(edge: .bottom))
                }
                
                CategoryFilter(selectedCategories: $selectedCategories)
            }
            .padding()
        }
        .onMapCameraChange { context in
            // 地図の操作に応じた処理を追加できる
        }
    }
}

// MARK: - カテゴリフィルター

struct CategoryFilter: View {
    @Binding var selectedCategories: Set<Landmark.Category>
    
    var body: some View {
        HStack(spacing: 8) {
            ForEach(Landmark.Category.allCases, id: \.self) { category in
                Button {
                    if selectedCategories.contains(category) {
                        selectedCategories.remove(category)
                    } else {
                        selectedCategories.insert(category)
                    }
                } label: {
                    HStack(spacing: 4) {
                        Image(systemName: category.iconName)
                        Text(category.rawValue)
                    }
                    .font(.caption)
                    .padding(.horizontal, 10)
                    .padding(.vertical, 6)
                    .background(
                        selectedCategories.contains(category)
                        ? category.color.opacity(0.2)
                        : Color.gray.opacity(0.1)
                    )
                    .foregroundStyle(
                        selectedCategories.contains(category)
                        ? category.color
                        : .gray
                    )
                    .clipShape(Capsule())
                }
            }
        }
        .padding(8)
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 16))
    }
}

// MARK: - ランドマーク詳細カード

struct LandmarkCard: View {
    let landmark: Landmark
    
    var body: some View {
        VStack(alignment: .leading, spacing: 6) {
            HStack {
                Image(systemName: landmark.category.iconName)
                    .foregroundStyle(landmark.category.color)
                Text(landmark.name)
                    .font(.headline)
                Spacer()
            }
            Text(landmark.description)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
        .padding()
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

このアプリは、東京の観光スポットを地図上に表示し、寺社・タワー・公園などのカテゴリごとに絞り込みながら確認できるアプリである。

マーカーを使って場所を分かりやすく表示し、観光地の説明も確認できる。

## コードの詳細解説

### データモデル（ランドマーク構造体）

```swift
struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let description: String
    let coordinate: CLLocationCoordinate2D
    let category: Category
    
    enum Category: String, CaseIterable {
        case temple = "寺社"
        case tower = "タワー"
        case park = "公園"
        
        var iconName: String {
            switch self {
            case .temple: return "building.columns"
            case .tower: return "antenna.radiowaves.left.and.right"
            case .park: return "leaf"
            }
        }
        
        var color: Color {
            switch self {
            case .temple: return .red
            case .tower: return .blue
            case .park: return .green
            }
        }
    }
}
```

**何をしているか：**

Landmark構造体は、地図上に表示する観光スポット1つ分のデータをまとめるためのものです。

名前、説明文、緯度・経度、カテゴリを持たせることで、地図上のマーカー表示や詳細情報の表示に使えるようにしています。

**なぜこう書くのか：**

Identifiableを使うことで、ForEachで複数の観光スポットを表示するときに、それぞれを区別できるようになります。

また、Categoryをenumで作ることで、「寺社」「タワー」「公園」のような種類を安全に管理でき、カテゴリごとにアイコンや色も分けやすくなります。

**もしこう書かなかったら：**

idやIdentifiableがないと、ForEachでランドマークを表示するときにエラーになることがあります。

また、カテゴリをただの文字列で管理すると、入力ミスが起きやすくなり、フィルターや色分けの処理が分かりにくくなります。

---

### 地図の表示とカメラ制御

```swift
@State private var cameraPosition: MapCameraPosition = .region(
    MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
        span: MKCoordinateSpan(latitudeDelta: 0.08, longitudeDelta: 0.08)
    )
)

Map(position: $cameraPosition) {
    ForEach(filteredLandmarks) { landmark in
        Marker(
            landmark.name,
            systemImage: landmark.category.iconName,
            coordinate: landmark.coordinate
        )
        .tint(landmark.category.color)
    }
}
.mapStyle(.standard(elevation: .realistic))
```

**何をしているか：**

この部分では、MapKitを使って地図を表示し、東京の観光スポットをマーカーとして表示しています。

また、cameraPositionを使うことで、地図を最初にどの位置・どの拡大率で表示するかを設定しています。

**なぜこう書くのか：**

@Stateを使ってcameraPositionを管理することで、地図の移動やズーム状態をSwiftUI側で保持できるようになります。

さらに、ForEachを使うことで、複数のランドマークをまとめて表示でき、データが増えても簡単に管理できます。

```swift
.mapStyle(.standard(elevation: .realistic))
```
を使うことで、立体感のある見やすい地図表示にしています。

**もしこう書かなかったら：**

cameraPositionを設定しない場合、地図が意図しない場所から始まる可能性があります。

また、ForEachを使わずに1つずつMarkerを書くと、観光スポットが増えた時にコードが長くなり、管理が大変になります。

.mapStyleを書かない場合は、デフォルトの地図表示になり、立体感のない見た目になります。

---

### マーカーの表示

```swift
ForEach(filteredLandmarks) { landmark in
    Marker(
        landmark.name,
        systemImage: landmark.category.iconName,
        coordinate: landmark.coordinate
    )
    .tint(landmark.category.color)
}
```

**何をしているか：**

この部分では、観光スポットのデータを1つずつ取り出して、地図上にマーカーを表示しています。

マーカーには場所の名前、カテゴリごとのアイコン、位置情報（緯度・経度）を設定し、カテゴリによって色も変更しています。

**なぜこう書くのか：**

ForEachを使うことで、配列に入っているランドマークを自動で繰り返し表示できます。

また、Markerを使うことで、MapKitの地図上に簡単にマーカーを配置できます。

カテゴリごとにiconNameやcolorを変えることで、ユーザーが見た時に「寺社」「タワー」「公園」の違いをすぐ分かるようにしています。

**もしこう書かなかったら：**

ForEachを使わない場合、観光スポットごとにMarkerを毎回手書きする必要があり、データが増えると管理が大変になります。

また、色やアイコンを分けなかった場合、すべて同じ見た目になり、どの種類の観光地なのか分かりにくくなります。

---

### フィルター機能

```swift
@State private var selectedCategories: Set<Landmark.Category> = Set(Landmark.Category.allCases)

var filteredLandmarks: [Landmark] {
    Landmark.sampleData.filter {
        selectedCategories.contains($0.category)
    }
}
```
```swift
ForEach(Landmark.Category.allCases, id: \.self) { category in
    Button {
        if selectedCategories.contains(category) {
            selectedCategories.remove(category)
        } else {
            selectedCategories.insert(category)
        }
    } label: {
        HStack(spacing: 4) {
            Image(systemName: category.iconName)
            Text(category.rawValue)
        }
    }
}
```

**何をしているか：**

この部分では、ユーザーがカテゴリごとに観光スポットを絞り込み表示できるフィルター機能を作っています。

「寺社」「タワー」「公園」のボタンを押すことで、選択されたカテゴリだけを地図上に表示するようにしています。

**なぜこう書くのか：**

Setを使うことで、選択中のカテゴリを重複なく管理できます。

また、filterを使うことで、選択されたカテゴリに一致するランドマークだけを簡単に取り出せます。

ForEachを使ってカテゴリボタンを自動生成しているため、新しいカテゴリが増えてもコードを大きく変更せずに対応できます。

**もしこう書かなかったら：**

フィルター機能がない場合、すべての観光スポットが常に表示されるため、地図が見づらくなる可能性があります。

また、カテゴリを1つずつ手書きで管理すると、カテゴリ追加時に修正箇所が増え、コード管理が大変になります。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`Map` | SwiftUIで地図を表示するビューコンポーネント | `Map(position: .constant(.region(region)))` |
| 例：`Marker` | 地図上に位置をマーキングするコンポーネント | `Marker("名前", coordinate: coordinate)` |
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
