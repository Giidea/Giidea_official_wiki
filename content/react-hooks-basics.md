---
title: "React Hooks の基礎"
slug: "react-hooks-basics"
tags: ["React", "Hooks", "JavaScript", "Frontend"]
author_id: "00000000-0000-0000-0000-000000000001"
created_at: "2024-01-15T09:00:00Z"
relations:
  - type: "parent"
    slug: "react"
  - type: "prerequisite"
    slug: "react-components"
  - type: "child"
    slug: "usestate-hook"
  - type: "child"
    slug: "useeffect-hook"
  - type: "comparison"
    slug: "vue-composition-api"
---

# React Hooks の基礎

## 概要

React Hooksは、関数コンポーネントでstate管理とライフサイクルメソッドを使用できるようにするReact 16.8で導入された機能です。

## 主要なHooks

### useState

状態管理のための基本的なHookです。

```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>現在のカウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        インクリメント
      </button>
    </div>
  );
}
```

### useEffect

副作用を処理するためのHookです。

```javascript
import React, { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(seconds => seconds + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <div>経過時間: {seconds}秒</div>;
}
```

## ベストプラクティス

1. **依存配列を正しく指定する**
   - useEffectの第二引数には、副作用で使用するすべての値を含める

2. **カスタムHooksを活用する**
   - 再利用可能なロジックはカスタムHooksに分離する

3. **パフォーマンスを考慮する**
   - useMemoやuseCallbackを適切に使用してレンダリングを最適化する

## 関連記事

- [State Management with Redux](./state-management-redux.md)
- [React Performance Optimization](./react-performance.md)