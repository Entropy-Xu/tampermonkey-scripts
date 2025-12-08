# Gemini 提示词管理器 - 变更日志

## 版本 1.2.0 (2024-12-08)

### 新增功能：快捷跳转按钮

**功能描述**：添加快捷跳转到页面顶部和底部的功能。

#### 实现细节

**面板展开时**：

- 面板底部显示两个按钮：`⬆ 顶部` 和 `⬇ 底部`
- 使用圆角矩形样式，填充渐变色

**面板收起时**：

- 右下角显示垂直排列的按钮组
- 包含三个按钮：⬆（顶部）、📝（打开面板）、⬇（底部）

**技术要点**：

- Gemini Business 使用 Shadow DOM，滚动容器在 Shadow DOM 内部
- 新增 `findScrollContainerInShadowDOM()` 方法递归查找滚动容器
- 滚动容器类名：`.chat-mode-scroller.tile-content`

---

## 版本 1.1.0 (2024-12-08)

### 新增功能：Gemini 商业版支持

**功能描述**：为脚本添加 Gemini 商业版（`business.gemini.google`）的支持。

#### 技术挑战

Gemini 商业版与普通版有以下重要差异：

| 特性       | 普通版 (gemini.google.com)  | 商业版 (business.gemini.google) |
| ---------- | --------------------------- | ------------------------------- |
| 编辑器类型 | Quill Editor (`.ql-editor`) | ProseMirror (`.ProseMirror`)    |
| DOM 结构   | 普通 DOM                    | Web Components + Shadow DOM     |
| 主容器     | 无                          | `<UCS-STANDALONE-APP>`          |

#### 主要修改

1. **新增 URL 匹配规则**

```javascript
// @match https://business.gemini.google/*
```

2. **站点检测变量**

```javascript
const isGeminiBusiness = window.location.hostname.includes(
  "business.gemini.google"
);
const isGemini =
  window.location.hostname.includes("gemini.google") && !isGeminiBusiness;
const isAnyGemini = isGemini || isGeminiBusiness;
```

3. **Shadow DOM 递归搜索**
   商业版的输入框在 Shadow DOM 中，普通的 `document.querySelector` 无法访问。新增了递归搜索函数。

4. **元素过滤逻辑**
   为避免误匹配（如顶部搜索框），添加了验证函数 `isValidChatInput()`。

5. **新增方法**

- `findAndInsertForBusiness()`: 商业版专用的异步查找和插入方法
- `insertToGeminiBusiness()`: 商业版专用的文本插入方法

---

## 优化记录

### 移除插入后的悬浮条 (1.1.0)

- 移除 `selectPrompt()` 中显示"当前提示词"悬浮条的逻辑
- 原因：插入后已有 toast 提示，悬浮条属于冗余

### 修复标题换行问题 (1.1.0)

- 添加 `white-space: nowrap` 防止标题换行
- "Gemini Enterprise" 缩短为 "Enterprise"
