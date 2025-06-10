---
title: "ブラウザ Event Loop の仕組み"
slug: "browser-event-loop"
tags: ["JavaScript", "Browser", "Event Loop", "Web API", "Frontend"]
author_id: "00000000-0000-0000-0000-000000000001"
created_at: "2024-01-16T10:00:00Z"
relations:
  - type: "comparison"
    slug: "nodejs-event-loop"
  - type: "parent"
    slug: "javascript-fundamentals"
---

# ブラウザ Event Loop の仕組み

## 概要

ブラウザのEvent Loopは、JavaScriptの非同期処理を管理する中核的なメカニズムです。シングルスレッドのJavaScriptで複雑なWebアプリケーションを実現するための仕組みを理解しましょう。

## Event Loopの基本構造

```
┌─────────────────────────────┐
│         Call Stack          │  <- 現在実行中の関数
└─────────────┬───────────────┘
              │
┌─────────────▼───────────────┐
│        Event Loop           │
└─────────────┬───────────────┘
              │
┌─────────────▼───────────────┐
│       Task Queue            │  <- Macrotasks
│   - setTimeout callbacks    │
│   - setInterval callbacks   │
│   - DOM events             │
│   - fetch() responses      │
└─────────────┬───────────────┘
              │
┌─────────────▼───────────────┐
│     Microtask Queue         │  <- Microtasks
│   - Promise.then()          │
│   - queueMicrotask()        │
│   - MutationObserver        │
└─────────────────────────────┘
```

## 実行順序の基本原則

1. **Call Stack** が空になる
2. **Microtask Queue** のすべてのタスクを実行
3. **Task Queue** から1つのタスクを実行
4. 再び Microtask Queue をチェック
5. 必要に応じて画面を再描画
6. 1に戻る

## Macrotasks vs Microtasks

### Macrotasks（Task Queue）
```javascript
// setTimeout/setInterval
setTimeout(() => console.log('timeout'), 0);

// DOM Events
button.addEventListener('click', () => console.log('click'));

// I/O操作（fetch等）
fetch('/api').then(() => console.log('fetch'));
```

### Microtasks（Microtask Queue）
```javascript
// Promise.then()
Promise.resolve().then(() => console.log('promise'));

// queueMicrotask()
queueMicrotask(() => console.log('microtask'));

// MutationObserver
const observer = new MutationObserver(() => console.log('mutation'));
```

## 実行順序の例

### 基本例
```javascript
console.log('=== START ===');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise 1'));
Promise.resolve().then(() => console.log('promise 2'));

queueMicrotask(() => console.log('microtask'));

console.log('=== END ===');

// Output:
// === START ===
// === END ===
// promise 1
// promise 2
// microtask
// timeout
```

### 複雑な例
```javascript
console.log('start');

setTimeout(() => {
  console.log('timeout 1');
  Promise.resolve().then(() => console.log('promise in timeout'));
}, 0);

Promise.resolve().then(() => {
  console.log('promise 1');
  setTimeout(() => console.log('timeout in promise'), 0);
});

Promise.resolve().then(() => console.log('promise 2'));

setTimeout(() => console.log('timeout 2'), 0);

console.log('end');

// Output:
// start
// end
// promise 1
// promise 2
// timeout 1
// promise in timeout
// timeout in promise
// timeout 2
```

## Web APIs との連携

### Fetch API
```javascript
console.log('1');

fetch('/api/data')
  .then(response => {
    console.log('2: fetch response');
    return response.json();
  })
  .then(data => {
    console.log('3: json parsed');
  });

Promise.resolve().then(() => console.log('4: immediate promise'));

console.log('5');

// Output:
// 1
// 5
// 4: immediate promise
// 2: fetch response
// 3: json parsed
```

### DOM Events
```javascript
const button = document.getElementById('btn');

button.addEventListener('click', () => {
  console.log('click handler 1');
  
  Promise.resolve().then(() => console.log('promise in click 1'));
  
  setTimeout(() => console.log('timeout in click'), 0);
});

button.addEventListener('click', () => {
  console.log('click handler 2');
  
  Promise.resolve().then(() => console.log('promise in click 2'));
});

// ボタンクリック時の出力:
// click handler 1
// click handler 2
// promise in click 1
// promise in click 2
// timeout in click
```

## 画面描画（Rendering）との関係

```javascript
const element = document.getElementById('box');

console.log('start');

// DOMを変更
element.style.backgroundColor = 'red';

// 同期的にスタイルを読み取ると強制リフロー
console.log('height:', element.offsetHeight);

Promise.resolve().then(() => {
  console.log('microtask executed');
  element.style.backgroundColor = 'blue';
});

setTimeout(() => {
  console.log('macrotask executed');
  element.style.backgroundColor = 'green';
}, 0);

console.log('end');

// 画面は最終的に緑色になる
// 中間状態（赤、青）は通常見えない
```

## パフォーマンス最適化

### 1. Long Tasks の分割
```javascript
// ❌ Bad: Long Task (Event Loopをブロック)
function heavyTask() {
  for (let i = 0; i < 1000000; i++) {
    // 重い処理
    complexCalculation();
  }
}

// ✅ Good: 分割して実行
function heavyTaskOptimized() {
  let i = 0;
  const batchSize = 10000;
  
  function processBatch() {
    const end = Math.min(i + batchSize, 1000000);
    
    while (i < end) {
      complexCalculation();
      i++;
    }
    
    if (i < 1000000) {
      setTimeout(processBatch, 0); // 次のバッチを予約
    }
  }
  
  processBatch();
}
```

### 2. requestAnimationFrame の活用
```javascript
// アニメーション処理
function animate() {
  // DOM更新処理
  updateElements();
  
  // 次のフレームで続行
  requestAnimationFrame(animate);
}

// 描画直前に実行される
requestAnimationFrame(animate);
```

### 3. Intersection Observer の活用
```javascript
// スクロール監視の最適化
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // 要素が表示されたときの処理
      loadContent(entry.target);
    }
  });
});

// 対象要素を監視
document.querySelectorAll('.lazy-load').forEach(el => {
  observer.observe(el);
});
```

## デバッグと分析

### Performance Timeline API
```javascript
// パフォーマンス測定
performance.mark('start-task');

// 重い処理
heavyTask();

performance.mark('end-task');
performance.measure('task-duration', 'start-task', 'end-task');

// 結果取得
const measures = performance.getEntriesByType('measure');
console.log(`Task took: ${measures[0].duration}ms`);
```

### Event Loop Monitoring
```javascript
// Event Loopのブロック検知
let lastTime = performance.now();

function checkEventLoop() {
  const currentTime = performance.now();
  const delta = currentTime - lastTime;
  
  if (delta > 50) { // 50ms以上の遅延
    console.warn(`Event Loop blocked for ${delta.toFixed(2)}ms`);
  }
  
  lastTime = currentTime;
  setTimeout(checkEventLoop, 0);
}

checkEventLoop();
```

## Node.js Event Loop との違い

| 特徴 | ブラウザ | Node.js |
|------|----------|---------|
| 環境 | ブラウザ内 | サーバーサイド |
| Timer精度 | 4ms最小間隔 | 1ms精度 |
| I/O | Web APIs | libuv |
| フェーズ | 単純な優先度 | 6つのフェーズ |
| `setImmediate` | なし | あり |
| `process.nextTick` | なし | あり（最高優先度） |

```javascript
// ブラウザでの実行順序
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));

// 出力: promise → timeout

// Node.jsでの実行順序
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));

// 出力: nextTick → promise → timeout → immediate
```

## まとめ

ブラウザのEvent Loopを理解することで：
- 非同期処理の実行順序が予測できる
- パフォーマンス問題を特定・解決できる
- ユーザー体験を改善できる
- 効率的なコードが書ける

## 関連記事

- [Node.js Event Loop との比較](./nodejs-event-loop.md)
- [Web Performance Optimization](./web-performance.md)
- [JavaScript Async Patterns](./async-patterns.md)