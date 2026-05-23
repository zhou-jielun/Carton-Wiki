# FAQ/专业问题模块 - 全面探索报告

## 📋 执行摘要
本报告详细记录了 `/Users/sophie/Desktop/Jay/LLM/Carton-Wiki/index.html` 中 FAQ/专业问题模块的完整实现状态，包括数据结构、Storage 层、渲染函数、CSS 样式、键盘快捷键等所有关键部分。

---

## 1️⃣ FAQ 数据结构 (Storage._defaults 中 faq 的定义)

### 📍 位置
- **行号**: 1405
- **函数**: `Storage._defaults()`

### 📦 默认数据结构
```javascript
return { 
  customers: [], 
  followups: [], 
  quotations: [], 
  socialMedia: {schedules:[], drafts:[], posts:[]}, 
  scripts: [], 
  competitors: [], 
  faq: [],  // ← FAQ 数据存储在这里
  todoDone: {}, 
  config: { ... }
};
```

### 🔑 问题对象字段
根据代码分析（行号 4188-4195, 4234-4235），每个 FAQ 问题对象包含以下字段：

| 字段 | 类型 | 必填 | 说明 | 行号 |
|------|------|------|------|------|
| `id` | String | ✅ | 唯一标识符，使用 `Utils.id()` 生成 | 4190, 4235 |
| `question` | String | ✅ | 问题标题 | 4191, 4235 |
| `answer` | String | ❌ | 问题解答（支持 HTML 和 Markdown） | 4192, 4235 |
| `category` | String | ❌ | 设备分类，来自 `EQUIPMENT_TYPES` | 4193, 4235 |
| `starred` | Boolean | ✅ | 是否收藏（默认 false） | 4194, 4235 |
| `createdAt` | String | ✅ | 创建时间（ISO 8601 格式） | 4195, 4235 |
| `updatedAt` | String | ❌ | 更新时间（仅编辑时更新） | 4233 |

### ⚠️ 重要发现
- ✅ **有 `createdAt` 字段**: 在 `saveFaq()` 和 `saveAiAnswerToFaq()` 中创建时自动添加
- ✅ **有 `updatedAt` 字段**: 在 `saveFaq()` 编辑时更新（行 4233）
- ❌ **无 `usageCount` 字段**: 当前没有使用次数统计
- ❌ **无预设示例数据**: `_defaults()` 中 `faq: []` 是空数组，没有初始数据

---

## 2️⃣ Storage 数据层

### 📍 位置
- **行号**: 1402-1489
- **对象**: `Storage`

### 🔧 FAQ 相关方法

#### `Storage.getFaq()` 
- **行号**: 1479
- **功能**: 获取所有 FAQ 数据
- **代码**:
```javascript
getFaq() { return this.get().faq || []; }
```

#### `Storage.setFaq(v)`
- **行号**: 1480
- **功能**: 保存 FAQ 数据到 localStorage
- **代码**:
```javascript
setFaq(v) { this.get().faq = v; this.save(); }
```

### 🔧 Config 结构
- **行号**: 1405-1421
- **完整结构**:
```javascript
config: {
  whatsappTemplates: [...],  // WhatsApp 模板
  emailTemplates: [...],      // 邮件模板
  lastQuoteNumber: {          // 报价编号
    year: 2026,
    month: 5,
    sequence: 0
  },
  aiApiKey: '',              // AI API Key
  aiProvider: 'deepseek'     // AI 提供商
}
```

### ⚠️ 重要发现
- ❌ **Config 中无 `usage` 相关字段**: 当前 config 没有使用统计或使用次数相关的字段
- ✅ **有 `getUsage()` 方法**: 行 1484-1488，但只是估算 localStorage 占用空间，不是 FAQ 使用统计

---

## 3️⃣ renderFaq 渲染函数 (完整代码)

### 📍 位置
- **行号**: 3936-4016
- **函数**: `Router.renderFaq(content, title, subtitle, actions)`

### 📏 行号范围
- **起始行**: 3936
- **结束行**: 4016
- **总行数**: 81 行

### 🔀 分支逻辑

#### 1. 搜索模式 (有搜索词)
- **条件**: `searchQuery` 不为空（行 3979）
- **行号**: 3979-3999
- **功能**:
  - 显示搜索结果数量（行 3981）
  - 提供"清除搜索"按钮（行 3982）
  - 如无结果，显示空状态和 AI 智能回答按钮（行 3984-3994）
  - 如有结果，调用 `faqSearchResult()` 渲染搜索结果卡片（行 3997）

#### 2. 浏览模式 (无搜索词)
- **条件**: `searchQuery` 为空（行 4001）
- **行号**: 4001-4014
- **功能**:
  - 显示收藏夹区域（行 4002-4009）
  - 显示所有问题卡片（行 4011-4013）
  - 如无数据，显示空状态（行 4012）

### 🔍 搜索框实现
- **行号**: 3966-3970
- **代码**:
```html
<div class="search-box" style="flex:1;min-width:200px;">
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
    <circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>
  </svg>
  <input type="text" placeholder="搜索问题/解答..." value="${searchQueryRaw}" 
         onkeydown="if(event.key==='Enter'){sessionStorage.setItem('faq_search',this.value);Router.handleRoute();}">
</div>
```
- **特性**:
  - 支持回车键触发搜索（行 3969）
  - 使用 `sessionStorage` 存储搜索词（行 3969）
  - 搜索词会保留在输入框中（行 3969）

### 🏷️ 筛选按钮实现
- **行号**: 3971-3975
- **代码**:
```html
<select class="filter-select" onchange="sessionStorage.setItem('faq_filter_category',this.value);Router.handleRoute();">
  <option value="all">全部分类</option>
  ${EQUIPMENT_TYPES.map(t => `<option value="${t}" ...>${t}</option>`).join('')}
  <option value="其他">其他</option>
</select>
```
- **特性**:
  - 使用 `sessionStorage` 存储筛选条件（行 3971）
  - 包含 `EQUIPMENT_TYPES` 所有分类（行 3973）
  - 支持"其他"分类（行 3974）

### ⭐ 收藏夹区域渲染
- **行号**: 4002-4009
- **条件**: `starred.length > 0`
- **布局**: 2 列网格（行 4005）
- **代码**:
```html
<div class="card" style="margin-bottom:16px;padding:16px 20px;">
  <div style="font-size:14px;font-weight:600;margin-bottom:12px;">★ 收藏夹 (${starred.length})</div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;">
    ${starred.map((f, i) => faqCard(f, i)).join('')}
  </div>
</div>
```

---

## 4️⃣ faqCard 函数 (浏览模式卡片)

### 📍 位置
- **行号**: 4018-4037
- **函数**: `faqCard(f, index)`

### 📏 行号范围
- **起始行**: 4018
- **结束行**: 4037
- **总行数**: 20 行

### 🎨 卡片布局结构
```html
<div class="faq-card ${hueClass}" onclick="toggleFaqAnswer(this)">
  <!-- 问题和收藏按钮 -->
  <div class="faq-question">
    <span class="faq-star ..." onclick="toggleFaqStar('${f.id}')">
      <svg>...</svg>
    </span>
    <span style="flex:1;">${f.question}</span>
  </div>
  
  <!-- 分类标签 -->
  <div style="display:flex;gap:6px;margin-top:8px;">
    <span class="badge badge-gray">${f.category || '未分类'}</span>
  </div>
  
  <!-- 答案区域（默认隐藏） -->
  <div class="faq-answer">${answerHtml}</div>
  
  <!-- 操作按钮 -->
  <div style="display:flex;gap:8px;margin-top:8px;...">
    <button onclick="editFaq('${f.id}')">编辑</button>
    <button onclick="deleteFaq('${f.id}')" style="color:var(--apple-red);">删除</button>
  </div>
</div>
```

### 🎨 卡片样式类
- **主容器**: `faq-card` + `faq-hue-{0-7}` (8 种颜色循环)
- **问题**: `faq-question`
- **答案**: `faq-answer` (默认 `display: none`，展开时添加 `open` 类)
- **收藏按钮**: `faq-star` + `active` (已收藏)

### ❓ 是否有「复制」按钮？
- **答案**: ❌ **没有**
- **当前操作按钮**（行 4032-4035）:
  1. 编辑按钮（行 4033）
  2. 删除按钮（行 4034）
- **对比**: 搜索模式卡片 (`faqSearchResult`) 有"复制"按钮（行 4060），但浏览模式卡片没有

---

## 5️⃣ faqSearchResult 函数 (搜索模式卡片)

### 📍 位置
- **行号**: 4040-4065
- **函数**: `faqSearchResult(f, searchQuery)`

### 📏 行号范围
- **起始行**: 4040
- **结束行**: 4065
- **总行数**: 26 行

### 🎨 卡片布局结构
```html
<div class="faq-search-result" id="faq-sr-${f.id}">
  <!-- 问题和收藏按钮 -->
  <div style="display:flex;align-items:flex-start;gap:8px;...">
    <span class="faq-star ..." onclick="toggleFaqStar('${f.id}')">
      <svg>...</svg>
    </span>
    <div style="flex:1;min-width:0;">
      <div class="faq-sr-question">${highlightMatches(f.question, searchQuery)}</div>
      <div style="display:flex;gap:6px;margin-top:4px;">
        <span class="badge badge-gray">${f.category || '未分类'}</span>
      </div>
    </div>
  </div>
  
  <!-- 答案预览 -->
  <div class="faq-sr-preview">${preview || '（暂无解答内容）'}</div>
  
  <!-- 完整答案（默认隐藏） -->
  <div class="faq-answer">${answerHtml}</div>
  
  <!-- 操作按钮 -->
  <div class="faq-sr-actions">
    <button onclick="toggleFaqAnswer(...)">展开详情</button>
    <button onclick="copyFaqContent('${f.id}')">复制</button>  ✅ 有复制按钮
    <button onclick="editFaq('${f.id}')">编辑</button>
    <button onclick="deleteFaq('${f.id}')" style="color:var(--apple-red);">删除</button>
  </div>
</div>
```

### ✅ 是否有 copyFaqContent？
- **答案**: ✅ **有**
- **函数位置**: 行 4068-4077
- **函数名**: `copyFaqContent(id)`
- **功能**: 
  - 获取 FAQ 问题和答案（行 4069-4071）
  - 去除 HTML 标签（行 4072）
  - 复制到剪贴板（行 4073-4075）
  - 显示提示（行 4076）

### 🔍 搜索高亮
- **函数**: `highlightMatches(text, query)` (行 1800-1810)
- **使用位置**: 行 4050
- **效果**: 搜索关键词会被 `<span class="search-highlight">` 包裹

---

## 6️⃣ FAQ 相关函数列表

### 📋 完整函数清单

| 函数名 | 行号 | 参数 | 功能 | 调用位置 |
|--------|------|------|------|----------|
| `Router.renderFaq()` | 3936-4016 | `content, title, subtitle, actions` | 渲染 FAQ 主页面 | 路由系统 |
| `faqCard()` | 4018-4037 | `f, index` | 渲染浏览模式卡片 | renderFaq (行 4006, 4013) |
| `faqSearchResult()` | 4040-4065 | `f, searchQuery` | 渲染搜索模式卡片 | renderFaq (行 3997) |
| `copyFaqContent()` | 4068-4077 | `id` | 复制 FAQ 内容到剪贴板 | faqSearchResult (行 4060) |
| `formatFaqAnswer()` | 4080-4103 | `text` | 格式化 FAQ 答案为 HTML | faqCard (行 4020), faqSearchResult (行 4041) |
| `formatFaqAnswerMarkdown()` | 4106-4130 | `md` | Markdown 转 HTML | formatFaqAnswer (行 4093, 4100) |
| `parseMarkdownTable()` | 4136-4150 | `md` | 解析 Markdown 表格 | formatFaqAnswerMarkdown (行 4128) |
| `askFaqAi()` | 4153-4179 | `question` | AI 智能回答 | renderFaq (行 3989) |
| `saveAiAnswerToFaq()` | 4181-4199 | `question, btn` | 保存 AI 答案到知识库 | askFaqAi (行 4170) |
| `toggleFaqAnswer()` | 4201-4204 | `card` | 展开/折叠答案 | faqCard (行 4021), faqSearchResult (行 4045, 4059) |
| `toggleFaqStar()` | 4206-4210 | `id` | 切换收藏状态 | faqCard (行 4023), faqSearchResult (行 4046) |
| `showFaqForm()` | 4212-4224 | `id` | 显示新增/编辑表单 | faqCard (行 4033), faqSearchResult (行 4061) |
| `saveFaq()` | 4226-4239 | `e` | 保存 FAQ（新增/编辑） | showFaqForm (行 4216) |
| `editFaq()` | 4241 | `id` | 编辑 FAQ | faqCard (行 4033), faqSearchResult (行 4061) |
| `deleteFaq()` | 4243-4249 | `id` | 删除 FAQ | faqCard (行 4034), faqSearchResult (行 4062) |

### 🔑 关键函数详解

#### `toggleFaqStar(id)`
- **行号**: 4206-4210
- **参数**: `id` (FAQ 问题 ID)
- **功能**: 
  1. 从 Storage 获取 FAQ 列表（行 4207）
  2. 查找对应 ID 的问题（行 4208）
  3. 切换 `starred` 状态（行 4209）
  4. 保存到 Storage 并重新渲染（行 4209）

#### `showFaqForm(id)`
- **行号**: 4212-4224
- **参数**: `id` (可选，编辑时传入)
- **功能**:
  1. 如果 `id` 存在，获取现有数据（行 4213）
  2. 打开模态框（行 4214）
  3. 渲染表单（行 4215-4223）
  4. 表单包含：问题、解答、设备分类（行 4218-4220）

#### `saveFaq(e)`
- **行号**: 4226-4239
- **参数**: `e` (表单提交事件)
- **功能**:
  1. 阻止默认提交（行 4227）
  2. 获取表单数据（行 4228-4229）
  3. 如果 `id` 存在，更新现有问题（行 4231-4233）
  4. 如果 `id` 不存在，新增问题（行 4234-4235）
  5. 保存到 Storage 并重新渲染（行 4237-4238）

---

## 7️⃣ CSS 样式

### 📋 完整样式清单

#### `.faq-card` (浏览模式卡片)
- **行号**: 649-650
- **代码**:
```css
.faq-card {
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  background: var(--card);
  padding: 20px 20px 20px 24px;
  transition: var(--transition);
  cursor: pointer;
  position: relative;
  overflow: hidden;
  border-left: 4px solid transparent;
}
.faq-card:hover {
  box-shadow: var(--shadow-hover);
  transform: translateY(-2px);
}
```

#### `.faq-hue-{0-7}` (8 种颜色循环)
- **行号**: 651-658
- **代码**:
```css
.faq-hue-0 { border-left-color: #C8B89A; }
.faq-hue-1 { border-left-color: #9AB0C4; }
.faq-hue-2 { border-left-color: #C4A0A0; }
.faq-hue-3 { border-left-color: #9AB89A; }
.faq-hue-4 { border-left-color: #A89AB8; }
.faq-hue-5 { border-left-color: #C4B48A; }
.faq-hue-6 { border-left-color: #8AB4AC; }
.faq-hue-7 { border-left-color: #C4A88E; }
```

#### `.faq-question` (问题标题)
- **行号**: 659
- **代码**:
```css
.faq-question {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-primary);
  display: flex;
  align-items: center;
  gap: 8px;
}
```

#### `.faq-answer` (答案区域)
- **行号**: 660-661
- **代码**:
```css
.faq-answer {
  font-size: 14px;
  line-height: 1.7;
  color: var(--text-secondary);
  margin-top: 12px;
  padding: 16px;
  border-radius: 12px;
  background: rgba(0,0,0,0.03);
  display: none;
}
.faq-answer.open {
  display: block;
}
```

#### `.faq-star` (收藏按钮)
- **行号**: 677-679
- **代码**:
```css
.faq-star {
  cursor: pointer;
  color: #C7C7CC;
  transition: var(--transition);
  flex-shrink: 0;
}
.faq-star.active {
  color: var(--apple-blue);
  fill: var(--apple-blue);
}
.faq-star:hover {
  transform: scale(1.15);
}
```

#### `.faq-search-result` (搜索结果卡片)
- **行号**: 681-689
- **代码**:
```css
.faq-search-result {
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  background: var(--card);
  padding: 16px 20px;
  transition: var(--transition);
  cursor: pointer;
  border: 1.5px solid rgba(0,122,255,0.08);
  margin-bottom: 8px;
}
.faq-search-result:hover {
  box-shadow: var(--shadow-hover);
  border-color: rgba(0,122,255,0.2);
}
.faq-search-result .faq-sr-header {
  display: flex;
  align-items: flex-start;
  gap: 10px;
}
.faq-search-result .faq-sr-question {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-primary);
  flex: 1;
  line-height: 1.4;
}
.faq-search-result .faq-sr-preview {
  font-size: 13px;
  line-height: 1.6;
  color: var(--text-secondary);
  margin-top: 8px;
  padding-left: 28px;
}
.faq-search-result .faq-sr-actions {
  display: flex;
  gap: 6px;
  align-items: center;
  margin-top: 10px;
  padding-top: 8px;
  border-top: 1px solid rgba(0,0,0,0.04);
  padding-left: 28px;
}
.faq-search-result .faq-sr-actions .btn {
  padding: 3px 10px;
  font-size: 12px;
}
.faq-search-result .faq-answer {
  display: none;
  margin: 12px 0 0 28px;
}
.faq-search-result .faq-answer.open {
  display: block;
}
```

#### `.search-highlight` (搜索高亮)
- **行号**: 549
- **代码**:
```css
.search-highlight {
  background: #FFF3CD;
  padding: 0 1px;
  border-radius: 2px;
}
```

### 🌙 深色模式样式

#### `.dark-mode .faq-card`
- **行号**: 695

#### `.dark-mode .faq-hue-{0-7}`
- **行号**: 696-703

#### `.dark-mode .faq-answer`
- **行号**: 704

#### `.dark-mode .faq-star`
- **行号**: 714

#### `.dark-mode .faq-search-result`
- **行号**: 691-693

---

## 8️⃣ 键盘快捷键

### 📍 位置
- **行号**: 5919-5956
- **事件监听**: `document.addEventListener('keydown', ...)`

### ⌨️ 现有快捷键清单

#### 全局快捷键（无焦点时）
| 快捷键 | 功能 | 行号 | 说明 |
|--------|------|------|------|
| `Cmd/Ctrl + N` | 新增客户 | 5924 | 仅在非输入框焦点时 |
| `Cmd/Ctrl + F` | 聚焦搜索框 | 5925-5929 | 仅在客户页面 |
| `Cmd/Ctrl + 1` | 跳转到首页 | 5930 | |
| `Cmd/Ctrl + 2` | 跳转到客户 | 5931 | |
| `Cmd/Ctrl + 3` | 跳转到报价 | 5932 | |
| `Cmd/Ctrl + 4` | 跳转到设置 | 5933 | |
| `Cmd/Ctrl + 5` | 跳转到社媒 | 5934 | |
| `Cmd/Ctrl + 6` | 跳转到话术 | 5935 | |
| `Cmd/Ctrl + 7` | 跳转到对手 | 5936 | |
| `Cmd/Ctrl + 8` | 跳转到 FAQ | 5937 | ✅ 已有 FAQ 快捷键 |
| `Cmd/Ctrl + 9` | 跳转到计算 | 5938 | |
| `Cmd/Ctrl + 0` | 跳转到模拟 | 5939 | |
| `Escape` | 关闭模态框 | 5940 | |

#### 客户详情页快捷键
| 快捷键 | 功能 | 行号 | 说明 |
|--------|------|------|------|
| `E` | 编辑客户 | 5946 | 仅在客户详情页 |
| `F` | 新增跟进 | 5947 | 仅在客户详情页 |
| `Q` | 新增报价 | 5948 | 仅在客户详情页 |
| `D` | 删除客户 | 5949 | 仅在客户详情页 |

### ❓ 是否有 Ctrl+K？
- **答案**: ❌ **没有**
- **当前状态**: 没有全局搜索快捷键
- **建议**: 可以添加 `Cmd/Ctrl + K` 作为全局搜索快捷键

### 🔍 输入框检测
- **行号**: 5921
- **代码**:
```javascript
const tag = e.target.tagName;
if ((tag === 'INPUT' || tag === 'TEXTAREA' || tag === 'SELECT') && e.key !== 'Escape') return;
```
- **功能**: 如果在输入框内，除了 `Escape` 键，其他快捷键不触发

---

## 9️⃣ FAQ 的数据

### 🏷️ EQUIPMENT_TYPES 常量（分类列表）
- **行号**: 1357
- **代码**:
```javascript
const EQUIPMENT_TYPES = [
  '瓦楞纸板生产线',
  '瓦楞纸箱生产线',
  '高速柔印开槽模切机',
  '自动打包机',
  '全自动粘箱打包一体机',
    '全自动粘钉打包一体机',
  '其他设备'
];
```
- **数量**: 7 个分类 + "其他"

### 📊 预设的示例 FAQ 数据
- **答案**: ❌ **没有预设数据**
- **当前状态**: `faq: []` (行 1405)
- **说明**: 用户需要手动添加 FAQ 数据，或者通过网络下载示例数据（行 5707-5709）

---

## 🔟 辅助函数

### `stripHtml(html)`
- **行号**: 1795-1798
- **功能**: 去除 HTML 标签和 Markdown 标记，返回纯文本
- **使用位置**: `copyFaqContent()` (行 4072), `faqSearchResult()` (行 4042)

### `highlightMatches(text, query)`
- **行号**: 1800-1810
- **功能**: 高亮文本中的搜索关键词
- **使用位置**: `faqSearchResult()` (行 4050)

### `formatFaqAnswer(text)`
- **行号**: 4080-4103
- **功能**: 格式化 FAQ 答案（Markdown + HTML → HTML）
- **使用位置**: `faqCard()` (行 4020), `faqSearchResult()` (行 4041)

### `formatFaqAnswerMarkdown(md)`
- **行号**: 4106-4130
- **功能**: Markdown 转 HTML（内部函数）
- **使用位置**: `formatFaqAnswer()` (行 4093, 4100)

### `parseMarkdownTable(md)`
- **行号**: 4136-4150
- **功能**: 解析 Markdown 表格
- **使用位置**: `formatFaqAnswerMarkdown()` (行 4128)

---

## 📊 总结

### ✅ 已有功能
1. ✅ FAQ 数据结构完整（id, question, answer, category, starred, createdAt, updatedAt）
2. ✅ Storage 层完整（getFaq, setFaq）
3. ✅ 渲染函数完整（renderFaq, faqCard, faqSearchResult）
4. ✅ 搜索功能完整（支持关键词分词、高亮）
5. ✅ 收藏功能完整（toggleFaqStar）
6. ✅ 增删改功能完整（showFaqForm, saveFaq, editFaq, deleteFaq）
7. ✅ 复制功能（仅搜索模式有 copyFaqContent）
8. ✅ AI 智能回答（askFaqAi, saveAiAnswerToFaq）
9. ✅ Markdown 支持（formatFaqAnswer）
10. ✅ 键盘快捷键（Cmd/Ctrl + 8 跳转 FAQ）

### ❌ 缺失功能
1. ❌ **浏览模式无复制按钮**（搜索模式有，浏览模式没有）
2. ❌ **无 `usageCount` 字段**（没有使用次数统计）
3. ❌ **无预设示例数据**（`faq: []` 是空数组）
4. ❌ **无 Ctrl+K 全局搜索快捷键**
5. ❌ **Config 中无 usage 相关字段**

### 🎯 改进建议
1. **添加浏览模式复制按钮**: 在 `faqCard()` 中添加"复制"按钮（参考 `faqSearchResult()` 行 4060）
2. **添加使用次数统计**: 在 FAQ 对象中添加 `usageCount` 字段，在 `copyFaqContent()` 或展开答案时递增
3. **添加预设示例数据**: 在 `Storage._defaults()` 中添加示例 FAQ 数据
4. **添加 Ctrl+K 快捷键**: 在键盘事件监听中添加全局搜索功能
5. **添加使用统计到 Config**: 在 `config` 中添加 `faqUsage` 字段

---

## 📎 附录：完整行号索引

| 功能/函数 | 起始行 | 结束行 | 行数 |
|-----------|--------|--------|------|
| `EQUIPMENT_TYPES` | 1357 | 1357 | 1 |
| `Storage._defaults()` | 1404 | 1422 | 19 |
| `Storage.getFaq()` | 1479 | 1479 | 1 |
| `Storage.setFaq()` | 1480 | 1480 | 1 |
| `stripHtml()` | 1795 | 1798 | 4 |
| `highlightMatches()` | 1800 | 1810 | 11 |
| `Router.renderFaq()` | 3936 | 4016 | 81 |
| `faqCard()` | 4018 | 4037 | 20 |
| `faqSearchResult()` | 4040 | 4065 | 26 |
| `copyFaqContent()` | 4068 | 4077 | 10 |
| `formatFaqAnswer()` | 4080 | 4103 | 24 |
| `formatFaqAnswerMarkdown()` | 4106 | 4130 | 25 |
| `parseMarkdownTable()` | 4136 | 4150 | 15 |
| `askFaqAi()` | 4153 | 4179 | 27 |
| `saveAiAnswerToFaq()` | 4181 | 4199 | 19 |
| `toggleFaqAnswer()` | 4201 | 4204 | 4 |
| `toggleFaqStar()` | 4206 | 4210 | 5 |
| `showFaqForm()` | 4212 | 4224 | 13 |
| `saveFaq()` | 4226 | 4239 | 14 |
| `editFaq()` | 4241 | 4241 | 1 |
| `deleteFaq()` | 4243 | 4249 | 7 |
| 键盘快捷键 | 5919 | 5956 | 38 |
| CSS: `.search-highlight` | 549 | 549 | 1 |
| CSS: `.faq-card` | 649 | 650 | 2 |
| CSS: `.faq-hue-{0-7}` | 651 | 658 | 8 |
| CSS: `.faq-question` | 659 | 659 | 1 |
| CSS: `.faq-answer` | 660 | 661 | 2 |
| CSS: `.faq-star` | 677 | 679 | 3 |
| CSS: `.faq-search-result` | 681 | 689 | 9 |

---

**报告生成时间**: 2026-05-23  
**文件版本**: `/Users/sophie/Desktop/Jay/LLM/Carton-Wiki/index.html`
