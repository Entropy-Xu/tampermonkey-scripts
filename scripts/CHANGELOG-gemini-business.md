# Gemini 商业版支持 - 变更说明

## 版本信息

- **版本**: 1.1.0
- **日期**: 2024-12-08
- **作者**: urzeye

## 背景

为油猴脚本 `gemini-helper.user.js` 添加 Gemini 商业版（`business.gemini.google`）的支持。

## 技术挑战

Gemini 商业版与普通版有以下重要差异：

| 特性       | 普通版 (gemini.google.com)  | 商业版 (business.gemini.google) |
| ---------- | --------------------------- | ------------------------------- |
| 编辑器类型 | Quill Editor (`.ql-editor`) | ProseMirror (`.ProseMirror`)    |
| DOM 结构   | 普通 DOM                    | Web Components + Shadow DOM     |
| 主容器     | 无                          | `<UCS-STANDALONE-APP>`          |

## 主要修改

### 1. 新增 URL 匹配规则

```javascript
// @match https://business.gemini.google/*
```

### 2. 站点检测变量

```javascript
const isGeminiBusiness = window.location.hostname.includes(
  "business.gemini.google"
);
const isGemini =
  window.location.hostname.includes("gemini.google") && !isGeminiBusiness;
const isAnyGemini = isGemini || isGeminiBusiness;
```

### 3. Shadow DOM 递归搜索

商业版的输入框在 Shadow DOM 中，普通的 `document.querySelector` 无法访问。新增了递归搜索函数：

```javascript
const searchInShadowDOM = (root, depth = 0) => {
  // 在 Shadow Root 中搜索
  // 递归处理嵌套的 Shadow DOM
};
```

### 4. 元素过滤逻辑

为避免误匹配（如顶部搜索框），添加了验证函数：

```javascript
const isValidChatInput = (element) => {
  if (element.type === "search") return false;
  if (element.classList.contains("main-input")) return false;
  if (element.getAttribute("aria-label")?.includes("搜索")) return false;
  // ...
};
```

### 5. 新增方法

- `findAndInsertForBusiness()`: 商业版专用的异步查找和插入方法
- `insertToGeminiBusiness()`: 商业版专用的文本插入方法

## 修改的文件

| 文件                            | 修改类型 |
| ------------------------------- | -------- |
| `scripts/gemini-helper.user.js` | 修改     |

## 测试验证

- ✅ Gemini 普通版 (gemini.google.com) 正常工作
- ✅ Gemini 商业版 (business.gemini.google) 正常工作
- ✅ Genspark 正常工作

如果输入框未找到，脚本会自动轮询等待最多 7.5 秒（15 次 × 500ms）。
