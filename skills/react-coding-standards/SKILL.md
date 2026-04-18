---
name: react-coding-standards
description: 'Reactのコーディング規約を参照・適用する。React コーディング規約、コンポーネント設計、命名規則、Props型定義、State管理、Hooks、パフォーマンス最適化、スタイリング方針、テストを確認・適用したいときに使用。Use when: applying React style guide, reviewing React code conventions, component design, props typing, hooks usage, memo/useMemo/useCallback, testing components.'
argument-hint: '確認・適用したいコーディング規約の項目（省略可）'
---

# React コーディング規約

## 概要

このスキルは React コードのコーディング規約を定義します。
TypeScript との併用を前提としています。
コードレビュー・新規実装の際はこの規約に従ってください。

---

## 1. コンポーネント設計原則

- **単一責任**：1コンポーネント1責務。肥大化したら分割する。
- **関数コンポーネント**のみ使用する。クラスコンポーネントは使わない。
- UI ロジックとビジネスロジックを分離する（カスタム Hooks に切り出す）。
- コンポーネントは純粋な関数として扱い、副作用は `useEffect` に限定する。

```tsx
// ✅ Good（UI とロジックを分離）
function useUserProfile(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  useEffect(() => { fetchUser(userId).then(setUser); }, [userId]);
  return user;
}

function UserProfileCard({ userId }: { userId: string }) {
  const user = useUserProfile(userId);
  if (!user) return <Skeleton />;
  return <Card title={user.name} />;
}

// ❌ Bad（UI コンポーネント内で直接 fetch）
function UserProfileCard({ userId }: { userId: string }) {
  useEffect(() => { fetch(`/api/users/${userId}`).then(/* ... */); }, [userId]);
  // ...
}
```

---

## 2. 命名規則

| 種別 | 規則 | 例 |
|------|------|----|
| コンポーネント | `UpperCamelCase` | `UserProfileCard` |
| コンポーネントファイル | `UpperCamelCase.tsx` | `UserProfileCard.tsx` |
| カスタム Hooks | `use` プレフィックス + `lowerCamelCase` | `useUserProfile` |
| Props 型 | コンポーネント名 + `Props` | `UserProfileCardProps` |
| イベントハンドラー | `handle` + イベント名 | `handleSubmit`, `handleClick` |
| Props のコールバック | `on` + イベント名 | `onSubmit`, `onClick` |

```tsx
// ✅ Good
interface UserProfileCardProps {
  userId: string;
  onEdit: (user: User) => void;
}

function UserProfileCard({ userId, onEdit }: UserProfileCardProps) {
  function handleEditClick() { onEdit(user); }
  // ...
}

// ❌ Bad
function userProfileCard({ id, editCallback }: any) { }
```

---

## 3. Props の型定義

- すべての Props は TypeScript の `interface` で型定義する。
- オプショナルな Props にはデフォルト値を設定する。
- `children` は `React.ReactNode` を使用する。
- コールバック Props の型は具体的に定義する（`Function` は使わない）。

```tsx
// ✅ Good
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
  children?: React.ReactNode;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

function Button({ label, variant = 'primary', disabled = false, onClick }: ButtonProps) {
  return <button className={variant} disabled={disabled} onClick={onClick}>{label}</button>;
}

// ❌ Bad
function Button({ label, variant, onClick }: any) { }
function Button(props: { onClick: Function }) { }
```

---

## 4. State 管理

- ローカル State は `useState`、複雑な State 遷移は `useReducer` を使用する。
- State の型は明示的に指定する。
- State をできる限り最小化し、派生値は `useMemo` で計算する。
- グローバル State は {例: Zustand / Jotai / Redux Toolkit} を使用する（プロジェクトに応じて変更）。

```tsx
// ✅ Good（単純な State）
const [isOpen, setIsOpen] = useState(false);
const [user, setUser] = useState<User | null>(null);

// ✅ Good（複雑な State は useReducer）
type Action =
  | { type: 'SET_LOADING' }
  | { type: 'SET_DATA'; payload: User[] }
  | { type: 'SET_ERROR'; payload: string };

function reducer(state: State, action: Action): State { /* ... */ }

// ❌ Bad
const [state, setState] = useState<any>({});  // any 型
```

---

## 5. Hooks の使い方

- Hooks はコンポーネントのトップレベルでのみ呼び出す（条件分岐・ループ内は禁止）。
- `useEffect` の依存配列は省略しない。不要な依存は `useCallback` / `useMemo` で安定化する。
- カスタム Hooks を積極的に作成して再利用する。

```tsx
// ✅ Good
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debouncedValue;
}

// ❌ Bad（条件分岐内で Hooks を呼び出す）
function Component({ isLoggedIn }: { isLoggedIn: boolean }) {
  if (isLoggedIn) {
    const [data, setData] = useState(null);  // ルール違反
  }
}
```

---

## 6. パフォーマンス最適化

- 最初から最適化しない。プロファイリングで問題が確認されてから適用する。
- `React.memo` で不要な再レンダリングを防ぐ。
- `useCallback` は Props として渡すコールバックに使用する。
- `useMemo` は計算コストが高い派生値に使用する。

```tsx
// ✅ Good（プロファイリング後に適用）
const MemoizedItem = React.memo(function Item({ item, onRemove }: ItemProps) {
  return <li>{item.name} <button onClick={() => onRemove(item.id)}>削除</button></li>;
});

function List({ items }: { items: Item[] }) {
  const handleRemove = useCallback((id: string) => {
    setItems((prev) => prev.filter((item) => item.id !== id));
  }, []);

  const sortedItems = useMemo(
    () => [...items].sort((a, b) => a.name.localeCompare(b.name)),
    [items],
  );

  return <ul>{sortedItems.map((item) => <MemoizedItem key={item.id} item={item} onRemove={handleRemove} />)}</ul>;
}
```

---

## 7. スタイリング方針

<!-- プロジェクトに応じて方針を選択・変更してください -->

- スタイリング手法: {例: Tailwind CSS / CSS Modules / styled-components}
- コンポーネントのスタイルはコンポーネントファイルと同じディレクトリに配置する。
- インラインスタイル（`style` Props）は動的な値のみに限定する。

```tsx
// ✅ Good（Tailwind CSS の例）
function Badge({ label, variant }: { label: string; variant: 'success' | 'error' }) {
  const variantClass = variant === 'success' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800';
  return <span className={`inline-flex items-center px-2 py-1 rounded ${variantClass}`}>{label}</span>;
}

// ❌ Bad（インラインスタイルを多用）
function Badge({ label }: { label: string }) {
  return <span style={{ display: 'inline-flex', padding: '4px 8px', borderRadius: '4px' }}>{label}</span>;
}
```

---

## 8. テスト

- コンポーネントのテストは {例: Vitest / Jest} + React Testing Library を使用する。
- 実装詳細ではなく、**ユーザーの操作・見た目**をテストする。
- スナップショットテストは変化の検知のみに使い、多用しない。

```tsx
// ✅ Good（ユーザー操作をテスト）
test('クリックでカウントが増加する', async () => {
  render(<Counter initialCount={0} />);
  const button = screen.getByRole('button', { name: '増やす' });
  await userEvent.click(button);
  expect(screen.getByText('1')).toBeInTheDocument();
});

// ❌ Bad（実装詳細をテスト）
test('setState が呼ばれる', () => {
  const instance = createComponent();
  instance.setState({ count: 1 });
  expect(instance.state.count).toBe(1);
});
```

---

## 9. コメント規約

- コンポーネントの Props に JSDoc / TSDoc コメントを付ける。
- 複雑なロジックには必ずコメントを付ける。
- TODO / FIXME は `// TODO: 説明` の形式で記述し、チケット番号を添える。

```tsx
/**
 * ユーザープロフィールカードを表示するコンポーネント。
 * @param userId - プロフィールを表示するユーザーの ID
 * @param onEdit - 編集ボタンクリック時のコールバック
 */
function UserProfileCard({ userId, onEdit }: UserProfileCardProps) {
  // ...
}

// TODO: #789 スケルトン UI に切り替える
```

---

## 10. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}
