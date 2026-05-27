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
```

**何をしているか：**

このコードは、PhotosPickerで選択された画像データを非同期で読み込み、SwiftUIの Image として画面に表示する処理を行っている。

流れとしては、

- PhotosPickerItem を受け取る
- loadTransferable() で画像データを取得する
- UIImage に変換する
- selectedImage に保存する
- 画面が自動更新される

という動きになっている。

**なぜこう書くのか：**

画像データの取得は時間がかかる可能性があるため、非同期処理 (async / await) を使う必要がある。

フォトライブラリ内の画像は、

- サイズが大きい
- iCloudからダウンロードされる場合がある
- 読み込みに時間がかかる

などの理由がある。

そのため、async await を使い、UIを止めずに安全に画像を読み込んでいる。

また、
```swift
guard let item = item else { return }
```
としている理由は、画像未選択状態で処理が実行されるのを防ぐためである。

もし nil のまま読み込みをすると、クラッシュやエラーの原因になる。

```swift
do {
    ...
} catch {
    ...
}
```
を使うことで、読み込み失敗時にもアプリが落ちず、エラー内容だけを表示できるようにしている。

**もしこう書かなかったら：**

① await を書かなかった場合

item.loadTransferable(type: Data.self) は非同期関数なので、await を付けないとコンパイルエラーになる。

Swiftは、「時間がかかる処理なので待機が必要です」と判断するためである。

② guard let item = item を書かなかった場合

item が nil の可能性を考慮できず、

- クラッシュ
- Optional関連エラー

などが発生する可能性がある。

③ UIImage(data:) を使わなかった場合

Data型 のままでは画像表示できない。

つまり、

```swift
selectedImage = Image(...)
```
へ直接渡せないため、画面に画像を表示できなくなる。

---

### UIViewControllerRepresentableによるカメラ連携

```swift
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
}
```

**何をしているか：**

このコードは、SwiftUIの画面からUIKitの UIImagePickerController を使って、カメラ撮影画面を表示するための処理である。

UIImagePickerController はSwiftUI専用の部品ではなく、UIKitの部品である。そのため、SwiftUIで直接使うことはできない。そこで UIViewControllerRepresentable を使い、UIKitの画面をSwiftUIの中で使える形に変換している。

makeUIViewController の中では、カメラ用の UIImagePickerController を作成し、sourceType = .camera にすることでカメラ撮影モードにしている。

**なぜこう書くのか：**

SwiftUIには標準のカメラ撮影用ビューがないため、UIKitの UIImagePickerController を利用する必要がある。

そのUIKitの画面をSwiftUIで表示するために、

```swift
UIViewControllerRepresentable
```
 を使っている。

また、

```swift
@Binding var capturedImage: UIImage?
```
を使うことで、カメラで撮影した画像を ContentView 側へ渡せるようにしている。

**もしこう書かなかったら：**

UIViewControllerRepresentable を使わないと、SwiftUIの画面内で UIImagePickerController を表示できない。

そのため、

- カメラ画面を開けない
- 撮影した画像をSwiftUIへ渡せない
- fullScreenCover でカメラ画面を表示できない

という問題が起きる。

---

### Coordinatorパターン

```swift
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
```

**何をしているか：**

Coordinatorは、UIKit側で起きた操作結果をSwiftUI側へ伝える役割をしている。

このアプリでは、カメラで写真を撮影したあと、

```swift
didFinishPickingMediaWithInfo
```

が呼ばれる。

その中で、撮影された画像を取り出し、

```swift
parent.capturedImage = image
```

によって ContentView 側の capturedUIImage に渡している。

また、撮影が終わったときやキャンセルしたときに、

```swift
parent.dismiss()
```

を呼び、カメラ画面を閉じている。

**なぜこう書くのか：**

UIKitの UIImagePickerController は、撮影完了やキャンセルなどの結果を delegate で受け取る仕組みになっている。

SwiftUIにはdelegateの仕組みがそのまま存在しないため、SwiftUIとUIKitの間をつなぐ役として Coordinator を作る必要がある。

つまり、Coordinatorは、

UIKitのカメラ操作結果 → SwiftUIの状態変数

へ変換する橋渡し役である。

**もしこう書かなかったら：**

Coordinatorを書かなかった場合、撮影はできても、撮影後の画像をSwiftUI側で受け取れない。

また、

- 撮影完了後に画面が閉じない
- キャンセルしても戻れない
- capturedImage に画像が入らない
- selectedImage が更新されない

という問題が起きる。

特にこの部分がないと、

```swift
parent.capturedImage = image
```

が実行されないため、撮影した写真を画面に表示できない。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| `PhotosPicker` | フォトライブラリから画像を選択するコンポーネント | `PhotosPicker(selection: $selectedItem, matching: .images)` |
| `UIImagePickerController` | カメラまたはフォトライブラリにアクセスするUIKitコンポーネント | `picker.sourceType = .camera` |
| `@State`| View内で状態を保持し、値が変わると画面を再描画する| `@State private var selectedImage: Image?`|
| `@Binding`|親Viewの状態を子Viewから共有・変更するための仕組み |`@Binding var capturedImage: UIImage?` |
| `@Environment`| SwiftUIの環境情報を取得する仕組み| `@Environment(\.dismiss)`|
| `@ViewBuilder`|条件によって複数のViewを切り替えて返せるようにする | `@ViewBuilder private var imageDisplayArea`|
| `NavigationStack`| 画面遷移やナビゲーションタイトルを管理するSwiftUIコンテナ| `NavigationStack { ... }`|
| `.onChange`| 値が変化したタイミングで処理を実行する| .onChange(of: selectedItem)|
| `Task`| 非同期処理を開始するための仕組み | `Task { await loadImage(...) }`|
| `async / await`| 非同期処理を安全に待機しながら実行する文法| `func loadImage(...) async`|
|`loadTransferable()` |PhotosPickerから画像データを取得するAPI |`item.loadTransferable(type: Data.self)` |
| `guard let`| nilチェックを行い、安全に値を取り出す| `guard let item = item else { return }`|
| `if let`| Optional型に値がある場合だけ安全に使う|`if let uiImage = newImage` |


## 自分の実験メモ

```swift
import SwiftUI
import PhotosUI

struct ContentView: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var selectedImage: Image?
    @State private var isShowingCamera = false
    @State private var capturedUIImage: UIImage?
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                imageDisplayArea
                
                HStack(spacing: 20) {
                    
                    // 実験1：
                    // matching: .images を削除
                    // 元コード：
                    // PhotosPicker(selection: $selectedItem, matching: .images)
                    PhotosPicker(selection: $selectedItem) {
                        Label("ライブラリ", systemImage: "photo.on.rectangle")
                    }
                    .buttonStyle(.bordered)
                    
                    Button {
                        isShowingCamera = true
                    } label: {
                        Label("カメラ", systemImage: "camera")
                    }
                    .buttonStyle(.borderedProminent)
                    
                    // 実験6：
                    // .disabled(...) を削除
                    // 元コード：
                    // .disabled(!UIImagePickerController.isSourceTypeAvailable(.camera))
                }
                .padding()
            }
            .navigationTitle("写真アプリ")
            
            // 実験2：
            // onChange をコメントアウト
            // 元コード：
            /*
            .onChange(of: selectedItem) { _, newItem in
                Task {
                    await loadImage(from: newItem)
                }
            }
            */
            
            .fullScreenCover(isPresented: $isShowingCamera) {
                CameraView(capturedImage: $capturedUIImage)
            }
            
            .onChange(of: capturedUIImage) { _, newImage in
                
                // 実験5：
                // if let を削除するとエラーになるため、
                // エラー確認用コードはコメントとして残す
                if let uiImage = newImage {
                    selectedImage = Image(uiImage: uiImage)
                }
                
                // エラーになる例：
                // selectedImage = Image(uiImage: newImage)
            }
        }
    }
    
    @ViewBuilder
    private var imageDisplayArea: some View {
        if let image = selectedImage {
            image
                .resizable()
            
                // 実験7：
                // aspectRatio を削除
                // 元コード：
                // .aspectRatio(contentMode: .fit)
            
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

struct CameraView: UIViewControllerRepresentable {
    
    // 実験4：
    // @Binding を普通の変数にすると親Viewへ画像が渡らない
    @Binding var capturedImage: UIImage?
    
    // エラー確認・実験用：
    // var capturedImage: UIImage?
    
    @Environment(\.dismiss) private var dismiss
    
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        
        // 実験3：
        // .camera を .photoLibrary に変更
        // 元コード：
        // picker.sourceType = .camera
        picker.sourceType = .photoLibrary
        
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(
        _ uiViewController: UIImagePickerController,
        context: Context
    ) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject,
                       UIImagePickerControllerDelegate,
                       UINavigationControllerDelegate {
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

**実験1：**
- やったこと：　matching: .images を削除して、PhotosPickerで画像以外も選択できる状態にした。
- 結果：　フォトライブラリは開いたが、画像以外のデータも選択対象になった。
- わかったこと：　matching: .images は「画像だけを選択可能にする」役割があると分かった。これがあることで、UIImageへ安全に変換できている。

**実験2：**
- やったこと：　.onChange(of: selectedItem) の部分をコメントアウトした。
- 結果：　写真は選択できたが、画面に画像が表示されなかった。
- わかったこと：　.onChange が、写真選択後に loadImage() を実行する役割をしていると分かった。この処理がないと、画像読み込みが行われず、画面表示も更新されない。

**実験3：**
- やったこと： picker.sourceType = .camera を .photoLibrary に変更した。
- 結果： カメラ画面ではなく、フォトライブラリ画面が表示された。
- わかったこと： sourceType によって、UIImagePickerControllerが「カメラを開くか」「フォトライブラリを開くか」を切り替えていることが分かった。

**実験4：**
- やったこと： @Binding var capturedImage を普通の変数に変更した。
- 結果： 撮影自体はできたが、撮影後に画像が画面へ表示されなかった。
- わかったこと： @Binding は親Viewと子Viewで同じ状態を共有するために必要だと分かった。普通の変数にすると、CameraView内だけ値が変わり、ContentView側は更新されなかった。

**実験5：**
- やったこと： if let uiImage = newImage を削除した。
- 結果： コンパイルエラーになった。
- わかったこと： newImage は Optional型 (UIImage?) のため、nilの可能性を考慮して安全に取り出す必要があると分かった。

**実験6：**
- やったこと： .disabled(!UIImagePickerController.isSourceTypeAvailable(.camera)) を削除した。
- 結果： シミュレータでもカメラボタンが押せるようになったが、実際にはカメラ機能が存在しないため正常に動作しなかった。
- わかったこと： .disabled() は、使えない機能を事前に無効化し、ユーザーの誤操作を防ぐ役割があると分かった。

**実験7：**
- やったこと： .aspectRatio(contentMode: .fit) を削除した。
- 結果：画像サイズが崩れたり、一部が伸びて表示された。
- わかったこと： .aspectRatio() は画像の縦横比を維持するために重要だと分かった。これがないと、画像が不自然に変形する場合がある。

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**　Struct と Class の違いは何か？
   
   **得られた理解：**　Structは値型で、代入するとデータがコピーされる。一方、Classは参照型で、同じデータを共有することが分かった。SwiftUIでは安全に画面更新しやすいためStructが多く使われ、delegate処理や共有管理ではClassが使われることを理解できた。

3. **質問：**　@Binding とは何か？
   
   **得られた理解：**　@Binding は、親Viewの状態を子Viewから直接変更するための仕組みだと理解した。今回のアプリでは、CameraViewで撮影した画像をContentViewへ渡すために使われており、親子で同じ状態を共有していることが分かった。

5. **質問：**　.onChange(of:) { _, newValue in } の文法は何か？
   
   **得られた理解：**　.onChange は値の変化を監視する機能で、変更前と変更後の値を受け取れることが分かった。_ は「使わない値」を意味し、今回のコードでは変更後の値だけを利用して画像読み込みを行っていることを理解できた。

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
