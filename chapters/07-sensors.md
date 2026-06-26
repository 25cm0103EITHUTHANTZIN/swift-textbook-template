# 第7章：センサーの活用

> 執筆者：　イトゥタンジン
> 最終更新：2026-06-26

## この章で学ぶこと

この章では、iPhoneの傾きを感知するセンサーを使って、水平器アプリを作る方法を学びます。CoreMotionを利用して、端末が前後や左右にどれくらい傾いているかをリアルタイムで取得し、その値に合わせて画面上のバブルが動く仕組みを実装します。また、SwiftUIで円や線を描いたり、アニメーションを付けて滑らかに動かしたりする方法も学びます。さらに、センサーが使えない場合の画面表示や、メモリを効率よく管理する書き方についても理解を深めます。この章を通して、iPhoneのセンサーとSwiftUIを組み合わせて、実際に動くアプリを作る方法を身に付けることが目標です。


## 模範コードの全体像

```swift
// ============================================
// 第7章（基本）：加速度センサーで動く水平器アプリ
// ============================================
// CoreMotionを使って端末の傾きをリアルタイムで取得し、
// 水平器（水準器）として表示するアプリです。
//
// 【注意】シミュレータではセンサーが使えません。
//         実機（iPhone / iPad）でテストしてください。
// ============================================

import SwiftUI
import CoreMotion

// MARK: - モーションマネージャー

@Observable
class MotionManager {
    private let motionManager = CMMotionManager()
    
    var pitch: Double = 0    // 前後の傾き
    var roll: Double = 0     // 左右の傾き
    var yaw: Double = 0      // 水平方向の回転
    var isAvailable: Bool
    
    init() {
        // 初回 body 評価時点で正しい値を返すよう、init で同期的にセット
        isAvailable = motionManager.isDeviceMotionAvailable
    }
    
    func startUpdates() {
        guard isAvailable else { return }
        
        motionManager.deviceMotionUpdateInterval = 1.0 / 60.0
        
        motionManager.startDeviceMotionUpdates(to: .main) { [weak self] motion, error in
            guard let self = self, let motion = motion else { return }
            
            self.pitch = motion.attitude.pitch
            self.roll = motion.attitude.roll
            self.yaw = motion.attitude.yaw
        }
    }
    
    func stopUpdates() {
        motionManager.stopDeviceMotionUpdates()
    }
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var motionManager = MotionManager()
    
    var body: some View {
        NavigationStack {
            if motionManager.isAvailable {
                VStack(spacing: 30) {
                    // 水平器の円
                    LevelIndicator(
                        pitch: motionManager.pitch,
                        roll: motionManager.roll
                    )
                    
                    // 数値表示
                    DataDisplay(
                        pitch: motionManager.pitch,
                        roll: motionManager.roll,
                        yaw: motionManager.yaw
                    )
                }
                .padding()
                .navigationTitle("水平器")
            } else {
                ContentUnavailableView(
                    "センサーが利用できません",
                    systemImage: "iphone.slash",
                    description: Text("このアプリは実機（iPhone）で動作します。\nシミュレータではセンサーが使えません。")
                )
            }
        }
        .onAppear {
            motionManager.startUpdates()
        }
        .onDisappear {
            motionManager.stopUpdates()
        }
    }
}

// MARK: - 水平器インジケーター

struct LevelIndicator: View {
    let pitch: Double
    let roll: Double
    
    private let maxOffset: CGFloat = 100
    
    private var xOffset: CGFloat {
        CGFloat(roll) * maxOffset
    }
    
    private var yOffset: CGFloat {
        CGFloat(pitch) * maxOffset
    }
    
    private var isLevel: Bool {
        abs(pitch) < 0.03 && abs(roll) < 0.03
    }
    
    var body: some View {
        ZStack {
            // 外側の円
            Circle()
                .stroke(.gray.opacity(0.3), lineWidth: 2)
                .frame(width: 250, height: 250)
            
            // 中心の十字線
            Path { path in
                path.move(to: CGPoint(x: 125, y: 0))
                path.addLine(to: CGPoint(x: 125, y: 250))
                path.move(to: CGPoint(x: 0, y: 125))
                path.addLine(to: CGPoint(x: 250, y: 125))
            }
            .stroke(.gray.opacity(0.2), lineWidth: 1)
            .frame(width: 250, height: 250)
            
            // 中間の円
            Circle()
                .stroke(.gray.opacity(0.2), lineWidth: 1)
                .frame(width: 125, height: 125)
            
            // バブル（傾きに応じて移動）
            Circle()
                .fill(isLevel ? .green : .red)
                .frame(width: 40, height: 40)
                .opacity(0.8)
                .shadow(color: isLevel ? .green : .red, radius: 8)
                .offset(
                    x: max(-maxOffset, min(maxOffset, xOffset)),
                    y: max(-maxOffset, min(maxOffset, yOffset))
                )
                .animation(.spring(duration: 0.1), value: xOffset)
                .animation(.spring(duration: 0.1), value: yOffset)
            
            // 水平時の表示
            if isLevel {
                Text("水平!")
                    .font(.headline)
                    .foregroundStyle(.green)
                    .offset(y: 140)
            }
        }
    }
}

// MARK: - 数値データ表示

struct DataDisplay: View {
    let pitch: Double
    let roll: Double
    let yaw: Double
    
    var body: some View {
        VStack(spacing: 12) {
            DataRow(
                label: "前後の傾き（Pitch）",
                value: pitch,
                icon: "arrow.up.and.down"
            )
            DataRow(
                label: "左右の傾き（Roll）",
                value: roll,
                icon: "arrow.left.and.right"
            )
            DataRow(
                label: "水平回転（Yaw）",
                value: yaw,
                icon: "arrow.triangle.2.circlepath"
            )
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 12)
                .fill(.gray.opacity(0.05))
        )
    }
}

struct DataRow: View {
    let label: String
    let value: Double
    let icon: String
    
    var body: some View {
        HStack {
            Image(systemName: icon)
                .frame(width: 30)
                .foregroundStyle(.blue)
            
            Text(label)
                .font(.caption)
            
            Spacer()
            
            Text(String(format: "%.3f rad", value))
                .font(.system(.caption, design: .monospaced))
                .foregroundStyle(.secondary)
            
            Text(String(format: "(%.1f°)", value * 180 / .pi))
                .font(.system(.caption, design: .monospaced))
                .foregroundStyle(.secondary)
                .frame(width: 60, alignment: .trailing)
        }
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

このアプリは、iPhoneの傾きをリアルタイムで測定し、水平かどうかを確認できる水平器アプリです。端末を前後や左右に傾けると、画面上の丸いバブルがその傾きに合わせて動きます。端末が水平になるとバブルが中央に移動し、「水平！」と表示されて色も緑色に変わります。また、画面には前後・左右の傾きや回転角度も数値で表示されるため、端末がどのくらい傾いているのかを確認することができます。

## コードの詳細解説

### CoreMotionの基本（CMMotionManager）

```swift
// 該当部分のコードを抜粋して貼る
```

**何をしているか：**
（この部分が果たしている役割を説明する）

**なぜこう書くのか：**
（別の書き方ではなく、この書き方が選ばれている理由を説明する）

**もしこう書かなかったら：**
（この部分を省略したり変えたりすると何が起きるか。実際に試した結果があればここに書く）

---

### デバイスの姿勢データ（pitch/roll/yaw）

```swift
@Observable
class MotionManager {
    private let motionManager = CMMotionManager()

    var pitch: Double = 0
    var roll: Double = 0
    var yaw: Double = 0
    var isAvailable: Bool

    init() {
        isAvailable = motionManager.isDeviceMotionAvailable
    }
}
```

**何をしているか：**

この部分では、iPhoneのモーションセンサーを管理するための CMMotionManager を作成しています。また、取得したセンサーの値を保存するために、前後の傾きを表す pitch、左右の傾きを表す roll、端末の向きを表す yaw を用意しています。

そして、isDeviceMotionAvailable を使って、この端末でモーションセンサーが利用できるかどうかを確認しています。これにより、センサーが使えない場合は、アプリが無理に動作しないようになっています。

**なぜこう書くのか：**

CMMotionManager は、CoreMotionでセンサーの情報を取得するための中心となるクラスです。そのため、最初にインスタンスを作成しておく必要があります。

また、isAvailable を init() の中で設定していることで、画面が最初に表示された時点でセンサーが利用できるかどうかをすぐ判断できます。これにより、利用できる端末では水平器を表示し、利用できない場合はエラーメッセージを表示するという処理を簡単に実現できます。

**もしこう書かなかったら：**

もし CMMotionManager を作成しなければ、iPhoneのセンサー情報を取得することができず、水平器はまったく動きません。

また、isDeviceMotionAvailable を確認しない場合、センサーが利用できない環境（シミュレータなど）でもセンサーを使おうとしてしまいます。その結果、画面には傾きが表示されず、正常に動作しないアプリになってしまいます。

---

### 歩数計（CMPedometer）

```swift
// 該当部分のコードを抜粋して貼る
```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### CoreLocationとの連携

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
| 例：`CMMotionManager` | 加速度・ジャイロ・気圧などのセンサーデータを取得 | `motionManager.startDeviceMotionUpdates(to: .main) { ... }` |
| 例：`CMPedometer` | 歩数や歩行距離をカウント | `pedometer.queryPedometerData(from: startDate, to: Date())` |
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
