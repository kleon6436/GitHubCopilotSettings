---
name: android-ui-guidelines
description: 'Android 向け UI ガイドライン。Material Design 3 (Expressive)、Jetpack Compose、Dynamic Color、Edge-to-Edge、WindowSizeClass、Predictive Back、Large Screen 対応、TalkBack アクセシビリティ、タッチ・ジェスチャーを確認・適用したいときに使用。Use when: designing or implementing Android app UI with Jetpack Compose or View system; applying Material 3; reviewing Android UI code.'
argument-hint: '確認したい項目（Material / Compose / Large Screen など、省略可）'
---

# Android UI ガイドライン

## 概要

このスキルは Android アプリ向け UI 規約を定義します。
**Material Design 3（Material You / Expressive）** と **Jetpack Compose** を中心に、Phone / Foldable / Tablet / ChromeOS まで対応する体験を作るためのルールをまとめています。

参考:
- [Material Design 3](https://m3.material.io/)
- [Android Developers - UI](https://developer.android.com/develop/ui)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Large Screen guidelines](https://developer.android.com/guide/topics/large-screens)

---

## 1. 基本原則

- **Jetpack Compose を第一選択にする。** 新規画面は Compose、既存 View 資産は段階移行。
- **Material Design 3（M3）に準拠する。** 独自テーマより `MaterialTheme` をカスタマイズ。
- **Dynamic Color を尊重する。** Android 12+ ではユーザーの壁紙由来のカラーを使用。
- **Edge-to-Edge をデフォルト化する。** Android 15（API 35）+ は Edge-to-Edge が強制される。
- **すべての画面サイズで動作させる。** Phone / Foldable 折りたたみ・展開 / Tablet / ChromeOS。
- **TalkBack で操作可能に。** すべてのインタラクティブ要素にコンテンツ説明を付与。

---

## 2. 推奨技術スタック

| カテゴリ | 推奨 |
|---|---|
| UI フレームワーク | **Jetpack Compose**（`androidx.compose.*`） |
| デザインシステム | **Material 3**（`androidx.compose.material3`） |
| ナビゲーション | **Navigation Compose**（Type-safe Navigation） |
| 画像読み込み | **Coil** |
| 非同期 | **Coroutines** + **Flow** |
| アダプティブ | **Material 3 Adaptive**（`androidx.compose.material3.adaptive`） |
| アニメーション | **Compose Animation** + **Motion** |

---

## 3. テーマ（Material 3）

### 基本テーマ

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> darkColorScheme(/* brand colors */)
        else -> lightColorScheme(/* brand colors */)
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        shapes = AppShapes,
        content = content,
    )
}
```

### カラーロール（M3）

| ロール | 用途 |
|---|---|
| `primary` / `onPrimary` | 主要アクション |
| `secondary` / `onSecondary` | 補助アクション |
| `tertiary` / `onTertiary` | コントラスト用アクセント |
| `surface` / `onSurface` | カード・シート背景 |
| `surfaceContainer*` | 階層化されたコンテナ背景 |
| `error` / `onError` | エラー・破壊的操作 |

- ハードコード色（`Color(0xFF...)`）を避け、`MaterialTheme.colorScheme.primary` を使う。

---

## 4. タイポグラフィ（Type Scale）

Material 3 タイプスケール:

| スタイル | 用途 |
|---|---|
| `displayLarge` / `displayMedium` / `displaySmall` | ヒーロー見出し |
| `headlineLarge` / `headlineMedium` / `headlineSmall` | 画面見出し |
| `titleLarge` / `titleMedium` / `titleSmall` | セクションタイトル、ダイアログ見出し |
| `bodyLarge` / `bodyMedium` / `bodySmall` | 本文 |
| `labelLarge` / `labelMedium` / `labelSmall` | ボタン・タブラベル |

```kotlin
Text(
    text = "タイトル",
    style = MaterialTheme.typography.headlineMedium,
)
```

- フォントは可変フォント（Roboto Flex / Google Sans）を優先。日本語は Noto Sans CJK JP。
- ユーザーのフォントスケール設定（Settings → 表示 → フォントサイズ）に追従。`sp` 単位を使い、`dp` は使わない（テキスト用）。

---

## 5. レイアウト・スペーシング

- **8dp グリッド** を基本。`4, 8, 12, 16, 24, 32, 40, 48` dp を使う。
- スクリーン端マージン: **16dp**（Compact）/ **24dp**（Medium 以上）。
- トークン化:

```kotlin
object Spacing {
    val xs = 4.dp
    val sm = 8.dp
    val md = 16.dp
    val lg = 24.dp
    val xl = 32.dp
}
```

- タッチターゲット最小 **48×48dp**（Material 推奨・WCAG 2.5.5 準拠）。

---

## 6. コンポーネント（M3）

### 主要コンポーネント

| 用途 | コンポーネント |
|---|---|
| メインアクション | `Button`（Filled） / `FilledTonalButton` / `OutlinedButton` / `TextButton` |
| FAB | `FloatingActionButton` / `ExtendedFloatingActionButton` |
| トップバー | `TopAppBar` / `CenterAlignedTopAppBar` / `MediumTopAppBar` / `LargeTopAppBar` |
| ボトムバー | `BottomAppBar` / `NavigationBar` |
| ナビゲーション | `NavigationBar`（≤ 5 項目）/ `NavigationRail`（Medium+）/ `NavigationDrawer`（Expanded） |
| カード | `Card` / `ElevatedCard` / `OutlinedCard` |
| リスト | `ListItem` |
| 入力 | `TextField`（Filled） / `OutlinedTextField` |
| 選択 | `Checkbox` / `RadioButton` / `Switch` / `Slider` |
| フィードバック | `Snackbar` / `LinearProgressIndicator` / `CircularProgressIndicator` |
| ダイアログ | `AlertDialog` / `DatePickerDialog` / `TimePickerDialog` / `ModalBottomSheet` |

### ボタン階層

- **FAB**: 画面内の 1 つの主要アクションのみ。
- **Filled Button**: 重要だが FAB 化しないアクション。
- **Tonal Button**: 中程度の強調。
- **Outlined Button**: 代替・セカンダリ。
- **Text Button**: 破壊的でない補助アクション。

```kotlin
Button(onClick = { /* ... */ }) { Text("保存") }
OutlinedButton(onClick = { /* ... */ }) { Text("キャンセル") }
```

---

## 7. ナビゲーション

### アダプティブナビゲーション

`WindowSizeClass` に応じてナビゲーションを切り替える:

| WindowSizeClass | ナビゲーション |
|---|---|
| **Compact**（幅 < 600dp） | `NavigationBar`（下部） |
| **Medium**（600 ≤ 幅 < 840dp） | `NavigationRail`（左） |
| **Expanded**（幅 ≥ 840dp） | `PermanentNavigationDrawer` or `NavigationRail` |

```kotlin
val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass

NavigationSuiteScaffold(
    navigationSuiteItems = {
        items.forEach { item ->
            item(
                icon = { Icon(item.icon, null) },
                label = { Text(item.label) },
                selected = item.selected,
                onClick = item.onClick,
            )
        }
    },
) {
    // content
}
```

- **Navigation Compose** の **Type-safe Navigation**（kotlinx-serialization ベース）を使用。

```kotlin
@Serializable data object Home
@Serializable data class Detail(val id: Long)

NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen(onOpenDetail = { navController.navigate(Detail(it)) }) }
    composable<Detail> { backStack ->
        val detail: Detail = backStack.toRoute()
        DetailScreen(id = detail.id)
    }
}
```

### Predictive Back

Android 14+ の **Predictive Back** に対応:

```kotlin
// AndroidManifest
// android:enableOnBackInvokedCallback="true"

PredictiveBackHandler(enabled = hasUnsavedChanges) { progress ->
    try {
        progress.collect { /* アニメーション進捗 */ }
        // ジェスチャー完了 → 戻る処理
    } catch (e: CancellationException) {
        // キャンセル
    }
}
```

---

## 8. Edge-to-Edge / インセット

Android 15（API 35）+ は Edge-to-Edge が **デフォルト強制**。

```kotlin
// Activity
override fun onCreate(savedInstanceState: Bundle?) {
    enableEdgeToEdge()
    super.onCreate(savedInstanceState)
    setContent { AppTheme { App() } }
}
```

- **WindowInsets** を適切に消費:

```kotlin
Scaffold(
    topBar = { TopAppBar(...) }, // インセット自動適用
    contentWindowInsets = WindowInsets.safeDrawing,
) { padding ->
    Column(Modifier.padding(padding)) { ... }
}

// カスタム要素
Box(Modifier.windowInsetsPadding(WindowInsets.systemBars)) { ... }
```

- ステータスバー・ナビゲーションバー背後にコンテンツを配置し、適切なインセットでコントロールを避ける。

---

## 9. Large Screen / Foldable 対応

- **WindowSizeClass** を使い、アプリ全体で **List-Detail** / **Supporting Pane** / **Feed** パターンを適用。

```kotlin
val navigator = rememberListDetailPaneScaffoldNavigator<Item>()

ListDetailPaneScaffold(
    directive = navigator.scaffoldDirective,
    value = navigator.scaffoldValue,
    listPane = { AnimatedPane { ItemList(onSelect = { navigator.navigateTo(ListDetailPaneScaffoldRole.Detail, it) }) } },
    detailPane = {
        AnimatedPane {
            navigator.currentDestination?.content?.let { DetailScreen(it) }
        }
    },
)
```

- 折りたたみ・展開に対応するため、**`WindowInfoTracker`** でヒンジ・折り目情報を取得可能。
- 横向き・分割画面・マルチウィンドウで崩れないことをテスト。
- **ChromeOS / DeX** 対応: マウス・キーボード・リサイズ可能ウィンドウを考慮。

---

## 10. アニメーション・モーション

- 基本トランジション:
  - `AnimatedVisibility` — 要素の出入り
  - `AnimatedContent` — 内容切替
  - `Crossfade` — シンプルな切替
  - `animate*AsState` — 値の補間
- **Shared Element Transition**（Compose 1.7+）でヒーロートランジション。

```kotlin
SharedTransitionLayout {
    AnimatedContent(targetState = state) { target ->
        if (target) {
            Image(
                modifier = Modifier.sharedElement(
                    rememberSharedContentState("image"),
                    animatedVisibilityScope = this@AnimatedContent,
                )
            )
        }
    }
}
```

- イージングは Material Motion の `FastOutSlowInEasing` / `EmphasizedEasing` を使用。
- 持続時間: **100〜400ms**。
- **Remove Animations** 設定（Accessibility）に対応:

```kotlin
val reduceMotion = LocalAccessibilityManager.current?.let { /* check */ } ?: false
```

---

## 11. アクセシビリティ（TalkBack）

- `contentDescription` を画像・アイコンボタンに設定。装飾画像は `null`。

```kotlin
Icon(
    imageVector = Icons.Default.Delete,
    contentDescription = "削除",
)

IconButton(onClick = { /* ... */ }) {
    Icon(Icons.Default.Share, contentDescription = "共有")
}
```

- セマンティクスの明示:

```kotlin
Modifier.semantics {
    contentDescription = "お気に入りに追加"
    role = Role.Button
    stateDescription = if (isFavorite) "追加済み" else "未追加"
}
```

- コントラスト比 **WCAG AA**（4.5:1 / 3:1）。
- タッチターゲット **48×48dp**（`Modifier.minimumInteractiveComponentSize()` で保証）。
- フォントスケール **200%** で崩れないこと。
- カラーだけで情報を伝えない。
- 詳細は `skills/ui-accessibility/SKILL.md` 参照。

---

## 12. リスト・パフォーマンス

- **`LazyColumn` / `LazyRow` / `LazyVerticalGrid`** を使用。`Column` + `verticalScroll` でリスト描画しない。
- `key` パラメーターで安定した ID を指定し、不要な再コンポジションを回避。

```kotlin
LazyColumn {
    items(items, key = { it.id }) { item ->
        ItemRow(item)
    }
}
```

- 画像は **Coil** で遅延ロード + `crossfade`。
- 再コンポジション最適化: `remember` / `derivedStateOf` を適切に使う。
- `@Stable` / `@Immutable` アノテーションで Compose にヒントを与える。

---

## 13. 入力・ジェスチャー

- **IME（キーボード）対応**: `Modifier.imePadding()` でキーボード回避。`WindowInsets.ime` を使用。
- テキスト入力: `KeyboardOptions`（`keyboardType` / `imeAction` / `autoCorrect`）を適切に設定。
- ジェスチャー: `Modifier.pointerInput` や `detectTapGestures` / `detectDragGestures`。
- タッチとマウス（ChromeOS）両方を考慮。`Modifier.hoverable` でホバー対応。

---

## 14. アイコン

- **Material Symbols**（`material-icons-extended`）を使用。
- カスタムアイコンは Vector Drawable（SVG 変換）で提供。
- サイズは **20 / 24 / 32 / 40dp** を基本。
- アプリアイコンは **Adaptive Icon**（foreground / background レイヤー）+ **Monochrome**（Themed Icon 用）を提供。

```xml
<!-- res/mipmap-anydpi-v26/ic_launcher.xml -->
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
    <monochrome android:drawable="@drawable/ic_launcher_monochrome" />
</adaptive-icon>
```

---

## 15. 通知・システム連携

- 通知は **Notification Channel** に分類（Android 8+ 必須）。
- **POST_NOTIFICATIONS** 権限（Android 13+）を適切に要求。
- リッチ通知（画像・アクション・返信）・通知トレイでの展開挙動を確認。
- **App Shortcuts** / **Widgets**（Glance API）で常用機能を露出。

---

## 16. テスト

- **Compose UI Test** で画面ロジック検証（`composeTestRule`）。
- **Accessibility Scanner** で自動アクセシビリティ検査。
- **Espresso** + **UIAutomator** で E2E。
- **Baseline Profile** / **Macrobenchmark** で起動・スクロールパフォーマンス測定。

---

## 17. デザインプレビュー

`@Preview` でさまざまな設定を網羅:

```kotlin
@Preview(name = "Light", showBackground = true)
@Preview(name = "Dark", uiMode = UI_MODE_NIGHT_YES, showBackground = true)
@Preview(name = "Large Font", fontScale = 2.0f)
@Preview(name = "Tablet", device = "spec:width=1280dp,height=800dp,dpi=240")
@Composable
fun ScreenPreview() {
    AppTheme { HomeScreen() }
}
```

---

## 関連スキル

- アクセシビリティ全般: `skills/ui-accessibility/SKILL.md`
- UI レビュー: `skills/ui-review-checklist/SKILL.md`
