---
name: performance-optimization
description: 'パフォーマンス最適化のガイドラインを参照・適用する。Core Web Vitals・メモリ管理・レンダリング最適化・ビルドサイズ削減・キャッシュ戦略・プラットフォーム別チューニングを確認・適用したいときに使用。Use when: optimizing app performance, reducing bundle size, improving Core Web Vitals, memory profiling, render optimization, caching strategies.'
argument-hint: '確認・適用したいパフォーマンス最適化の項目（省略可）'
---

# パフォーマンス最適化 ガイドライン

## 概要

このスキルはアプリケーションのパフォーマンス最適化の規約を定義します。
「計測してから最適化する」を基本原則とし、推測での最適化を避けてください。

---

## 1. 共通原則

- **計測が先、最適化が後** — プロファイラーで根拠を持ってから最適化する
- **ユーザー体験に直結する指標を優先** — FID・LCP・CLS・起動時間・スクロール滑らかさ
- **早すぎる最適化を避ける** — 可読性を犠牲にする前に必要性を確認する
- **パフォーマンスバジェットを設定** — 目標値を定め、CI で継続監視する

---

## 2. Web パフォーマンス

### Core Web Vitals 目標値

| 指標 | 良好 | 改善が必要 | 不良 |
|------|------|-----------|------|
| LCP（最大コンテンツの描画） | ≤ 2.5s | 2.5 〜 4.0s | > 4.0s |
| INP（次のペイントへの応答） | ≤ 200ms | 200 〜 500ms | > 500ms |
| CLS（累積レイアウトシフト） | ≤ 0.1 | 0.1 〜 0.25 | > 0.25 |

### 画像最適化

```tsx
// ✅ Good: next/image で自動最適化（Next.js）
import Image from 'next/image';
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />

// ✅ Good: 適切な format と lazy loading
<img src="image.webp" loading="lazy" decoding="async" width="400" height="300" />

// ❌ Bad: サイズ未指定（CLS の原因）
<img src="image.jpg" />
```

### バンドルサイズ削減

```ts
// ✅ Good: 動的インポートで Code Splitting
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <Skeleton />,
});

// ✅ Good: ツリーシェイキング対応の named import
import { debounce } from 'lodash-es';

// ❌ Bad: デフォルトインポートでバンドル全体を読み込む
import _ from 'lodash';
```

### キャッシュ戦略

| リソース | Cache-Control |
|---------|--------------|
| HTML | `no-cache`（常に最新を取得） |
| JS / CSS（ハッシュ付き） | `max-age=31536000, immutable` |
| 画像 | `max-age=86400`（1日） |
| API レスポンス | `max-age=60, stale-while-revalidate=300` |

### レンダリング最適化（React）

```tsx
// ✅ Good: useMemo で重い計算をメモ化
const sortedList = useMemo(() => expensiveSort(data), [data]);

// ✅ Good: useCallback で関数の再生成を防ぐ
const handleClick = useCallback(() => {
  onSelect(item.id);
}, [item.id, onSelect]);

// ✅ Good: React.memo でコンポーネントの再レンダーを防ぐ
const ListItem = memo(({ item, onSelect }: Props) => {
  return <div onClick={() => onSelect(item.id)}>{item.name}</div>;
});

// ❌ Bad: レンダー中に高コスト計算を直書き
const sorted = data.sort((a, b) => /* expensive comparison */);
```

---

## 3. iOS / macOS パフォーマンス

### Instruments を使った計測

- **Time Profiler** — CPU 使用率のホットスポットを特定する
- **Allocations** — メモリアロケーション・解放を追跡する
- **Leaks** — メモリリークを検出する
- **Core Data** — フェッチ回数・クエリの遅さを確認する

### SwiftUI 最適化

```swift
// ✅ Good: equatable で不要な再描画を防ぐ
struct ListRowView: View, Equatable {
    let item: Item
    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.item.id == rhs.item.id && lhs.item.updatedAt == rhs.item.updatedAt
    }
}

// ✅ Good: lazy で必要時だけ評価
LazyVStack {
    ForEach(items) { item in
        ItemRow(item: item)
    }
}

// ✅ Good: @StateObject は初回のみ生成
@StateObject private var viewModel = ItemListViewModel()

// ❌ Bad: @ObservedObject を外部から渡す（毎回再生成の可能性）
```

### 非同期・バックグラウンド処理

```swift
// ✅ Good: メインスレッドをブロックしない
Task {
    let data = await fetchData()  // バックグラウンドで処理
    await MainActor.run {
        self.items = data         // UI 更新はメインスレッドで
    }
}

// ❌ Bad: メインスレッドで同期的にネットワーク通信
let data = URLSession.shared.synchronousRequest(url)  // NG
```

---

## 4. Android パフォーマンス

### Compose 最適化

```kotlin
// ✅ Good: remember で重い計算をキャッシュ
val sortedItems = remember(items) {
    items.sortedBy { it.name }
}

// ✅ Good: derivedStateOf で不要な Recomposition を防ぐ
val isButtonEnabled by remember {
    derivedStateOf { selectedItems.isNotEmpty() }
}

// ✅ Good: LazyColumn に key を指定してアニメーション最適化
LazyColumn {
    items(items, key = { it.id }) { item ->
        ItemRow(item = item)
    }
}

// ❌ Bad: key なしの LazyColumn（全 item が再 Compose される）
```

### ANR 防止

```kotlin
// ✅ Good: IO 処理は Dispatchers.IO で実行
viewModelScope.launch(Dispatchers.IO) {
    val result = repository.fetchData()
    withContext(Dispatchers.Main) {
        _uiState.value = UiState.Success(result)
    }
}

// ❌ Bad: メインスレッドで重い処理
fun loadData() {
    _uiState.value = repository.fetchDataBlocking()  // ANR の原因
}
```

### プロファイリングツール

- **Android Studio Profiler** — CPU・メモリ・ネットワーク・エネルギー
- **Compose Inspector** — Recomposition の回数を確認
- **Systrace / Perfetto** — フレームタイムのボトルネックを特定

---

## 5. メモリ管理

### 共通パターン

```swift
// Swift: 循環参照を [weak self] で防ぐ
someViewModel.onComplete = { [weak self] result in
    self?.handleResult(result)
}
```

```kotlin
// Kotlin: lifecycleScope を使い、自動的にキャンセルされるようにする
viewLifecycleOwner.lifecycleScope.launch {
    viewModel.uiState.collect { state ->
        updateUI(state)
    }
}
```

```ts
// React: useEffect のクリーンアップでサブスクリプションを解除する
useEffect(() => {
  const subscription = dataService.subscribe(setData);
  return () => subscription.unsubscribe();
}, []);
```

---

## 6. パフォーマンスバジェット

プロジェクト開始時に以下の目標値を設定し、CI で監視すること。

| 指標 | 目標値（例） |
|------|------------|
| JS バンドルサイズ（初期ロード） | 200KB 以下（gzip） |
| TTI（Time to Interactive） | 3.0 秒以下 |
| LCP | 2.5 秒以下 |
| アプリ起動時間（iOS cold） | 400ms 以下 |
| スクロールフレームレート | 60fps（120fps デバイス対応で 120fps） |
| メモリ使用量（通常利用時） | {プロジェクト固有の目標値} |

---

## 7. チェックリスト

- [ ] プロファイラーで計測済みのボトルネックに対して最適化している
- [ ] Core Web Vitals の目標値が設定され、CI で監視されている
- [ ] 画像に適切な format・サイズ・lazy loading が設定されている
- [ ] バンドルサイズに Code Splitting が適用されている
- [ ] 重い計算が適切にキャッシュ（memoize）されている
- [ ] メインスレッドで重い同期処理を実行していない
- [ ] メモリリークの可能性（循環参照・未解除リスナー）を確認した
- [ ] リスト描画に仮想化・遅延ロードが適用されている
