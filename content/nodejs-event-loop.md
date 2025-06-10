---
title: "Node.js Event Loop の仕組み"
slug: "nodejs-event-loop"
tags: ["Node.js", "JavaScript", "Backend", "Async", "Performance"]
author_id: "00000000-0000-0000-0000-000000000001"
created_at: "2024-01-15T11:00:00Z"
relations:
  - type: "parent"
    slug: "nodejs"
  - type: "child"
    slug: "nodejs-timers"
  - type: "child"
    slug: "nodejs-streams"
  - type: "comparison"
    slug: "browser-event-loop"
---

# Node.js Event Loop の仕組み

## 概要

Node.jsのEvent Loopは、非同期処理を効率的に管理するための中核的なメカニズムです。シングルスレッドでありながら高いパフォーマンスを実現する秘密がここにあります。

## Event Loopの基本構造

```
┌───────────────────────────┐
┌─>│           timers          │  <- setTimeout(), setInterval()
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  <- I/O callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  <- internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  <- fetching new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  <- setImmediate() callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  <- socket.on('close', ...)
   └───────────────────────────┘
```

## 各フェーズの詳細

### 1. Timers Phase
```javascript
console.log('start');

setTimeout(() => {
  console.log('timeout 1');
}, 0);

setTimeout(() => {
  console.log('timeout 2');
}, 0);

console.log('end');

// Output:
// start
// end
// timeout 1
// timeout 2
```

### 2. Poll Phase
```javascript
const fs = require('fs');

console.log('start');

fs.readFile('file.txt', (err, data) => {
  console.log('file read complete');
});

console.log('end');

// Output:
// start
// end
// file read complete
```

### 3. Check Phase
```javascript
console.log('start');

setImmediate(() => {
  console.log('setImmediate');
});

setTimeout(() => {
  console.log('setTimeout');
}, 0);

console.log('end');

// Output:
// start
// end
// setTimeout
// setImmediate
```

## Microtasks vs Macrotasks

### Microtasks（優先度高）
- `Promise.then()`
- `queueMicrotask()`
- `process.nextTick()` (Node.js特有)

### Macrotasks（優先度低）
- `setTimeout()`
- `setInterval()`
- `setImmediate()`
- I/O操作

```javascript
console.log('=== START ===');

setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));

Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));

console.log('=== END ===');

// Output:
// === START ===
// === END ===
// nextTick
// promise
// timeout
// immediate
```

## パフォーマンス最適化

### 1. ブロッキング処理を避ける
```javascript
// ❌ Bad: ブロッキング
const result = fs.readFileSync('large-file.txt');

// ✅ Good: 非ブロッキング
fs.readFile('large-file.txt', (err, data) => {
  // 処理
});

// ✅ Better: Promise based
const data = await fs.promises.readFile('large-file.txt');
```

### 2. CPU集約的タスクの分割
```javascript
// ❌ Bad: Event Loopをブロック
function heavyCalculation() {
  let result = 0;
  for (let i = 0; i < 10000000; i++) {
    result += Math.random();
  }
  return result;
}

// ✅ Good: 分割して処理
function heavyCalculationAsync(callback) {
  let result = 0;
  let i = 0;
  const batchSize = 100000;
  
  function processBatch() {
    const end = Math.min(i + batchSize, 10000000);
    while (i < end) {
      result += Math.random();
      i++;
    }
    
    if (i < 10000000) {
      setImmediate(processBatch);
    } else {
      callback(result);
    }
  }
  
  processBatch();
}
```

### 3. Worker Threadsの活用
```javascript
// main.js
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  worker.postMessage({ numbers: [1, 2, 3, 4, 5] });
  
  worker.on('message', (result) => {
    console.log('Result:', result);
  });
} else {
  parentPort.on('message', ({ numbers }) => {
    const sum = numbers.reduce((a, b) => a + b, 0);
    parentPort.postMessage(sum);
  });
}
```

## デバッグとモニタリング

### Event Loopラグの測定
```javascript
const { performance } = require('perf_hooks');

function measureEventLoopLag() {
  const start = performance.now();
  
  setImmediate(() => {
    const lag = performance.now() - start;
    console.log(`Event Loop Lag: ${lag.toFixed(2)}ms`);
  });
}

setInterval(measureEventLoopLag, 1000);
```

### プロファイリング
```bash
# CPU プロファイリング
node --prof app.js

# Event Loopラグ監視
node --trace-event-categories v8 app.js
```

## 関連記事

- [Asynchronous JavaScript Patterns](./async-patterns.md)
- [Node.js Performance Optimization](./nodejs-performance.md)