# Pencil MCP 操作手册

> 基于官方文档 + 实测验证。所有操作已逐项测试，确认行为与预期一致。

---

## 核心原则

1. **binding 变量仅在同一 `batch_design` 调用内有效**，跨调用必须用节点 ID（字符串）
2. **节点 ID 以数字开头时**，必须用字符串包裹（如 `I("1GmYb", {...})`），否则 JS 解析器将 `1GmYb` 识别为非法数字字面量
3. **flexbox 布局下 x/y 无效**：父节点使用 `layout: "vertical"` 或 `"horizontal"` 时，子节点位置由布局引擎自动排列
4. **字体族名**：避免使用不存在或不规范的字体名（如 `system-ui` 单独使用会报错），用 `"Inter, system-ui, sans-serif"` 或系统默认
5. **变量引用**：用 `$--varName` 或 `$alias:varName` 引用设计令牌，不要硬编码颜色值

---

## 操作语法总览

| 操作 | 函数 | 说明 | 路径 |
|------|------|------|------|
| Insert | `I(parent, props)` | 插入新节点 | 节点 ID（字符串） |
| Update | `U(nodeId, props)` | 更新节点属性 | 节点 ID（字符串） |
| Delete | `D(nodeId)` | 删除节点及其子树 | 节点 ID（字符串） |
| Move | `M(nodeId, parentId)` | 移动节点到新父节点 | 两个字符串参数 |
| Replace | `R(nodeId, newNode)` | 替换整个节点 | 节点 ID（字符串） |
| Copy | `C(nodeId, parentId)` | **有 API bug** | 见下方替代方案 |

---

## 逐项验证

### I (Insert) — 插入节点

```javascript
// 插入到文档根级别
frame=I(document, {type: "frame", name: "MyFrame", layout: "vertical", width: 1440, height: 900, fill: "#FAFAFA", x: 0, y: 0})

// 插入到已有节点下（用字符串 ID）
card=I("parentNodeId", {type: "frame", name: "Card", layout: "vertical", width: 300, height: 150, fill: "#FFFFFF", borderRadius: 8, padding: 20, gap: 8})

// 嵌套插入——在同一次调用内用 binding
title=I(card, {type: "text", content: "Hello", fontFamily: "Inter, system-ui, sans-serif", fontSize: 18, fontWeight: "600"})
```

**已验证**：
- `I(document, ...)` 插入到文档根级别
- `I("nodeId", ...)` 跨批次引用已有节点
- `I(binding, ...)` 同批次内引用刚创建的节点
- ID 以数字开头必须加引号

### U (Update) — 更新属性

```javascript
// 跨批次更新
U("nodeId", {fill: "#F0F7FF", stroke: {align: "inside", fill: "#3B82F6", thickness: 2}})
U("textNodeId", {content: "New Text", fontSize: 22})
```

**已验证**：可跨批次更新任何节点属性。

### D (Delete) — 删除节点

```javascript
D("nodeId")
```

**已验证**：删除节点及其所有子节点。

### M (Move) — 移动节点

```javascript
// 移动子节点到新父节点
M("childNodeId", "newParentNodeId")
```

**已验证**：两个字符串参数。不是斜杠路径，不是 binding 变量。

### R (Replace) — 替换节点

```javascript
// 直接用节点 ID 替换（不需要完整路径）
R("oldNodeId", {type: "frame", name: "New Frame", fill: "#F0FFF0", width: 300, height: 200})
```

**已验证**：用字符串 ID 直接替换，不需要 `document/parent/child` 路径。替换后原子节点的子节点丢失，需要重新插入。

### C (Copy) — 复制节点（有 bug）

```javascript
// C 操作在 MCP 端有 bug：传字符串报 TypeError: Cannot use 'in' operator，传对象报 must be a string
// 替代方案：用 ref 实例化
dup=I("parentNodeId", {type: "ref", ref: "sourceNodeId"})
```

**已验证**：用 `ref` 替代 C 操作，效果等同。

---

## 组件实例化

```javascript
// shadcn/ui 组件实例化（带 descendants 自定义）
btn=I("parentNodeId", {type: "ref", ref: "1:VSnC2", descendants: {"1:8tnXG": {content: "Submit"}}})

// 不带自定义的简单实例
icon=I("parentNodeId", {type: "icon_font", iconFontFamily: "lucide", iconFontName: "plus", width: 16, height: 16, fill: "#FFFFFF"})
```

**已验证**：
- `ref` 引用可复用组件 ID（如 `1:VSnC2` 是 Button/Default）
- `descendants` 用斜杠路径引用子节点：`"parentId/childId"`
- 图标用 `icon_font` 类型，支持 lucide、feather、Material Symbols

---

## 布局验证工具

### snapshot_layout — 布局分析

```javascript
snapshot_layout({filePath: "file.pen", parentId: "nodeId", maxDepth: 3})
```

返回每个节点的 `x, y, width, height` 及 `problems`（如 "partially clipped"）。用于检测重叠、裁剪等布局问题。

### get_screenshot — 截图渲染

```javascript
get_screenshot({filePath: "file.pen", nodeId: "nodeId"})
```

返回节点渲染为 PNG 的截图。用于视觉验证。

---

## 布局模式速查

### 侧边栏 + 主内容

```javascript
screen=I(document, {type: "frame", name: "Dashboard", layout: "horizontal", width: 1440, height: 900, fill: "#FAFAFA"})
sidebar=I(screen, {type: "ref", ref: "1:PV1ln", height: "fill_container"})
main=I(screen, {type: "frame", layout: "vertical", width: "fill_container", padding: 32, gap: 24})
```

### 卡片网格

```javascript
grid=I(container, {type: "frame", layout: "horizontal", width: "fill_container", gap: 16})
card=I(grid, {type: "ref", ref: "1:pcGlv", width: "fill_container"})
```

### 指标卡片

```javascript
metrics=I(main, {type: "frame", layout: "horizontal", width: "fill_container", gap: 16})
metric=I(metrics, {type: "frame", layout: "vertical", width: "fill_container", padding: 24, gap: 8, fill: "#FFFFFF", borderRadius: 8})
```

### 表格

用 ref 引用数据表格组件：

```javascript
table=I(main, {type: "ref", ref: "1:shadcnDataTable", width: "fill_container"})
```

或手动构建：

```javascript
table=I(main, {type: "frame", layout: "vertical", width: "fill_container", fill: "#FFFFFF", borderRadius: 8, stroke: {align: "inside", fill: "#E5E5E5", thickness: 1}})
row=I(table, {type: "frame", layout: "horizontal", width: "fill_container", height: 48, gap: 16})
cell=I(row, {type: "text", content: "Cell Content", width: 200})
```

---

## 设计令牌（Variables）

Pencil 支持变量系统，用 `$` 前缀引用：

| 令牌 | 用途 |
|------|------|
| `$--background` | 页面背景 |
| `$--foreground` | 主文字色 |
| `$--muted-foreground` | 次要文字色 |
| `$--card` | 卡片背景 |
| `$--border` | 边框色 |
| `$--primary` | 主操作色 |
| `$--secondary` | 次操作色 |
| `$--destructive` | 危险色 |

shadcn/ui 主题令牌（带 `1:` 前缀）：

| 令牌 | 用途 |
|------|------|
| `$1:--primary` | shadcn 主色 |
| `$1:--primary-foreground` | shadcn 主色前景 |
| `$1:--secondary` | shadcn 次色 |
| `$1:--background` | shadcn 背景 |
| `$1:--card` | shadcn 卡片 |
| `$1:--border` | shadcn 边框 |
| `$1:--sidebar` | shadcn 侧边栏背景 |

**规则**：优先使用 shadcn 主题令牌（`$1:--*`），保证与 shadcn/ui 组件一致。

---

## 常用 shadcn 组件 ID

| 组件 | ID | 说明 |
|------|-----|------|
| Sidebar | `1:PV1ln` | 侧边栏 |
| Button/Default | `1:VSnC2` | 主按钮 |
| Button/Secondary | `1:e8v1X` | 次按钮 |
| Button/Outline | `1:C10zH` | 描边按钮 |
| Button/Destructive | `1:YKnjc` | 危险按钮 |
| Button/Ghost | `1:3f2VW` | 幽灵按钮 |
| Icon Button/Default | `1:urnwK` | 图标按钮 |
| Card | `1:pcGlv` | 卡片（有 header/content/actions slots） |
| Card/Plain | `1:fpgbn` | 纯内容卡片 |
| Card/Image | `1:JENPq` | 图片卡片 |
| Card Action | `1:PiMGI` | 行动卡片 |
| Badge/Default | `1:UjXug` | 标签 |
| Badge/Secondary | `1:WuUMk` | 次标签 |
| Badge/Destructive | `1:YvyLD` | 危险标签 |
| Input Group/Default | `1:1415a` | 输入框组 |
| Input/Default | `1:fEUdI` | 输入框 |
| Textarea Group/Default | `1:BjRan` | 文本域组 |
| Select Group/Default | `1:w5c1O` | 下拉选择 |
| Checkbox/Checked | `1:ovuDP` | 复选框 |
| Switch/Checked | `1:c8fiq` | 开关 |
| Progress | `1:hahxH` | 进度条 |
| Alert/Default | `1:QyzNg` | 警告框 |
| Alert/Destructive | `1:K53jF` | 危险警告 |
| Tabs | `1:PbofX` | 标签页容器 |
| Tab Item/Active | `1:coMmv` | 活动标签 |
| Tab Item/Inactive | `1:QY0Ka` | 非活动标签 |
| Dialog | `1:OtykB` | 对话框 |
| Dropdown | `1:cTN8T` | 下拉菜单 |
| Modal/Left | `1:oVUJY` | 左模态 |
| Modal/Center | `1:X6bmd` | 中模态 |
| Pagination | `1:U5noB` | 分页 |
| Data Table | `1:shadcnDataTable` | 完整数据表格 |
| Table Row | `1:LoAux` | 表格行 |
| Table Cell | `1:FulCp` | 表格单元格 |
| Sidebar Item/Active | `1:qCCo8` | 活动导航项 |
| Sidebar Item/Default | `1:jBcUh` | 默认导航项 |
| Sidebar Section Title | `1:24cM4` | 导航分组标题 |
| Avatar/Text | `1:DpPVg` | 文字头像 |
| Tooltip | `1:lxrnE` | 工具提示 |
| Accordion/Open | `1:pfIN1` | 手风琴 |
| Breadcrumb Item/Default | `1:KUk4t` | 面包屑 |
| Breadcrumb Item/Current | `1:FBqua` | 当前面包屑 |

---

## 工作流程最佳实践

1. **先建框架**：用 `I(document, ...)` 创建主框架
2. **逐层填充**：在框架内用 `I("parentId", ...)` 添加内容
3. **每次操作后验证**：用 `get_screenshot` 看效果，用 `snapshot_layout` 查问题
4. **组件优先**：优先用 `ref` 引用 shadcn 组件，不要自己造轮子
5. **颜色用令牌**：全部用 `$1:--*` 变量，不要硬编码 hex 值
6. **小步迭代**：每批 25 个操作以内，不要一次性塞太多
7. **字体简洁**：用 `"Inter"` 或 `"system-ui"`，不要用复杂字体栈

---

## 已知坑

1. **C (Copy) 操作有 API bug** → 用 `ref` 替代
2. **数字开头的 ID 必须加引号** → `I("1GmYb", {...})` 不能写成 `I(1GmYb, {...})`
3. **binding 不跨批次** → 每次 `batch_design` 都是新的作用域
4. **flexbox 下 x/y 无效** → 布局引擎会忽略，靠 gap/padding 控制间距
5. **fontFamily 'system-ui' 单独使用报错** → 用 `"Inter"` 或完整栈 `"Inter, system-ui, sans-serif"`
6. **R (Replace) 会丢失子节点** → 替换后需重新插入子内容

---

## 与 Impeccable 的分工

Pencil 和 Impeccable 在 hq-scrum SOP 中扮演不同阶段的角色：

| 阶段 | 工具 | 输入 | 输出 |
|------|------|------|------|
| Phase 1 Sprint 0 原型 | Pencil MCP | PRD 文字/ASCII + design-constraints.md | `.pen` 文件 + 真实 UI 截图 |
| Phase 4 实现后抛光 | Impeccable | 已有 HTML/组件代码 + `.impeccable.md` | 代码 diff（针对性修改） |
| Phase 5 验证评审 | Impeccable | 已有页面 | `/critique` 评分报告 + `/audit` 技术检查 |

**Pencil** 是"从无到有"——把 PRD 需求翻译为视觉原型，约束规则确保美感底线。
**Impeccable** 是"从有到精"——对已实现的代码做精细化设计调整，18 个命令精确控制每个维度。

两者配合：Pencil 定方向（原型验证），Impeccable 做收尾（代码抛光）。
