---
name: apple-ui-guidelines
description: 'Apple プラットフォーム（iOS / iPadOS / macOS）向け UI ガイドライン。Human Interface Guidelines (HIG)、Liquid Glass、SwiftUI レイアウト、Dynamic Type、SF Symbols、VoiceOver、サイズクラス、ナビゲーション構造、ツールバー、シート、アイコンを確認・適用したいときに使用。Use when: designing or implementing UI for iOS, iPadOS, or macOS apps with SwiftUI; applying Apple HIG; adopting Liquid Glass; reviewing Apple platform UI code.'
argument-hint: '対象プラットフォーム（iOS / iPadOS / macOS）と確認したい項目（省略可）'
---

# Apple UI ガイドライン（iOS / iPadOS / macOS）

## 概要

このスキルは Apple プラットフォーム（iOS 26 / iPadOS 26 / macOS 26 Tahoe 以降）向け UI の設計・実装規約を定義します。
Human Interface Guidelines (HIG) に準拠し、Liquid Glass マテリアルを正しく扱うためのルールをまとめています。

参考:
- [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Adopting Liquid Glass](https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass)

---

## プラットフォームタグ

各セクションの指示には以下のタグが付くことがあります。対象プラットフォームに応じて取捨選択してください。

- **【iOS】** iPhone（iOS）向け
- **【iPadOS】** iPad 向け
- **【macOS】** Mac 向け
- **【共通】** 全プラットフォーム適用
- タグなし = 共通

---

## 1. HIG 準拠ルール（共通）

- **SF Symbols を優先使用する。** テキストラベルよりもアイコンを活用し、インターフェースをクリーンに保つ。タブバー・ツールバーでは filled バリアントを使用する。
- **システムカラー・アクセントカラーを使用する。** ハードコードされた色の代わりに `Color.accentColor` や `ShapeStyle` のセマンティックカラーを使用する。
- **ライト / ダークモード両対応を必須とする。** カスタムカラーは Light・Dark・増加コントラスト（Increased Contrast）の各バリアントを定義する。
- **コントロールを密集・重複させない。** Liquid Glass 要素をレイヤーとして重ねない。
- **標準スペーシングメトリクスを使用する。** システムのデフォルトスペーシングを上書きしない。
- **VoiceOver / Voice Control に対応する。** すべてのカスタム UI に適切な `accessibilityLabel` / `accessibilityHint` を設定する。
- **Dynamic Type に対応する。** フォントには必ずシステムフォント（`.body`・`.headline` 等）または `Font.custom(_:size:relativeTo:)` を使用し、固定サイズを避ける。
- **【iOS / iPadOS】Safe Area を尊重する。** ノッチ・Dynamic Island・ホームインジケーター領域にインタラクティブな要素を配置しない。

---

## 2. Liquid Glass 基本方針（共通）

- **標準コンポーネントを最大活用する。** `NavigationStack` / `NavigationSplitView` / `TabView` / ツールバー / シート / ポップオーバーは Liquid Glass を自動適用する。
- **ナビゲーション要素へのカスタム背景適用を禁止する。** タブバー・ナビゲーションバー・ツールバー・サイドバー・シートに独自の `background` / `visualEffect` を設定しない。
  - **【macOS 例外】** 没入型コンテンツアプリ（写真・地図など）では `.toolbarBackground(.hidden, for: .windowToolbar)` が許容される（HIG: "Consider temporarily hiding toolbars for a distraction-free experience"）。
- **`glassEffect` の過剰使用を禁止する。** `glassEffect(_:in:)` はカスタムコントロールなど最も重要な機能要素に限定する。
- **アクセシビリティ設定でテストする。** 「透明度を下げる」「視差効果を減らす」を有効にしてもカスタムエフェクトが適切に動作することを確認する。

---

## 3. ナビゲーション構造

### 【iOS】iPhone

- **`NavigationStack`** を基本とする。

```swift
NavigationStack(path: $path) {
    ContentView()
        .navigationDestination(for: Item.self) { item in
            DetailView(item: item)
        }
}
```

### 【iPadOS / macOS】

- **`NavigationSplitView`** でサイドバーレイアウトを実現する。

```swift
NavigationSplitView {
    SidebarView()
} detail: {
    DetailView()
}
```

- インスペクターパネルは **`inspector(isPresented:content:)`** を使用する。
- サイドバー・インスペクター隣のコンテンツには **`backgroundExtensionEffect()`** を適用してエッジトゥエッジ表現を実現する。

```swift
Image("hero")
    .resizable()
    .scaledToFill()
    .backgroundExtensionEffect()
```

---

## 4. タブバー

### 【iOS / iPadOS】

| 項目 | iOS | iPadOS |
|------|-----|--------|
| 表示位置 | 画面下部（フローティング） | 画面上部 |
| サイドバー変換 | 非対応 | `.sidebarAdaptable` で変換可 |
| カスタマイズ | — | `TabViewCustomization` で項目追加・削除可 |

```swift
TabView {
    Tab("ホーム", systemImage: "house.fill") { HomeView() }
    Tab("ライブラリ", systemImage: "books.vertical.fill") { LibraryView() }
    Tab(role: .search) { SearchView() }
}
.tabViewStyle(.sidebarAdaptable) // 【iPadOS】
```

- **【iOS】** スクロール時にタブバーを縮小する: `.tabBarMinimizeBehavior(.onScrollDown)`
- **【iPadOS】** `TabViewCustomization` でユーザーがタブ項目を追加・削除できるようにする。
- タブバーを無効化・非表示にしない。コンテンツが空でも理由を説明しつつタブを表示し続ける。
- タブラベルは単語単位で簡潔に。
- タブアイコンは SF Symbols の filled バリアントを優先。

### 【macOS】

- タブビューには **`.tabViewStyle(.sidebarAdaptable)`** を採用し、サイドバーへ自動変換させる。

---

## 5. ツールバー

- ツールバーアイテムは機能ごとにグループ化し、`ToolbarSpacer` で区切る。

```swift
.toolbar {
    ToolbarItemGroup(placement: .bottomBar) { // 【iOS】
        Button("編集", systemImage: "pencil") { }
        Button("削除", systemImage: "trash") { }
    }
    ToolbarSpacer(.fixed)
    ToolbarItemGroup(placement: .bottomBar) {
        Button("共有", systemImage: "square.and.arrow.up") { }
    }
}
```

- アイコンのみのアイテムには必ず `accessibilityLabel` を設定。
- 非表示は `.hidden()` ではなく **`ToolbarContent/hidden(_:)`** を使用。
- スクロール時の可読性確保に **`scrollEdgeEffectStyle`** を設定。

---

## 6. コントロール

- グラス効果のボタンは **`.buttonStyle(.glass)`** / **`.buttonStyle(.glassProminent)`** を使用。

```swift
Button("追加") { }.buttonStyle(.glass)
Button("確認") { }.buttonStyle(.glassProminent)
```

- カスタムコントロールの角丸は **`ConcentricRectangle`** または **`rect(corners:isUniform:)`** で周囲要素と同心円的に揃える。
- 複数のカスタム Liquid Glass エフェクトは **`GlassEffectContainer`** + **`glassEffectID(_:in:)`** でまとめる。

```swift
GlassEffectContainer {
    ForEach(items) { item in
        ItemView(item: item)
            .glassEffect(.regular, in: .capsule)
            .glassEffectID(item.id, in: namespace)
    }
}
```

---

## 7. シート・モーダル

- **【iOS / iPadOS】** iOS 26 のシートは角丸が増加し、ハーフシートは画面端からインセット表示される。コンテンツが角丸付近に重ならないよう余白を確保。
- **`presentationDetents`** で適切なサイズ制御。

```swift
.sheet(isPresented: $showSheet) {
    SheetContentView()
        .presentationDetents([.medium, .large])
        .presentationDragIndicator(.visible)
}
```

- シート・ポップオーバーのカスタム背景ビュー（`visualEffectView` 等）は削除し、システムの Liquid Glass 背景に委ねる。
- アクションシートは **`confirmationDialog`** を使用し、`presenting` パラメーターで起点データを指定。

```swift
.confirmationDialog(
    "操作を選択",
    isPresented: $showDialog,
    titleVisibility: .visible,
    presenting: selectedItem
) { item in
    Button("削除", role: .destructive) { delete(item) }
    Button("共有") { share(item) }
}
```

---

## 8. リスト・フォーム

- フォームには **`.formStyle(.grouped)`** を使用。

```swift
Form {
    Section("設定") {
        Toggle("通知", isOn: $notificationsEnabled)
        Slider(value: $volume, in: 0...1)
    }
}
.formStyle(.grouped)
```

- `Section` ヘッダーは **Title Case**（ALL CAPS は使わない）。
- **【iOS / iPadOS】** コンテキストメニュー先頭アクションとスワイプアクション先頭を一致させる。

```swift
.swipeActions(edge: .trailing, allowsFullSwipe: true) {
    Button("削除", role: .destructive) { delete(item) }
}
.contextMenu {
    Button("削除", role: .destructive) { delete(item) }
    Button("共有") { share(item) }
}
```

---

## 9. 検索

- 検索タブは **`Tab(role: .search)`** で定義。システムがトレーリング端に自動配置する。
- **【iOS】** 検索フィールドはボトムツールバー内に配置。
- **【iPadOS】** 画面上部トレーリング端に自動配置。

---

## 10. タイポグラフィ

- Dynamic Type スケール（`.largeTitle` / `.title` / `.headline` / `.body` / `.callout` / `.subheadline` / `.footnote` / `.caption`）を使用。
- カスタムフォントは `Font.custom(_:size:relativeTo:)` で Dynamic Type に追従させる。
- 同一画面内でフォントウェイトは 2〜3 種類に絞る。
- `tracking` / `lineSpacing` の独自設定は最小限。

```swift
Text("タイトル")
    .font(.title2)
    .fontWeight(.semibold)
Text("説明文")
    .font(.body)
    .foregroundStyle(.secondary)
```

---

## 11. スペーシング・グリッド

- **8pt グリッド** を基準。`8, 16, 24, 32` の倍数を使用。
- マジックナンバー禁止。スペーシング定数を定義。

```swift
enum Spacing {
    static let xs: CGFloat = 8
    static let sm: CGFloat = 16
    static let md: CGFloat = 24
    static let lg: CGFloat = 32
}
```

- 近い要素は近く、異なるグループは広い余白で区切る。

---

## 12. コンテンツファーストレイアウト

- Liquid Glass 上のナビゲーション要素背後にコンテンツを透過させる。フルブリード配置を意識。
- **【iOS / iPadOS】** ヒーローイメージには **`ignoresSafeArea(.container, edges: .top)`**。
- **【macOS / iPadOS】** サイドバー隣には **`backgroundExtensionEffect()`**。
- `contentMargins` / `safeAreaPadding` / `scrollEdgeEffectStyle` を適切に設定。

---

## 13. アニメーション・トランジション

- 基本アニメーションは **`.animation(.spring(duration: 0.3), value:)`**。`.linear` は特別な理由なく使わない。
- 要素の変容には **`matchedGeometryEffect`**。
- Liquid Glass のモーフィングには **`glassEffectID(_:in:)` + `withAnimation`**。
- Reduce Motion 対応:

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

.animation(reduceMotion ? .none : .spring(duration: 0.3), value: isExpanded)
```

---

## 14. カラー設計

- セマンティックカラーを階層的に:
  - 最重要: `.primary`
  - 補助: `.secondary`
  - より補助的: `.tertiary`
  - 無効: `.quaternary`
- アクセントカラーは `Color.accentColor`。ハードコード RGB を避ける。
- Liquid Glass 上のテキストは `.shadow(radius:)` や `.foregroundStyle(.primary)` で視認性確保。
- カスタムカラーは Assets.xcassets に Light / Dark / Increased Contrast の 3 バリアントを定義。

---

## 15. サイズクラス対応【iOS / iPadOS】

- `@Environment(\.horizontalSizeClass)` でレイアウト幅クラスを判定:
  - **Compact 幅**: iPhone（縦横とも）、iPad 縦向きの狭いウィンドウ
  - **Regular 幅**: iPad 横向き・広いウィンドウ、**iPhone Pro Max / Plus の横向き**

```swift
@Environment(\.horizontalSizeClass) var horizontalSizeClass

var body: some View {
    if horizontalSizeClass == .compact {
        VStack { ... }
    } else {
        HStack { ... }
    }
}
```

- `@Environment(\.verticalSizeClass)`:
  - 縦向き: `regular`
  - 横向き（全 iPhone 共通）: `compact`

---

## 16. ハプティクス【iOS】

- **`.sensoryFeedback(_:trigger:)`** を使用。`UIImpactFeedbackGenerator` は SwiftUI 非対応時のみ。

```swift
Button("削除") { delete() }
    .sensoryFeedback(.warning, trigger: isDeleted)
Toggle("通知", isOn: $enabled)
    .sensoryFeedback(.selection, trigger: enabled)
```

- 対応:
  - 成功: `.success`
  - 警告: `.warning`
  - エラー: `.error`
  - 選択: `.selection`
  - 軽いタップ: `.impact(weight: .light)`

---

## 17. 空状態・エラー状態

- **`ContentUnavailableView`** を使用する。独自の空画面は作らない。

```swift
if items.isEmpty {
    ContentUnavailableView(
        "アイテムがありません",
        systemImage: "tray",
        description: Text("新しいアイテムを追加してください。")
    )
}

ContentUnavailableView.search(text: searchText)
```

---

## 18. ローディング / スケルトン UI

- **`.redacted(reason: .placeholder)`** でスケルトン表示。`ProgressView()` の全画面表示は避ける。

```swift
ItemRowView(item: placeholderItem)
    .redacted(reason: isLoading ? .placeholder : [])
```

---

## 19. アダプティブレイアウト

- **`ViewThatFits`** で代替レイアウトを提供。

```swift
ViewThatFits {
    HStack { LabelView(); ValueView() }
    VStack { LabelView(); ValueView() }
}
```

- 固定幅 `frame(width:)` を避け、`.frame(maxWidth: .infinity)` / `.fixedSize()` を優先。
- **【iOS】** **`containerRelativeFrame`** で機種差（幅 375〜440 pt）を吸収。
- `GeometryReader` の過剰使用を避ける。

---

## 20. キーボード・フォーカス管理

- **`@FocusState`** でフォーカスを明示的に管理。

```swift
@FocusState private var isFieldFocused: Bool

TextField("名前", text: $name)
    .focused($isFieldFocused)
```

- **【iPadOS / macOS】** 主要アクションに **`KeyboardShortcut`** を割り当てる。

```swift
Button("新規作成") { createItem() }
    .keyboardShortcut("n", modifiers: .command)
Button("保存") { save() }
    .keyboardShortcut("s", modifiers: .command)
```

- **【macOS】** macOS はキーボードファースト。標準慣習（⌘N, ⌘S, ⌘W 等）に従う。Tab フォーカス順序を論理的に設計。

---

## 21. ウィンドウ【macOS】

- 任意サイズへのリサイズをサポートし、適切な最小サイズを設定。
- `NavigationSplitView` でリサイズ時のフルードトランジションを自動取得。
- `safeAreaInsets` / レイアウトガイドを正しく設定し、ウィンドウコントロールとタイトルバーの重なりを防ぐ。
- `NavigationSplitView` の列幅は固定しない。

---

## 22. アプリアイコン

**Icon Composer**（Xcode 26 内蔵）でレイヤー構造のアイコンを作成する。

- レイヤー構成: 前景 / 中景 / 背景（システムが反射・屈折・シャドウ・ブラーを自動適用）
- 不規則形状にはシステムが自動でバックグラウンドを付与
- 要素はアイコン中央に配置、角丸クリッピングを考慮

### 必要なバリアント

| プラットフォーム | バリアント |
|---|---|
| 【iOS / iPadOS】 | Default (Light) / Dark / Clear / Tinted |
| 【macOS】 | Default (Light) / Dark / Clear (Light) / Clear (Dark) / Tinted (Light) / Tinted (Dark) |

---

## 23. 画面サイズ対応テスト【iOS】

HIG は「最大・最小レイアウトを先にテストせよ」と明言。以下機種で必ず確認:

| テスト対象 | 幅 | 高さ | 確認ポイント |
|---|---|---|---|
| iPhone SE（4.7-inch） | 375 | 667 | 最小クラス。コンテンツ欠損なし |
| iPhone 17 Pro Max | 440 | 956 | 最大クラス。横向き時 Regular 幅 |

確認事項:
- テキスト・コントロールが Safe Area 内に収まる
- Dynamic Type 最大サイズ（Accessibility XL）で崩れない
- 横向きで `verticalSizeClass` ベースの切替が正しい

---

## 関連スキル

- アクセシビリティ全般: `skills/ui-accessibility/SKILL.md`
- UI レビュー: `skills/ui-review-checklist/SKILL.md`
- Swift コーディング規約: `skills/swift-coding-standards/SKILL.md`
