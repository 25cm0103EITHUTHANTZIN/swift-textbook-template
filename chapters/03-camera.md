# 第3章：カメラの利用

> 執筆者：イトゥタンジン
> 最終更新：２026-05-20

## この章で学ぶこと

この章では、SwiftUIで写真を選択・撮影して画面に表示する方法を学ぶ。

具体的には、PhotosPickerを使ったフォトライブラリ選択、UIImagePickerControllerを利用したカメラ撮影、そして非同期で画像データを読み込む方法を理解する。

さらに、UIViewControllerRepresentableやCoordinatorパターンを使い、UIKitのカメラ機能をSwiftUIと連携させる流れも学ぶ。

## 模範コードの全体像

```swift
// ============================================
// 第3章（基本）：写真を選択・撮影して表示するアプリ
// ============================================
// PhotosPickerを使ってフォトライブラリから写真を選択し、画面に表示します。
// 「カメラ」ボタンで撮影もできます。
//
// 【動作環境】
//   - フォトライブラリから選択：シミュレータでも動作します。
//   - カメラ撮影：実機（iPhone / iPad）専用。シミュレータでは
//     カメラボタンが自動的に無効化されます。
//
// 【注意】実機でカメラを使う場合は Info.plist に以下を追加してください：
//   - NSCameraUsageDescription
//     値: "撮影した写真を表示するためにカメラを使用します"
// ============================================

import SwiftUI
import PhotosUI

// MARK: - メインビュー

struct ContentView: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var selectedImage: Image?
    @State private var isShowingCamera = false
    @State private var capturedUIImage: UIImage?
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // 画像表示エリア
                imageDisplayArea
                
                // ボタンエリア
                HStack(spacing: 20) {
                    // フォトライブラリから選択
                    PhotosPicker(selection: $selectedItem, matching: .images) {
                        Label("ライブラリ", systemImage: "photo.on.rectangle")
                    }
                    .buttonStyle(.bordered)
                    
                    // カメラで撮影（シミュレータには未搭載のため自動的に無効化）
                    Button {
                        isShowingCamera = true
                    } label: {
                        Label("カメラ", systemImage: "camera")
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(!UIImagePickerController.isSourceTypeAvailable(.camera))
                }
                .padding()
            }
            .navigationTitle("写真アプリ")
            .onChange(of: selectedItem) { _, newItem in
                Task {
                    await loadImage(from: newItem)
                }
            }
            .fullScreenCover(isPresented: $isShowingCamera) {
                CameraView(capturedImage: $capturedUIImage)
            }
            .onChange(of: capturedUIImage) { _, newImage in
                if let uiImage = newImage {
                    selectedImage = Image(uiImage: uiImage)
                }
            }
        }
    }
    
    // MARK: - 画像表示エリア
    
    @ViewBuilder
    private var imageDisplayArea: some View {
        if let image = selectedImage {
            image
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(maxHeight: 400)
                .clipShape(RoundedRectangle(cornerRadius: 16))
                .shadow(radius: 4)
                .padding()
        } else {
            RoundedRectangle(cornerRadius: 16)
                .fill(.gray.opacity(0.1))
                .frame(height: 300)
                .overlay {
                    VStack(spacing: 8) {
                        Image(systemName: "photo")
                            .font(.system(size: 48))
                            .foregroundStyle(.gray)
                        Text("写真を選択または撮影してください")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                    }
                }
                .padding()
        }
    }
    
    // MARK: - 画像の読み込み
    
    func loadImage(from item: PhotosPickerItem?) async {
        guard let item = item else { return }
        
        do {
            if let data = try await item.loadTransferable(type: Data.self),
               let uiImage = UIImage(data: data) {
                selectedImage = Image(uiImage: uiImage)
            }
        } catch {
            print("画像の読み込みに失敗: \(error.localizedDescription)")
        }
    }
}

// MARK: - カメラビュー（UIKit連携）

struct CameraView: UIViewControllerRepresentable {
    @Binding var capturedImage: UIImage?
    @Environment(\.dismiss) private var dismiss
    
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.sourceType = .camera
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: CameraView
        
        init(_ parent: CameraView) {
            self.parent = parent
        }
        
        func imagePickerController(
            _ picker: UIImagePickerController,
            didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
        ) {
            if let image = info[.originalImage] as? UIImage {
                parent.capturedImage = image
            }
            parent.dismiss()
        }
        
        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.dismiss()
        }
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

このアプリは、フォトライブラリから写真を選択したり、カメラで撮影した写真を画面に表示したりできる写真表示アプリである。

ユーザーは「ライブラリ」ボタンから端末内の画像を選択でき、「カメラ」ボタンから撮影した写真もそのまま表示できる。

また、SwiftUIとUIKitを連携させることで、iPhoneのカメラ機能を利用している。

## コードの詳細解説

### PhotosPickerによる写真選択

```swift
@State private var selectedItem: PhotosPickerItem?

PhotosPicker(selection: $selectedItem, matching: .images) {
    Label("ライブラリ", systemImage: "photo.on.rectangle")
}
.buttonStyle(.bordered)

.onChange(of: selectedItem) { _, newItem in
    Task {
        await loadImage(from: newItem)
    }
}
```

**何をしているか：**

このコードは、iPhoneのフォトライブラリから写真を選択する機能を作っている。

- @State private var selectedItem
  → ユーザーが選択した写真情報を保存するための変数。
- PhotosPicker
  → 写真選択画面を表示するSwiftUIの部品。
- selection: $selectedItem
  → 選択された画像を selectedItem に自動保存する。
- matching: .images
  → 写真だけを選択できるように制限している。
- .onChange(of: selectedItem)
  → 写真が選ばれた瞬間を検知する。
- Task { await loadImage(...) }
  → 非同期処理で画像データを読み込み、画面へ表示する。

つまり、

1. ボタンを押す
2. フォトライブラリが開く
3. 写真を選ぶ
4. selectedItem に保存される
5. loadImage() が呼ばれる
6. 画像が画面に表示される

という流れになっている。



**なぜこう書くのか：**

PhotosPicker は、Appleが用意している新しい写真選択APIであり、SwiftUIと相性が良いからである。

特に重要なのは、

```swift
.onChange(of: selectedItem)
```

PhotosPicker は「選択された瞬間」に自動で処理を実行してくれないため、自分で変化を監視する必要がある。

```swift
Task {
    await loadImage(from: newItem)
}
```
としている理由は、画像読み込みが非同期処理だからである。

```swift
item.loadTransferable(type: Data.self)
```
は時間がかかる可能性があるため、await を使って安全に読み込む必要がある。

もし同期処理で無理やり読み込むと、画面が一時停止したり、アプリ操作が重くなる可能性がある。

**もしこう書かなかったら：**

① .onChange(of: selectedItem) を書かなかった場合

写真を選択しても、loadImage(from:) が呼ばれないため、画像は表示されない。

実際には、

- フォトライブラリは開く
- 写真選択もできる
- しかし画面が変化しない

という状態になる。

② matching: .images を書かなかった場合

画像以外のデータも選択対象になる可能性がある。

例えば動画などを選べるようになり、UIImage(data:) で変換できずエラーになる場合がある。

③ Task や await を使わなかった場合

非同期処理が正しく動かず、

- コンパイルエラー
- UIフリーズ
- 画像読み込み失敗

などが発生する可能性がある。

---

### 画像の非同期読み込み

```swift
// 該当部分のコードを抜粋して貼る
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### UIViewControllerRepresentableによるカメラ連携

```swift
// 該当部分のコードを抜粋して貼る
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### Coordinatorパターン

```swift
// 該当部分のコードを抜粋して貼る
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| 例：`PhotosPicker` | フォトライブラリから画像を選択するコンポーネント | `PhotosPicker(selection: $selectedItem, matching: .images)` |
| 例：`UIImagePickerController` | カメラまたはフォトライブラリにアクセスするUIKitコンポーネント | `picker.sourceType = .camera` |
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
