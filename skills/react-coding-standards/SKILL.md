---
name: react-coding-standards
description: 'Reference and apply React coding standards. Use when: applying React style guide, reviewing React code conventions, component design, props typing, hooks usage, memo/useMemo/useCallback, testing components.'
argument-hint: 'Coding standard item to check or apply (optional)'
---

# React Coding Standards

## Overview

This skill defines coding standards for React code.
It assumes usage together with TypeScript.
Follow these conventions during code review and new implementation.

---

## 1. Component Design Principles

- **Single Responsibility**: One component, one concern. Split when a component grows too large.
- Use **function components** only. Do not use class components.
- Separate UI logic from business logic (extract into custom Hooks).
- Treat components as pure functions and limit side effects to `useEffect`.

```tsx
// ✅ Good (separate UI and logic)
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

// ❌ Bad (fetch directly inside UI component)
function UserProfileCard({ userId }: { userId: string }) {
  useEffect(() => { fetch(`/api/users/${userId}`).then(/* ... */); }, [userId]);
  // ...
}
```

---

## 2. Naming Conventions

| Category | Rule | Example |
|----------|------|---------|
| Component | `UpperCamelCase` | `UserProfileCard` |
| Component file | `UpperCamelCase.tsx` | `UserProfileCard.tsx` |
| Custom Hooks | `use` prefix + `lowerCamelCase` | `useUserProfile` |
| Props type | Component name + `Props` | `UserProfileCardProps` |
| Event handler | `handle` + event name | `handleSubmit`, `handleClick` |
| Props callback | `on` + event name | `onSubmit`, `onClick` |

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

## 3. Props Type Definitions

- Define all Props with TypeScript `interface`.
- Set default values for optional Props.
- Use `React.ReactNode` for `children`.
- Define callback Props types specifically (do not use `Function`).

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

## 4. State Management

- Use `useState` for local state and `useReducer` for complex state transitions.
- Specify State types explicitly.
- Minimize State as much as possible and compute derived values with `useMemo`.
- Use {e.g. Zustand / Jotai / Redux Toolkit} for global state (adjust per project).

```tsx
// ✅ Good (simple State)
const [isOpen, setIsOpen] = useState(false);
const [user, setUser] = useState<User | null>(null);

// ✅ Good (complex State — use useReducer)
type Action =
  | { type: 'SET_LOADING' }
  | { type: 'SET_DATA'; payload: User[] }
  | { type: 'SET_ERROR'; payload: string };

function reducer(state: State, action: Action): State { /* ... */ }

// ❌ Bad
const [state, setState] = useState<any>({});  // any type
```

---

## 5. Using Hooks

- Call Hooks only at the top level of components (not inside conditionals or loops).
- Never omit the `useEffect` dependency array. Stabilize unnecessary dependencies with `useCallback` / `useMemo`.
- Actively create and reuse custom Hooks.

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

// ❌ Bad (calling Hooks inside a conditional)
function Component({ isLoggedIn }: { isLoggedIn: boolean }) {
  if (isLoggedIn) {
    const [data, setData] = useState(null);  // Rules violation
  }
}
```

---

## 6. Performance Optimization

- Do not optimize prematurely. Apply only after profiling confirms a problem.
- Use `React.memo` to prevent unnecessary re-renders.
- Use `useCallback` for callbacks passed as Props.
- Use `useMemo` for derived values with high computation cost.

```tsx
// ✅ Good (apply after profiling)
const MemoizedItem = React.memo(function Item({ item, onRemove }: ItemProps) {
  return <li>{item.name} <button onClick={() => onRemove(item.id)}>Remove</button></li>;
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

## 7. Styling Guidelines

<!-- Select and adjust the approach according to the project -->

- Styling method: {e.g. Tailwind CSS / CSS Modules / styled-components}
- Place component styles in the same directory as the component file.
- Limit inline styles (`style` Props) to dynamic values only.

```tsx
// ✅ Good (Tailwind CSS example)
function Badge({ label, variant }: { label: string; variant: 'success' | 'error' }) {
  const variantClass = variant === 'success' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800';
  return <span className={`inline-flex items-center px-2 py-1 rounded ${variantClass}`}>{label}</span>;
}

// ❌ Bad (overusing inline styles)
function Badge({ label }: { label: string }) {
  return <span style={{ display: 'inline-flex', padding: '4px 8px', borderRadius: '4px' }}>{label}</span>;
}
```

---

## 8. Testing

- Use {e.g. Vitest / Jest} + React Testing Library for component tests.
- Test **user interactions and appearance**, not implementation details.
- Use snapshot tests only for change detection; do not overuse them.

```tsx
// ✅ Good (testing user interactions)
test('count increases on click', async () => {
  render(<Counter initialCount={0} />);
  const button = screen.getByRole('button', { name: 'Increment' });
  await userEvent.click(button);
  expect(screen.getByText('1')).toBeInTheDocument();
});

// ❌ Bad (testing implementation details)
test('setState is called', () => {
  const instance = createComponent();
  instance.setState({ count: 1 });
  expect(instance.state.count).toBe(1);
});
```

---

## 9. Comment Conventions

- Add JSDoc / TSDoc comments to component Props.
- Always add comments to complex logic.
- Write TODO / FIXME in the format `// TODO: description` and include a ticket number.

```tsx
/**
 * Component that displays a user profile card.
 * @param userId - ID of the user whose profile is displayed
 * @param onEdit - Callback fired when the edit button is clicked
 */
function UserProfileCard({ userId, onEdit }: UserProfileCardProps) {
  // ...
}

// TODO: #789 Switch to skeleton UI
```

---

## 10. Project-Specific Rules

<!-- Add rules specific to your project here -->

- {List project-specific rules here}
