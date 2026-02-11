# ネコトコ (Nekotoko) アプリ仕様・引き継ぎ書

## 1. アプリ概要

| 項目 | 内容 |
|------|------|
| アプリ名 | ネコトコ |
| サブタイトル | ゆるく繋がる猫の気配タイマー |
| パッケージ名 (Android) | `com.silentfocus.silent_focus` |
| バンドルID (iOS) | `com.silentfocus.silentFocus` |
| フレームワーク | Flutter (Dart) |
| 対象プラットフォーム | iOS / Android |
| リリース対象地域 | 日本のみ |
| バージョン | 1.0.0+1 |
| 対象ユーザー | 集中作業をしたい人、猫好きな人 |

### コンセプト
猫と一緒にゆるく集中する作業タイマーアプリ。猫のアバターが画面に常駐し、タイマー中は他のユーザーの猫の「気配」がリアルタイムで表示される。一人で作業していても「誰かと一緒に頑張っている」感覚を得られる。

---

## 2. 主要機能

### 2.1 タイマー機能
- **ポモドーロモード**: 集中 → 休憩 → 集中 のサイクル（集中: 5〜120分、休憩: 1〜30分、カスタマイズ可能）
- **フリーモード**: 制限なしのストップウォッチ式タイマー（1分以上で記録保存）
- **カウントダウン演出**: 開始前に3秒カウントダウン + 「深呼吸して整える」メッセージ
- **完了演出**: AfterGlow（温かみのあるオーバーレイ）+ 猫の褒めメッセージ
- **バックグラウンド対応**: アプリが背景に行ってもタイマードリフト補正で正確な時間を維持

### 2.2 猫アバター
- デフォルトはイラスト猫アイコン
- ユーザーが自分の猫の写真を設定可能（カメラ撮影 or フォトライブラリ選択）
- ML Kit による自動正方形クロップ
- 猫の名前をカスタマイズ可能（デフォルト: 「ねこ」）
- 呼吸アニメーションで常に微動

### 2.3 猫の気配（プレゼンス機能）
- Firebase Firestore を使用したリアルタイム同期
- Firebase Anonymous Auth で匿名ログイン
- タイマー実行中、他ユーザーの猫の名前・作業内容が画面下部に「気配」として表示（最大3匹）
- 30秒ごとにハートビートを送信、90秒以上応答がないユーザーは非アクティブ扱い
- アクティブユーザー数を表示

### 2.4 ASMR サウンド
- 猫のゴロゴロ音を5種類収録（全て CC0 / Pixabay Content License）
  - 静かなゴロゴロ、まどろみループ、低めの喉鳴り、近距離ゴロゴロ、穏やかな呼吸
- タイマー中にバックグラウンド再生可能

### 2.5 背景テーマ
- 7種類: 自動（時間帯連動）、朝、カフェ、夕暮れ、夜、雨、季節
- 自動モードは端末の現在時刻に応じて背景色が変化

### 2.6 集中ログ・統計
- 集中完了時に日付・分数・タスク名・猫名・モード・時間帯を記録
- 今日の集中時間（分）を表示
- 連続日数（ストリーク）を計算・表示
- データは SharedPreferences にローカル保存（JSON形式）

### 2.7 アプリ内課金 (IAP)
- **買い切り** (`nekotoko_lifetime`): 全機能永久解放（1回限りの購入）
- 無料版の制限: 猫写真設定不可、一部背景テーマがロック

### 2.8 オンボーディング
- 初回起動時にウェルカム画面を表示
- 猫の名前入力 + 猫写真設定（スキップ可能）

---

## 3. 使用している外部サービス・SDK

### 3.1 Firebase（Google）

| サービス | 用途 | 収集データ |
|----------|------|-----------|
| **Firebase Anonymous Auth** | ユーザー識別（プレゼンス機能用） | 匿名UID（自動生成、個人特定不可） |
| **Cloud Firestore** | プレゼンス（猫の気配）のリアルタイム同期 | 匿名UID, 猫の名前, 作業タスク名, モード, フェーズ, アクティブ状態, タイムスタンプ |
| **Firebase Crashlytics** | クラッシュレポート収集 | デバイス情報, OS バージョン, クラッシュスタックトレース |
| **Firebase Analytics** | 利用状況分析 | 後述のイベント一覧参照 |

### 3.2 Analytics イベント一覧

| イベント名 | パラメータ | 説明 |
|-----------|-----------|------|
| `timer_start` | `mode` (pomodoro/free), `minutes` | タイマー開始時 |
| `timer_complete` | `mode`, `minutes` | タイマー正常完了時 |
| `mode_change` | `mode` | タイマーモード切替時 |
| `settings_open` | なし | 設定画面を開いた時 |
| `purchase_open` | `product_type` | 課金画面を開いた時 |

### 3.3 その他のサービス・SDK

| SDK | 用途 |
|-----|------|
| Google Play Billing / StoreKit (in_app_purchase) | アプリ内課金 |
| flutter_local_notifications | タイマー完了通知 |
| image_picker | カメラ/フォトライブラリからの画像選択 |
| image_cropper (UCrop) | 猫写真の正方形クロップ |
| audioplayers | ASMR サウンド再生 |
| shared_preferences | ローカル設定・履歴保存 |
| path_provider | 画像ファイルの永続保存先取得 |

---

## 4. データの取り扱い

### 4.1 ローカル保存データ（SharedPreferences）

| キー | 型 | 内容 |
|------|-----|------|
| `hasLaunched` | bool | 初回起動済みフラグ |
| `catName` | String | 猫の名前 |
| `catImagePath` | String | 猫写真のローカルファイルパス |
| `soundEnabled` | bool | サウンド ON/OFF |
| `timerMode` | String | 選択中のタイマーモード |
| `timerMinutes` | int | 集中時間（分） |
| `breakMinutes` | int | 休憩時間（分） |
| `backgroundTheme` | String | 背景テーマ |
| `entitlement_lifetime` | bool | 買い切り購入状態 |
| `entitlement_subscription` | bool | サブスクリプション状態 |
| `focusHistory` | String (JSON) | 集中ログ履歴 |

### 4.2 サーバー保存データ（Firestore `presence` コレクション）

| フィールド | 型 | 内容 |
|-----------|-----|------|
| `uid` | String | Firebase 匿名UID |
| `catName` | String | 猫の名前 |
| `task` | String | 現在の作業名 |
| `mode` | String | タイマーモード |
| `phase` | String | focus / rest |
| `isActive` | bool | アクティブ状態 |
| `lastBeat` | Timestamp | 最終ハートビート |
| `startedAt` | Timestamp | セッション開始時刻 |

**注意**: サーバーに保存されるのはプレゼンス情報のみ。集中ログ・設定・購入状態は端末内のみに保存。

### 4.3 収集しない情報
- 氏名、メールアドレス、電話番号などの個人情報
- 位置情報
- 連絡先
- ブラウザ履歴
- 端末固有ID（IDFA/AAID）の明示的な収集

---

## 5. 権限（パーミッション）

### iOS (Info.plist)

| 権限 | 説明文 |
|------|--------|
| `NSCameraUsageDescription` | 猫の写真を撮影するためにカメラを使用します |
| `NSPhotoLibraryUsageDescription` | 猫の写真を選択するためにフォトライブラリを使用します |

### Android (AndroidManifest.xml)

| 権限 | 用途 |
|------|------|
| `INTERNET` | Firebase 通信、課金処理 |
| `POST_NOTIFICATIONS` | タイマー完了通知 |

---

## 6. 技術構成

### ディレクトリ構造
```
lib/
  main.dart              # エントリポイント（Crashlytics, 通知初期化）
  app.dart               # MaterialApp 定義
  screens/
    splash_screen.dart   # スプラッシュ画面
    focus_screen.dart    # メイン画面（タイマー, 設定, 課金, プレゼンス）
  models/
    app_colors.dart      # カラーパレット
    enums.dart           # TimerMode, TimerPhase, BackgroundTheme
    background_option.dart
    cat_session.dart     # プレゼンス用データクラス
    focus_log.dart       # 集中ログデータクラス
    asmr_track.dart
  constants/
    messages.dart        # 完了メッセージ, 休憩ヒント, アイドル行動
    products.dart        # IAP 商品ID
    asmr_tracks.dart     # ASMR トラック定義
  helpers/
    analytics.dart       # Firebase Analytics ラッパー
    timer_formatter.dart # 時間フォーマット
    streak_calculator.dart # 連続日数計算
    focus_stats.dart     # 今日の集中時間, メッセージ選択
  painters/
    cat_painters.dart    # DefaultCatIcon, CatPainter, HappyCatPainter, SleepingCatPainter
```

### テスト
- 30件のテスト（全通過）
- ユニットテスト: helpers (timer_formatter, streak_calculator, focus_stats), models (focus_log)
- ウィジェットテスト: FocusScreen (7件), SplashScreen (3件), App (1件)

---

## 7. 音源ライセンス

| トラック | ソース | ライセンス |
|---------|--------|-----------|
| 静かなゴロゴロ | Pixabay | Pixabay Content License |
| まどろみループ | Freesound | CC0 (パブリックドメイン) |
| 低めの喉鳴り | Freesound | CC0 |
| 近距離ゴロゴロ | Freesound | CC0 |
| 穏やかな呼吸 | Freesound | CC0 |

---

## 8. プライバシーポリシーに含めるべき事項

1. **収集する情報**: Firebase 匿名UID, 猫の名前, 作業タスク名（プレゼンス用）、クラッシュレポート、Analytics イベント
2. **収集しない情報**: 個人情報（氏名, メール, 電話番号, 位置情報など）
3. **データの保存先**: 設定・履歴は端末内のみ。プレゼンスデータは Google Cloud (Firebase) に一時保存
4. **データの共有**: 他ユーザーに表示されるのは「猫の名前」「作業タスク名」のみ（匿名）
5. **データの削除**: アプリをアンインストールするとローカルデータは削除。Firestore のプレゼンスデータは 90秒後に自動的に非アクティブになり、表示されなくなる
6. **第三者サービス**: Firebase (Google) のプライバシーポリシーへのリンク
7. **子供のプライバシー**: 13歳未満を対象としていない旨
8. **お問い合わせ先**: 開発者の連絡先
