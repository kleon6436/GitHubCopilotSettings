---
name: windows-ui-guidelines
description: 'Windows 向け UI ガイドライン。Fluent Design System、WinUI 3 / WPF、Mica / Acrylic マテリアル、タイトルバー統合、Narrator アクセシビリティ、タッチ・マウス・ペン入力、Windows 11 デザイン原則を確認・適用したいときに使用。Use when: designing or implementing Windows desktop UI with WinUI 3, WPF, or MAUI; applying Fluent Design; reviewing Windows app UI.'
argument-hint: '確認したい項目（Fluent / Mica / アクセシビリティ など、省略可）'
---

# Windows UI ガイドライン

## 概要

このスキルは Windows 11 以降のデスクトップアプリ向け UI 規約を定義します。
Microsoft の **Fluent Design System** に準拠し、WinUI 3 / WPF / .NET MAUI で一貫した体験を提供するためのルールをまとめています。

参考:
- [Fluent Design System](https://learn.microsoft.com/windows/apps/design/)
- [Windows 11 Design principles](https://learn.microsoft.com/windows/apps/design/signature-experiences/design-principles)
- [WinUI 3 documentation](https://learn.microsoft.com/windows/apps/winui/winui3/)

---

## 1. Fluent Design 基本原則

- **Effortless** — 目的達成までの摩擦を減らす。標準コントロールを使用し、独自実装を最小限に。
- **Calm** — 視覚的ノイズを抑え、コンテンツを主役にする。過剰なアニメーション・装飾を避ける。
- **Personal** — ユーザーのテーマ（Light / Dark / アクセントカラー）・システム設定を尊重する。
- **Familiar** — Windows 11 の標準パターン（ナビゲーションビュー、コマンドバー、コンテキストメニュー）に従う。
- **Complete** — タッチ・マウス・キーボード・ペン・ゲームパッドすべての入力に対応する。

---

## 2. 推奨技術スタック

| 用途 | 推奨 | 備考 |
|---|---|---|
| 新規デスクトップアプリ | **WinUI 3** + Windows App SDK | 最新の Fluent、Mica、Windows 11 統合 |
| 既存 WPF 継続 | **WPF** + ModernWpf / WPF UI | Fluent テーマを外部ライブラリで適用 |
| クロスプラットフォーム | **.NET MAUI** | Windows / macOS / iOS / Android |
| Web ベース | **WebView2** ホスト + Fluent UI Web | ハイブリッドアプリ向け |

---

## 3. マテリアル（Mica / Acrylic）

Windows 11 では背景マテリアルにより階層と奥行きを表現する。

| マテリアル | 用途 | 実装 |
|---|---|---|
| **Mica** | ウィンドウ背景（長時間表示される領域） | `SystemBackdrop="Mica"` |
| **Mica Alt** | タブ付きウィンドウ | `SystemBackdrop="MicaAlt"` |
| **Acrylic** | 一時的 UI（フライアウト、メニュー、ダイアログ） | `AcrylicBrush` |
| **Smoke** | モーダル背景 | `SystemBackdrop="Smoke"` |

```xml
<!-- WinUI 3: ウィンドウへの Mica 適用 -->
<Window x:Class="MyApp.MainWindow"
        xmlns="...">
    <Window.SystemBackdrop>
        <MicaBackdrop Kind="Base" />
    </Window.SystemBackdrop>
</Window>
```

- **Mica は不透明度・色を上書きしない。** ユーザーのデスクトップ壁紙・アクセントカラーを尊重する。
- Acrylic はコンテンツ領域の背景に使わない（パフォーマンス・可読性に影響）。フライアウトやタスクペインに限定。

---

## 4. タイトルバー統合

Windows 11 ではタイトルバーをアプリコンテンツと統合する（`ExtendsContentIntoTitleBar`）。

```csharp
// WinUI 3
this.ExtendsContentIntoTitleBar = true;
this.SetTitleBar(AppTitleBar);
```

- タイトルバーにはアプリアイコン・タイトル・検索ボックス・プロファイル等を配置可能。
- ドラッグ領域（`IsHitTestVisible="False"`）と操作領域を明確に分ける。
- Caption Button（最小化/最大化/閉じる）は OS に任せ、位置・サイズを再現しない。

---

## 5. ナビゲーション構造

### NavigationView（推奨）

トップレベルナビゲーションには **`NavigationView`** を使用する。

```xml
<NavigationView PaneDisplayMode="Auto"
                IsBackButtonVisible="Auto">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Home" Icon="Home" Tag="home" />
        <NavigationViewItem Content="Library" Icon="Library" Tag="library" />
    </NavigationView.MenuItems>
    <Frame x:Name="ContentFrame" />
</NavigationView>
```

- `PaneDisplayMode="Auto"` でウィンドウ幅に応じて Left / Top / LeftCompact / LeftMinimal を自動切替。
- 5 項目以下は Top、6 項目以上は Left を推奨。
- 設定は **`IsSettingsVisible="True"`** でフッター固定。

### TabView（ドキュメント型アプリ）

複数ドキュメントを扱う場合は **`TabView`**（ブラウザ・エディタスタイル）。

---

## 6. コマンド表面（コマンドバー）

| 表面 | 用途 |
|---|---|
| **CommandBar** | ページ上部・下部の主要アクション群 |
| **CommandBarFlyout** | コンテキストメニュー・選択時アクション |
| **MenuFlyout** | 右クリック・オーバーフロー |

```xml
<CommandBar DefaultLabelPosition="Right">
    <AppBarButton Icon="Add" Label="New" />
    <AppBarButton Icon="Save" Label="Save" />
    <AppBarSeparator />
    <AppBarButton Icon="Share" Label="Share" />
</CommandBar>
```

- 重要度順に **Primary Command**（左 or 上）、二次は Secondary Command（オーバーフロー）。
- アイコンには [Segoe Fluent Icons](https://learn.microsoft.com/windows/apps/design/style/segoe-fluent-icons-font) を使用（Windows 11）。

---

## 7. コントロール

- 標準 WinUI コントロールを優先（`Button` / `ToggleSwitch` / `Slider` / `ComboBox` / `CalendarDatePicker` 等）。
- ボタンスタイル:
  - **AccentButtonStyle**: 画面内の主要 1 アクションのみ
  - **DefaultButton**: 通常操作
  - **SubtleButtonStyle**: 補助操作
- 破壊的操作は `Foreground="{ThemeResource SystemFillColorCriticalBrush}"` で赤系に。

```xml
<Button Content="保存"
        Style="{ThemeResource AccentButtonStyle}" />
```

---

## 8. タイポグラフィ

Windows 11 の **Segoe UI Variable** を基本フォントとする。

| スタイル | サイズ | 用途 |
|---|---|---|
| Caption | 12 | 補助情報 |
| Body | 14 | 本文 |
| Body Strong | 14 Semibold | 強調本文 |
| Body Large | 18 | 本文（強調） |
| Subtitle | 20 Semibold | サブ見出し |
| Title | 28 Semibold | セクション見出し |
| Title Large | 40 Semibold | ページ見出し |
| Display | 68 Semibold | ヒーロー見出し |

```xml
<TextBlock Text="見出し" Style="{ThemeResource TitleTextBlockStyle}" />
<TextBlock Text="本文" Style="{ThemeResource BodyTextBlockStyle}" />
```

- カスタムフォントは Segoe UI Variable を優先し、日本語は Yu Gothic UI にフォールバック。
- ユーザーの「テキストのサイズ」設定（100% 〜 225%）でレイアウトが崩れないことを確認。

---

## 9. カラー・テーマ

- **テーマリソース**（`{ThemeResource ...}`）を使用し、ハードコード色を避ける。
  - `SystemAccentColor` — システムアクセント
  - `TextFillColorPrimaryBrush` / `TextFillColorSecondaryBrush` — テキスト
  - `SolidBackgroundFillColorBaseBrush` — ベース背景
- Light / Dark / High Contrast の 3 テーマで動作検証。
- アクセントカラーはシステム設定を尊重（ユーザー選択を上書きしない）。

---

## 10. スペーシング・グリッド

- **4px グリッド** が基本（4, 8, 12, 16, 20, 24, 32, 40）。
- ページ端パディング: 左右 **24px** 以上。
- コントロール間隔: 垂直 **12px**、水平 **8px** を基準。

```xml
<Thickness x:Key="PageMargin">24,24,24,24</Thickness>
<StackPanel Spacing="12" />
```

---

## 11. 入力対応

Windows は複数入力をネイティブにサポートする。

| 入力 | 考慮点 |
|---|---|
| **マウス** | ホバー状態・右クリック対応 |
| **キーボード** | Tab 順序、アクセラレータ（Alt+キー）、ショートカット（Ctrl+S 等） |
| **タッチ** | ターゲット最小 **40×40 ピクセル**（推奨 48×48） |
| **ペン** | インク入力対応（`InkCanvas`）、ホバー（Windows Ink） |
| **ゲームパッド** | Xbox Game Bar / Xbox アプリ向け。XY フォーカスナビゲーション |

```xml
<!-- キーボードアクセラレータ -->
<Button Content="保存">
    <Button.KeyboardAccelerators>
        <KeyboardAccelerator Key="S" Modifiers="Control" />
    </Button.KeyboardAccelerators>
</Button>
```

- `AccessKey="S"` で Alt ナビゲーションに対応。

---

## 12. アニメーション・モーション

- **Connected Animation** で要素の遷移を滑らかに表現。

```csharp
ConnectedAnimationService.GetForCurrentView()
    .PrepareToAnimate("forwardAnimation", sourceElement);
```

- 標準イージング `FluentAnimations.Standard` を使用。
- モーション持続時間は **150〜300ms** が基本。
- システム設定「アニメーション効果を表示する」OFF 時はアニメーションをスキップ。

```csharp
if (UISettings.AnimationsEnabled) { /* animate */ }
```

---

## 13. アクセシビリティ

- **Narrator** で全画面を操作可能にする。
- `AutomationProperties.Name` / `AutomationProperties.HelpText` を全インタラクティブ要素に設定。

```xml
<Button Icon="Delete"
        AutomationProperties.Name="削除"
        ToolTipService.ToolTip="削除" />
```

- コントラスト比は **WCAG 2.2 AA 以上**（通常テキスト 4.5:1、大テキスト 3:1）。
- **High Contrast テーマ**で全 UI が視認可能であること。`SystemColor*Brush` を使用。
- Tab フォーカスは論理順序、`TabIndex` で明示的に制御可能。
- 自動フォーカスリング（XAML の既定）を隠さない。
- 詳細は `skills/ui-accessibility/SKILL.md` 参照。

---

## 14. レスポンシブレイアウト

- **Adaptive Triggers** でウィンドウ幅に応じたレイアウト切替。

```xml
<VisualStateManager.VisualStateGroups>
    <VisualStateGroup>
        <VisualState x:Name="Narrow">
            <VisualState.StateTriggers>
                <AdaptiveTrigger MinWindowWidth="0" />
            </VisualState.StateTriggers>
        </VisualState>
        <VisualState x:Name="Wide">
            <VisualState.StateTriggers>
                <AdaptiveTrigger MinWindowWidth="720" />
            </VisualState.StateTriggers>
        </VisualState>
    </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

- ブレークポイント目安: **640 / 1008 / 1366 / 1920 px**。
- 最小ウィンドウサイズ: 幅 **500px**、高さ **320px** 以上を確保。
- タブレットモード・縦向きも考慮。

---

## 15. ダイアログ・Teaching Tip

- モーダル確認は **`ContentDialog`**。

```csharp
var dialog = new ContentDialog
{
    Title = "削除しますか？",
    Content = "この操作は元に戻せません。",
    PrimaryButtonText = "削除",
    CloseButtonText = "キャンセル",
    DefaultButton = ContentDialogButton.Close,
    XamlRoot = this.XamlRoot
};
```

- 新機能案内は **`TeachingTip`**。
- トースト通知は **Windows App Notification**（旧 Toast Notification）。

---

## 16. アイコン

- システムアイコンは **Segoe Fluent Icons**（Windows 11）。
- カスタムアイコンは SVG 推奨（`PathIcon`）。
- サイズは **16 / 20 / 24 / 32 / 48 px** を基本グリッドに。
- アプリアイコンは `.ico`（16 / 24 / 32 / 48 / 64 / 256）と `StoreLogo` / `Square44x44Logo` / `Square150x150Logo` / `Wide310x150Logo` を Package.appxmanifest に登録。

---

## 17. Windows 11 統合機能

- **Snap Layouts** — 最大化ボタンホバーでレイアウト候補を表示。ウィンドウサイズ制約に注意。
- **Widgets** — Adaptive Cards ベース。`WindowsAppSDK` で対応。
- **Share Charm** — `DataTransferManager` で共有元として登録。
- **File Picker** — `FileOpenPicker` / `FileSavePicker` を使用し、独自実装しない。

---

## 関連スキル

- アクセシビリティ全般: `skills/ui-accessibility/SKILL.md`
- UI レビュー: `skills/ui-review-checklist/SKILL.md`
