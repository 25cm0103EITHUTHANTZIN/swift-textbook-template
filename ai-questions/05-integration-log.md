# AI質問ログ：第5章 機能統合の実践

## 使用した生成AIツール

ChatGPT 

## 質問と回答の記録

### Q1

**質問：**

① imageData: Data? = nil 　この書き方の意味は？

② 計算プロパティ　とはなんですか

**AIの回答の要点：**

① 写真データがなくてもよいようにする書き方です。

画像を選択しなかった場合は、自動で nil になります。

② 計算プロパティは値を保存するのではなく、呼ばれたときに中の処理を実行して、結果を返します。

```swift
var coordinate: CLLocationCoordinate2D {
CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
}
```
これは、緯度 latitude と経度 longitude を使って、地図で使える座標データを作っています。latitude と longitude は別々の数字です.

でも、SwiftUIの Map や Marker、Annotation では、別々の数字ではなく、

**CLLocationCoordinate2D** という形の座標データが必要です。

だからこのコードで、

```swift
CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
```

として、地図で使える形に変換しています。

```swift
var uiImage: UIImage? {
guard let data = imageData else {
return nil
}
return UIImage(data: data)
}
```
これは、保存している画像データ imageData を、画面に表示できる UIImage に変換しています。

SwiftDataに画像を保存するときは、UIImage のままではなく、Data として保存しています。(var imageData: Data?)

つまり保存するときは、　画像 → Data　という形です。

でも、画面に表示するときは、　Data → UIImage　に戻す必要があります。

その変換をしているのが uiImage です。

var uiImage: UIImage?　最後に ? が付いています。　これは、画像が作れない場合もあるという意味です。

つまり、UIImageがあるかもしれない　、nilになるかもしれない　ということです。

```swift
guard let data = imageData else {
return nil
}
```
imageData は Data? です。 つまり、画像データがあるかもしれないし、ないかもしれない状態です。

- imageData にデータがあるなら、data という名前で取り出す。
- imageData が nil なら、ここで nil を返して終わる。

```swift
return UIImage(data: data)
```
ここでは、取り出した Data を UIImage に変換しています。

UIImage(data: data) は、Data型の画像情報からUIImageを作る書き方です。

成功すれば UIImage が返ります。

失敗すれば nil が返ります。

だから uiImage の型は UIImage? になっています。


**自分の理解：**

UIImage(data: data) の左側の data: は引数ラベル、右側の data は実際の画像データを持つ変数である。

このコードによって、保存していた Data 型の画像を UIImage 型に変換し、画面に表示できるようにしていることが分かった。

### Q2

**質問：**

@Observable　をつけるとどうなる？役割はなんですか？なんで使うか？

**AIの回答の要点：**

**@Observable** の役割は　簡単に言うと、

「このクラスの値が変わったら、画面も自動で更新してください」

という意味です。

```swift
@Observable
class LocationManager {
}
```
を付けると SwiftUIが currentLocation が変わった！ と検知します。 すると画面を再描画します。

**自分の理解：**

@Observable は、クラスのプロパティが変更されたことを SwiftUI に通知するためのマクロである。

これを付けることで、currentLocation などの値が変わったときに画面が自動で再描画される。

位置情報のようにリアルタイムで変化するデータを画面へ反映したい場合に使用する。

### Q3

**質問：**

```swift
@Observable
class LocationManager: NSObject, CLLocationManagerDelegate
{
    private let manager = CLLocationManager()
    var currentLocation: CLLocationCoordinate2D?
    //最初currentLocation = nilです。
    //@ObservableがなければSwiftUIはcurrentLocationが変わったことを知りません。
    
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
```
このところを一つずつわかるように説明お願いします。

**AIの回答の要点：**

このコードでは、 currentLocation が変わったときに画面を更新したいので、@Observable を付けています。

❶ LocationManager -> というクラスを作っています。

役割は、iPhoneの現在地を取得することです。

つまり、このクラスは位置情報専用の管理クラスです。

❷  : NSObject  -> これは Apple の古い仕組みとつなげるために必要です。

CLLocationManagerDelegate を使うときは、NSObject を継承しておくことが多いです。

iPhoneの位置情報システムとやり取りするための土台です。

❸  CLLocationManagerDelegate  ->  これは、位置情報の結果を受け取るためのルールです。

CLLocationManager が現在地を取得したとき、「位置情報が更新されました」という通知を受け取る必要があります。

その通知を受け取るために、CLLocationManagerDelegate を付けています。

I.    private let manager = CLLocationManager()

これは、実際に位置情報を取得するためのオブジェクトを作っています。

LocationManager クラスは自分でGPSを直接読むわけではありません。

実際にiPhoneの位置情報を取得する仕事をするのは、CLLocationManager　です。

その CLLocationManager を manager という名前で持っています。

II.   var currentLocation: CLLocationCoordinate2D?

これは、現在地の座標を保存する変数です。型は CLLocationCoordinate2D? です。

これは、現在地があるかもしれない、まだ取得できていないので nil かもしれない　という意味です。

アプリを開いた直後は、まだGPSの結果が来ていないので nil です。

位置情報が取れると、currentLocation = 現在地の緯度・経度　になります。

III.   override init() 

これは、このクラス(Location Manager) が作られたときに最初に実行される処理です。

LocationManager が作られた瞬間に、

- 位置情報の準備
- 許可のリクエスト
- 位置情報の取得開始

を行っています。

IV.   super.init()

親クラスである NSObject の初期化処理を先に実行しています。

LocationManager は NSObject を継承しているので、自分の準備をする前に親クラスの準備も必要です。

V.    manager.delegate = self

これは、位置情報の結果は、この LocationManager クラス(manager)に知らせてください という設定です。

self は LocationManager 自分自身です。

つまり、

CLLocationManager が位置情報を取得する
→
delegate に結果を知らせる
→
LocationManager が受け取る

という流れになります。

これを書かないと、　didUpdateLocations　が呼ばれません。

VI.   manager.desiredAccuracy = kCLLocationAccuracyBest

これは、位置情報の精度を設定しています。

kCLLocationAccuracyBest は、 できるだけ正確な位置情報を取得する という意味です。

VII.   manager.requestWhenInUseAuthorization()

これは、ユーザーに位置情報の使用許可を求める処理です。

意味は、アプリを使っている間だけ位置情報を使ってもいいですか？ です。

これを書かないと、iPhoneは勝手に位置情報を使わせてくれません。

また、Info.plist に NSLocationWhenInUseUsageDescription が必要です。

VIII.    manager.startUpdatingLocation()

これは、現在地の取得を開始する処理です。

この処理を呼ぶと、iPhoneが位置情報を取得し始めます。

位置情報が取れると、次のメソッドが自動で呼ばれます。 locationManager(_:didUpdateLocations:)

IX.   func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation])

これは、位置情報が更新されたときに自動で呼ばれるメソッドです。

自分で直接呼ぶメソッドではありません。

CLLocationManager が現在地を取得したときに、delegate を通して呼ばれます。

locations: [CLLocation]

これは、取得された位置情報の配列です。

位置情報は1つだけではなく、複数入ってくることがあります。

その中で一番新しい位置情報を使いたいので、次のように書いています。 locations.last

X.    currentLocation = locations.last?.coordinate

これは、取得した位置情報の一番最後、つまり最新の座標を currentLocation に保存しています。

locations.last は、配列の最後の CLLocation を取り出します。

?.coordinate は、もし last が存在すれば coordinate を取り出すという意味です。

そして currentLocation に保存します。

**自分の理解：**

この LocationManager クラスは、iPhoneの現在地を取得するためのクラスである。CLLocationManager を使って位置情報の許可を求め、現在地の取得を開始している。位置情報が更新されると didUpdateLocations が呼ばれ、最新の座標を currentLocation に保存する。@Observable を付けているため、currentLocation が変わるとSwiftUIの画面も自動で更新される。

## 今日の質問を振り返って

（どんな質問が良い質問だったか。生成AIの回答で間違いや不正確な部分はあったか。次回はどんな質問をしてみたいか。）
