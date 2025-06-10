---
title: "State Management パターン"
slug: "state-management"
tags: ["React", "Redux", "Zustand", "State", "Architecture"]
author_id: "00000000-0000-0000-0000-000000000001"
created_at: "2024-01-15T10:00:00Z"
relations:
  - type: "parent"
    slug: "react"
  - type: "comparison"
    slug: "redux"
  - type: "comparison"
    slug: "zustand"
  - type: "comparison"
    slug: "mobx"
---

# State Management パターン

## 概要

Reactアプリケーションにおける状態管理は、アプリケーションの複雑さとともに重要性が増します。適切な状態管理パターンを選択することで、保守性とスケーラビリティを向上させることができます。

## 状態管理の選択肢

### 1. React内蔵の状態管理

#### useState + useContext
```javascript
import React, { createContext, useContext, useState } from 'react';

const AppContext = createContext();

export function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}

export function useAppState() {
  return useContext(AppContext);
}
```

#### useReducer
```javascript
import React, { useReducer } from 'react';

const initialState = { count: 0, user: null };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    case 'setUser':
      return { ...state, user: action.payload };
    default:
      return state;
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>
        Increment
      </button>
    </div>
  );
}
```

### 2. Redux Toolkit

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;

export const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});
```

### 3. Zustand

```javascript
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));

function Counter() {
  const { count, increment, decrement, reset } = useStore();
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

## 選択指針

| アプリケーションサイズ | 推奨ソリューション |
|---------------------|-------------------|
| 小規模（~5画面） | useState + useContext |
| 中規模（5-20画面） | useReducer または Zustand |
| 大規模（20画面以上） | Redux Toolkit |

## パフォーマンス考慮事項

1. **不要な再レンダリングを避ける**
   - React.memoでコンポーネントをメモ化
   - useMemoでexpensiveな計算をキャッシュ

2. **状態の正規化**
   - ネストしたオブジェクトは避ける
   - IDベースの参照を使用

3. **非同期状態の管理**
   - TanStack QueryやSWRを検討
   - loading/error/dataの状態を適切に管理

## 関連記事

- [React Hooks の基礎](./react-hooks-basics.md)
- [Redux Best Practices](./redux-best-practices.md)