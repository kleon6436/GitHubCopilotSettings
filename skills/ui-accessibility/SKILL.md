---
name: ui-accessibility
description: 'クロスプラットフォーム UI アクセシビリティ原則。WCAG 2.2 AA、コントラスト比、フォーカス順序、スクリーンリーダー（VoiceOver / TalkBack / Narrator / NVDA / JAWS）、キーボード操作、Reduce Motion、ユーザー設定尊重を確認・適用したいときに使用。Use when: reviewing UI accessibility across any platform; applying WCAG; designing inclusive experiences.'
argument-hint: '確認したい項目（コントラスト / フォーカス / スクリーンリーダー など、省略可）'
---

# UI アクセシビリティガイドライン（クロスプラットフォーム共通）

## 概要

このスキルはプラットフォームを問わず UI アクセシビリティを確保するための共通原則を定義します。
個別プラットフォームの実装詳細は各 UI スキル（`apple-ui-guidelines` / `windows-ui-guidelines` / `web-ui-guidelines` / `android-ui-guidelines`）を参照してください。

参考:
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [Inclusive Design Principles](https://inclusivedesignprinciples.org/)

---

## 1. 4 原則（POUR）

WCAG の基本原則。すべての UI 判断はこの 4 つに立ち返って評価します。

| 原則 | 意味 |
|---|---|
| **Perceivable**（知覚可能） | 情報は利用者が知覚できる形で提供される（視覚・聴覚・触覚のいずれか） |
| **Operable**（操作可能） | UI はあらゆる入力手段（キーボード・マウス・タッチ・音声）で操作できる |
| **Understandable**（理解可能） | 情報と操作方法は利用者が理解できる |
| **Robust**（堅牢） | 多様な支援技術から確実に解釈される |

---

## 2. コントラスト

WCAG AA 基準:

| 対象 | 通常 | 大テキスト（18pt / 14pt bold） | 非テキスト（アイコン・境界線） |
|---|---|---|---|
| コントラスト比 | **4.5:1** | **3:1** | **3:1** |

AAA を目指す場合は **7:1 / 4.5:1**。

### 実装上のポイント

- プレースホルダーテキストも本文と同等のコントラストを満たす。薄すぎるプレースホルダーは不可。
- 無効（disabled）状態は **例外的に緩和可** だが、重要情報を disabled で隠さない。
- フォーカスインジケーターは背景に対し **3:1 以上**。
- ツール: [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) / Figma Contrast プラグイン / Xcode Accessibility Inspector / Chrome DevTools。

### 色だけに頼らない

情報は色以外の手がかり（アイコン・テキスト・パターン）でも伝える:

- エラー: 赤色 **+ アイコン + テキスト**
- リンク: 色 **+ 下線**
- 必須フィールド: 色 **+ 「*」 + `aria-required`**
- 選択状態: 背景色 **+ チェックマーク**

---

## 3. タッチターゲットサイズ

| プラットフォーム | 最小推奨 |
|---|---|
| Web（WCAG 2.5.5 AA） | **24×24 CSS px**（最小）/ **44×44 推奨** |
| iOS / iPadOS | **44×44 pt** |
| Android | **48×48 dp** |
| Windows | **40×40 px**（タッチ時推奨 48×48） |

- 隣接ターゲット間に最低 **8px** の余白を確保。
- 視覚的サイズが小さくても、ヒット領域（`hitSlop` / padding）を拡大すれば OK。

---

## 4. キーボード操作

### 必須要件

- **すべてのインタラクティブ要素がキーボードのみで到達・操作できる**（WCAG 2.1.1）。
- **キーボードトラップを作らない**（WCAG 2.1.2）。フォーカスはすべての要素から抜け出せる。
- **操作順序が論理的**（WCAG 2.4.3）。DOM / 視覚順序と一致させる。

### 標準ショートカット

| 操作 | キー |
|---|---|
| 次要素へ | `Tab` |
| 前要素へ | `Shift + Tab` |
| ボタン実行 | `Space` / `Enter` |
| リンク追跡 | `Enter` |
| チェックボックス切替 | `Space` |
| ドロップダウン展開 | `Space` / `Enter` / 矢印キー |
| モーダル閉じる | `Esc` |
| リスト移動 | 矢印キー |

### フォーカスインジケーター

- **フォーカスリングを消さない**（`:focus { outline: none }` 禁止）。
- Web では `:focus-visible` を活用し、マウスクリック時のみ非表示にできる。
- プラットフォームネイティブのフォーカスリングを尊重し、代替時は視認性（3:1 以上）を確保。

---

## 5. スクリーンリーダー対応

### 主要スクリーンリーダー

| プラットフォーム | スクリーンリーダー |
|---|---|
| iOS / iPadOS | **VoiceOver** |
| macOS | **VoiceOver** |
| Android | **TalkBack** |
| Windows | **Narrator** / **NVDA** / **JAWS** |
| Web | すべて対応可能（ブラウザ + SR） |

### 共通要件

- すべてのインタラクティブ要素に **アクセシブルな名前**（accessible name）を提供:
  - Web: `<button>削除</button>` / `aria-label="削除"`
  - iOS: `.accessibilityLabel("削除")`
  - Android: `contentDescription = "削除"`
  - Windows: `AutomationProperties.Name="削除"`
- アイコンのみのボタンには必ずラベル。
- 装飾的な画像・アイコンは支援技術から **隠す**:
  - Web: `aria-hidden="true"` / `alt=""`
  - iOS: `.accessibilityHidden(true)`
  - Android: `contentDescription = null`
- 状態変化を **ライブリージョン**で通知（読み上げ）:
  - Web: `aria-live="polite"` / `role="status"`
  - iOS: `AccessibilityNotification.Announcement`
  - Android: `Modifier.semantics { liveRegion = LiveRegionMode.Polite }`

### セマンティクスの正しい表現

- **役割（role）**: button / link / checkbox / radio など本来の役割を明示。
- **状態（state）**: 選択・展開・無効・押下中などを `aria-expanded` / `aria-selected` / `aria-pressed` 等で提供。
- **プロパティ（property）**: 関連要素（`aria-describedby` / `aria-labelledby`）、必須性（`aria-required`）。

---

## 6. テキスト・可読性

- **行の長さ**: 1 行あたり **45〜75 文字**（日本語は 30〜40 文字目安）。
- **行間**: 本文 **1.5 倍以上**（WCAG 1.4.12）。
- **段落間**: 行間の **2 倍**以上。
- **テキストのリサイズ**: 200% まで拡大しても内容・機能が失われない（WCAG 1.4.4）。
- **リフロー**: 幅 **320 CSS px** で横スクロールなしに表示できる（WCAG 1.4.10）。
- **テキスト画像を避ける**（ロゴ除く）。実テキストで表現。
- 行長・文字間隔・行間・段落間隔をユーザーが調整可能なテキストスペーシング（WCAG 1.4.12）で崩れない。

---

## 7. Reduce Motion（動きの低減）

ユーザーが「動きの低減」を有効にしている場合、アニメーションを最小化:

| プラットフォーム | API |
|---|---|
| Web | `@media (prefers-reduced-motion: reduce)` |
| iOS | `@Environment(\.accessibilityReduceMotion)` |
| Android | `Settings.Global.ANIMATOR_DURATION_SCALE` / `AccessibilityManager` |
| Windows | `UISettings.AnimationsEnabled` |

対応ルール:
- 視差効果・無限ループ・自動再生・点滅を停止、または簡略化。
- 必要なフィードバックはフェード等の穏やかな表現に置換。
- **自動再生コンテンツ**（動画・カルーセル）には一時停止手段を必ず提供。
- **3 回/秒を超える点滅を禁止**（光感受性発作対策、WCAG 2.3.1）。

---

## 8. Reduce Transparency / High Contrast

- **透明度の低減**: 半透明背景（Liquid Glass / Acrylic / frosted glass）はフォールバックを用意。
  - iOS/macOS: `@Environment(\.accessibilityReduceTransparency)`
  - Windows: System Backdrop が自動調整
  - Web: `@media (prefers-reduced-transparency: reduce)`
- **High Contrast / Increased Contrast**:
  - iOS/macOS: `@Environment(\.colorSchemeContrast)` / Assets の Increased Contrast バリアント
  - Windows: **High Contrast テーマ**で `SystemColor*Brush` を使用
  - Web: `@media (prefers-contrast: more)` / `forced-colors: active`

---

## 9. フォームアクセシビリティ

- すべての入力に **可視ラベル**を関連付ける（`<label for>` / `.accessibilityLabel` / `ContentDescription`）。
- **エラーメッセージ**:
  - エラー内容を明示的に（「入力エラー」ではなく「メールアドレスに @ が含まれていません」）
  - エラー位置を明示（`aria-describedby` / `accessibilityHint`）
  - 修正提案を提示
- **必須フィールド**を視覚 + 支援技術両方で示す（`aria-required="true"` / `required`）。
- **autocomplete / autofill** を適切に設定（WCAG 1.3.5）。
- 成功メッセージもライブリージョンで通知。

---

## 10. 画像・メディア

### 代替テキスト

- **情報を伝える画像**: 内容を簡潔に説明する `alt` / `accessibilityLabel`。
- **装飾画像**: `alt=""` / `accessibilityHidden`。
- **機能を持つ画像**（リンク画像・ボタン画像）: 機能を説明するテキスト。
- **複雑な画像**（図表・グラフ）: 短い alt + 長文説明（`aria-describedby` / 近隣テキスト）。

### 動画・音声

- **字幕（Captions）**: 音声情報を持つ動画に必須（WCAG 1.2.2）。
- **音声ガイド（Audio Description）**: 視覚情報の音声説明（WCAG 1.2.5）。
- **トランスクリプト**: 音声コンテンツの文字起こし。
- 自動再生はミュートで、停止手段を明示。

---

## 11. 言語・地域対応

- ドキュメント言語を明示（`<html lang="ja">` / `UIAccessibility.announceLanguage`）。
- 部分的な別言語には `lang` 属性（発音を SR に伝える）。
- 日付・数値・通貨はロケールに応じてフォーマット（`Intl` / `NumberFormatter`）。

---

## 12. タイムアウト

- タイムアウトがある場合、ユーザーに **延長・解除・再開始**のいずれかを提供（WCAG 2.2.1）。
- **20 時間以内**のセッションは維持可能に。
- 自動更新・リダイレクトはユーザーが停止できるようにする。

---

## 13. テスト・検証

### 自動テスト

| ツール | 対象 |
|---|---|
| **axe DevTools** | Web |
| **Lighthouse** | Web |
| **WAVE** | Web |
| **Accessibility Inspector**（Xcode） | iOS / macOS |
| **Accessibility Scanner** | Android |
| **Accessibility Insights** | Windows / Web |

### 手動テスト（必須）

1. **キーボードのみで全機能を操作**（マウス・トラックパッド不使用）
2. **スクリーンリーダーで全画面を走査**（VoiceOver / TalkBack / NVDA）
3. **フォントサイズ 200%** で崩れを確認
4. **Reduce Motion ON** でアニメーション動作確認
5. **High Contrast / Dark Mode** で視認性確認
6. **ズーム 400%**（Web の場合）で横スクロールなしに操作可能か確認

### ユーザーテスト

可能な限り実際の障害を持つユーザーによるテストを実施。自動テストでは約 **30%** の問題しか検出できない。

---

## 14. 認知的アクセシビリティ

- **シンプルな言葉**を使う（専門用語は必要時に定義）。
- **一貫したナビゲーション**（WCAG 3.2.3）。同じ機能は同じ場所に。
- **一貫した識別**（WCAG 3.2.4）。同じ機能は同じラベル・アイコン。
- **重要な操作には確認ステップ**（削除・決済）。取り消し可能にする（WCAG 3.3.4）。
- **プログレスインジケーター**で長時間処理を明示。
- **エラー防止** — 入力前のヒント・リアルタイム検証・送信前確認。

---

## 15. WCAG 2.2 新規要件（重要）

2023 年以降に追加された基準:

- **2.4.11 Focus Not Obscured (Minimum)** (AA) — フォーカスされた要素が他の UI（固定ヘッダ等）に隠れない。
- **2.5.7 Dragging Movements** (AA) — ドラッグで実行できる操作はシングルポインター（クリック・タップ）でも可能に。
- **2.5.8 Target Size (Minimum)** (AA) — 入力ターゲットは **24×24 CSS px 以上**（例外あり）。
- **3.2.6 Consistent Help** (A) — ヘルプ機能は一貫した位置に。
- **3.3.7 Redundant Entry** (A) — 同一プロセス内で一度入力した情報の再入力を要求しない。
- **3.3.8 Accessible Authentication (Minimum)** (AA) — 認知機能テスト（パズル・画像選択）に依存しない認証手段を提供。

---

## 16. レビュー観点チェックリスト

UI レビュー時に以下を必ず確認:

- [ ] すべてのインタラクティブ要素にアクセシブルな名前がある
- [ ] キーボードのみで全機能が操作できる
- [ ] フォーカスインジケーターが視認できる（3:1 以上）
- [ ] フォーカス順序が論理的
- [ ] Esc でモーダル・ポップオーバーが閉じる
- [ ] テキストコントラストが AA 以上（4.5:1 / 3:1）
- [ ] 色だけで情報を伝えていない
- [ ] タッチターゲットが推奨サイズ以上
- [ ] フォント拡大 200% で崩れない
- [ ] Dark Mode / High Contrast で視認可能
- [ ] Reduce Motion ON で過剰なアニメーションが抑制される
- [ ] 画像に適切な alt / 装飾画像は SR から隠されている
- [ ] 動画に字幕がある
- [ ] フォームのラベルと入力が関連付けられている
- [ ] エラーメッセージが具体的で、修正方法が分かる
- [ ] ライブリージョンで動的変化が SR に通知される
- [ ] ドラッグ操作に代替手段がある

---

## 関連スキル

- Apple UI: `skills/apple-ui-guidelines/SKILL.md`
- Windows UI: `skills/windows-ui-guidelines/SKILL.md`
- Web UI: `skills/web-ui-guidelines/SKILL.md`
- Android UI: `skills/android-ui-guidelines/SKILL.md`
- UI レビュー: `skills/ui-review-checklist/SKILL.md`
