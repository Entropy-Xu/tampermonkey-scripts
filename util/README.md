# util - 通用工具库

供油猴脚本通过 `@require` 引入的通用工具集。

## 目录

| 工具                                                             | 描述           |
| ---------------------------------------------------------------- | -------------- |
| [background-keep-alive.user.js](./background-keep-alive.user.js) | 后台保活工具集 |

---

## background-keep-alive.user.js

解决浏览器后台标签页节流问题的通用工具集，包含三大模块。

### 引入方式

```javascript
// ==UserScript==
// @require https://update.greasyfork.org/scripts/559089/1714656/background-keep-alive.js
// ==/UserScript==
```

---

### 模块 1: BackgroundTimer

基于 Web Worker 的保活定时器，绑定后台环境下不被节流。

```javascript
const timer = new BackgroundTimer(() => {
  console.log("心跳:", new Date().toTimeString());
}, 1000);

timer.start(); // 启动
timer.stop(); // 停止
timer.setInterval(2000); // 动态调整间隔
timer.isRunning(); // 获取状态
timer.destroy(); // 销毁实例
```

---

### 模块 2: AudioKeepAlive

使用静音音频对抗 Chrome 5 分钟强力休眠。

> ⚠️ 需要用户交互后才能启动（浏览器自动播放策略限制）

```javascript
const audio = new AudioKeepAlive();

// 需要在用户交互后启动
document.addEventListener("click", () => audio.start(), { once: true });

audio.stop(); // 停止
audio.isActive(); // 获取状态
audio.destroy(); // 销毁实例
```

---

### 模块 3: NetworkMonitor

Hook Fetch **和 XHR** 监控任务完成状态，使用**防抖 + 活跃计数器**算法。

```javascript
const monitor = new NetworkMonitor({
  // 监控的 URL 模式（包含匹配）
  // 注意：避免使用通用 RPC 方法如 batchexecute，会产生误判
  urlPatterns: ["BardFrontendService", "StreamGenerate"],

  // 静默判定时间（毫秒）
  silenceThreshold: 3000,

  // 任务完成回调
  onComplete: (ctx) => {
    console.log("任务完成", ctx);
    // ctx = { activeCount, lastUrl, timestamp }
  },

  // 任务开始回调（可选）
  onStart: (ctx) => {
    console.log("任务开始", ctx);
  },

  // DOM 二次验证（可选，自定义）
  domValidation: (ctx) => {
    // 返回 true 表示验证通过，触发 onComplete
    // 返回 false 表示还需等待
    return !document.querySelector(".stop-button");
  },
});

monitor.start(); // 开始监控
monitor.stop(); // 停止监控
monitor.isIdle(); // 是否空闲
monitor.getActiveCount(); // 活跃请求数
monitor.destroy(); // 销毁实例
```

**算法说明**：

1. 记录当前活跃请求数
2. 每次请求开始/结束都重置静默计时器
3. 活跃数为 0 + 静默期结束 + DOM 验证通过 → 触发 onComplete
