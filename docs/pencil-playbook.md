# Pencil MCP 操作手册

> 基于官方文档 + 实测验证。所有操作已逐项测试，确认行为与预期一致。

---

## 核心原则

1. **binding 变量仅在同一 `batch_design` 调用内有效**，跨调用必须用节点 ID（字符串）
2. **节点 ID 以数字开头时**，必须用字符串包裹（如 `I("1GmYb", {...})`），否则 JS 解析器将 `1GmYb` 识别为非法数字字面量
3. **flexbox 布局下 x/y 无效**：父节点使用 `layout: "vertical"` 或 `"horizontal"` 时，子节点位置由布局引擎自动排列
4. **字体族名**：避免使用不存在或不规范的字体名（如 `system-ui` 单独使用会报错），用 `"Inter, system-ui, sans-serif"` 或系统默认
5. **不确定操作时**：先用 `/web-access` 技能查阅 [Pencil 官方文档](https://docs.pencil.dev/for-developers/the-pen-format)，验证后再写入 playbook。所有 playbook 条目必须经过实际测试
6. **操作前检查**：每批 `batch_design` 前，先用 `batch_get` 读取当前文档顶层节点，确认目标节点不存在再创建。避免 binding 失效后重复插入
7. **无自动保存**：Pencil 不支持自动保存，每次修改后必须 Cmd+S 手动保存
8. **撤销/重做功能有限**：不要依赖撤销回退，重大操作前先保存/commit

### 开始工作前必须保存

**重要**：`/new` 创建的临时文档（未保存到磁盘）每次重新序列化时会重新分配节点 ID，导致之前记录的 ID 全部失效。

- 如果 `batch_design` 操作发生回滚，临时文档的 ID 会变化
- 一旦保存到 `.pen` 文件，ID 就固定了，不会变化

**工作流**：`/new` → 立即 Cmd+S 保存为 `.pen` 文件 → 开始 `batch_design`。不要在未保存的临时文档上做操作。

### 绝对不要做的事

9. **不要用 `replace_all_matching_properties` 做颜色替换**：它会替换子树中 ALL 匹配的属性，顺序执行导致连锁污染（例如 `#666666` → `#999999` 后再把 `#999999` 的节点也改了）。这是不可逆的破坏性操作。**正确做法**：用 `U("nodeId", {...})` 逐个节点更新，或设计阶段就用正确颜色
10. **不要全删重建**：这是操作不熟练的表现。用 `batch_get` 读当前状态 → 用 `U()` 更新 → 用 `I()` 增量添加。删除应该是最后手段
11. **不要批量修改变量引用**：`replace_all_matching_properties` 会把 `$1:--foreground` 变成 `\\$1:--foreground`（带转义），Pencil 渲染引擎无法解析。硬编码颜色值更安全

---

## .pen 文件结构（官方文档要点）

### 文件本质
- JSON 格式的对象树，类似 HTML/SVG
- 每个对象必须有 `id`（文档内唯一）和 `type`（rectangle/frame/text/ref 等）
- 顶层对象用 `x`/`y` 定位在无限画布上
- 嵌套对象相对于父节点左上角定位

### 组件系统
```javascript
// 标记为可复用组件
{ "id": "foo", "type": "frame", "reusable": true, ... }

// 创建组件实例
{ "id": "bar", "type": "ref", "ref": "foo", ... }

// 实例覆盖属性
{ "id": "baz", "type": "ref", "ref": "foo", "fill": "#FF0000" }

// 自定义后代节点（用斜杠路径）
{ "type": "ref", "ref": "card", "descendants": { "title": { "content": "New Title" } } }
```

### 设计库
- `.lib.pen` 后缀的文件标记为设计库
- 库中的组件修改会自动反映到所有引用处
- shadcn 组件就是作为设计库导入的（`pencil:shadcn.lib.pen`）

### 变量系统
- 类似 CSS 自定义属性/设计令牌
- 支持颜色（HEX）、数字、字符串
- 支持主题（light/dark）
- shadcn 的 `1:--*` 变量来自导入的设计库，**不可修改**
- 自定义变量（无前缀）可以自由创建和修改
- **重要**：`replace_all_matching_properties` 不能正确生成变量引用（会变成 `\\$var` 转义字符串）

### Slots（插槽）
- 组件内可以定义 slot，标记为可填充区域（对角线视觉标识）
- 只有组件 origin 中的空 frame 可以转为 slot
- 可以为 slot 指定"suggested components"，引导使用者插入什么组件
- 实例化组件后，拖放或粘贴元素到 slot 区域
- MCP 中用 `descendants` 的 `children` 属性替换 slot 内容

### 快捷键（关键）
| 操作 | 快捷键 |
|------|--------|
| 保存文件 | Cmd+S |
| 创建新文件 | Cmd+N |
| 打开文件 | Cmd+O |
| 切换 AI Chat | Cmd+K |
| 组件转换 | Cmd+Opt+K |
| 组件解绑 | Cmd+Opt+X |
| 应用 flex 布局 | Cmd+Opt+G |
| 直接选择（最深元素）| Cmd+Click |
| 选中父节点 | Shift+Enter |
| 选中子节点 | Enter |
| 画布平移 | Spacebar + Drag |
| 缩放适配 | 0(100%) / 1(适配) / 2(适配选中) |

---

## 更新规范

每次新增 playbook 条目时：

1. **查文档**：用 `/web-access` 打开 Pencil 官方文档，找到对应功能的官方说明
2. **写代码**：在 Pencil MCP 中编写测试代码
3. **截图验证**：用 `get_screenshot` 确认效果
4. **写文档**：验证通过后写入 playbook，附上正确/错误示例

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

### 按钮（文字居中）

**方案 A：容器用 flexbox 居中（推荐，适合所有按钮）**

```javascript
// 父容器用 flexbox，子节点不设 x/y
btn=I(parent, {type: "frame", layout: "horizontal", justifyContent: "center", alignItems: "center", width: 120, height: 44, fill: "#5c00cb", borderRadius: 6})
btnText=I(btn, {type: "text", content: "按钮文字", fontFamily: "Inter", fontSize: 14, fontWeight: "600", fill: "#FFFFFF"})
// 注意：btnText 不设 x/y，flexbox 会自动居中
```

**方案 B：文字节点 textAlign 居中（适合空状态页的标题文字）**

当文字需要在父容器内居中显示时（如空状态标题），用 `textAlign: "center"` + 指定 `width`：

```javascript
card=I(parent, {type: "frame", layout: "vertical", justifyContent: "center", alignItems: "center", width: 560, height: 280, ...})
title=I(card, {type: "text", content: "把视频或音频拖到这里", fontFamily: "Inter", fontSize: 20, fontWeight: "700", fill: "#1a1a1a", textAlign: "center", width: 560})
// textAlign 控制文字在其自身宽度内的对齐，width 决定了文字块的宽度
```

**错误示例**（不要这样做）：
```javascript
btn=I(parent, {type: "frame", layout: "none", width: 120, height: 44, ...})  // ❌
btnText=I(btn, {type: "text", ..., x: 20, y: 12})  // ❌ 文字偏移，不会居中

// ❌ 文字没有指定 width，textAlign 无效
title=I(card, {type: "text", content: "标题", textAlign: "center"})  // ❌ 文字宽度 = 内容宽度，textAlign 不起作用
```

### 多屏原型（Frame 水平排列）

做多个原型界面时，**每个 frame 的 x 坐标必须错开**，否则会叠加在同一个位置 (0,0) 互相遮挡：

```javascript
// 第一屏：x: 0
screen1=I(document, {type: "frame", name: "S1 - Empty State", layout: "none", width: 1440, height: 900, fill: "#FAFAFA", x: 0, y: 0})

// 第二屏：x: 1480（上一屏宽度 + 40px 间隔）
screen2=I(document, {type: "frame", name: "S2 - Editor", layout: "none", width: 1440, height: 900, fill: "#FAFAFA", x: 1480, y: 0})

// 第三屏：x: 2960（上两屏宽度 + 2×40px 间隔）
screen3=I(document, {type: "frame", name: "S3 - Portrait", layout: "none", width: 1440, height: 900, fill: "#FAFAFA", x: 2960, y: 0})
```

**规则**：间距 40px 保证在 Pencil 画布中每张截图之间有足够的空隙，不会互相干扰。公式：`x_N = (N-1) × (1440 + 40)`。

**错误示例**（不要这样做）：
```javascript
// ❌ 所有 frame 默认 x: 0, y: 0，会层层叠加
screen1=I(document, {type: "frame", ..., x: 0, y: 0})
screen2=I(document, {type: "frame", ..., x: 0, y: 0})  // 遮住 screen1
```

---

## 颜色系统

Pencil 支持两种颜色设置方式：

### 方式一：硬编码颜色值（MCP 推荐）

```javascript
frame=I(parent, {fill: "#5c00cb"})  // ✅ 可靠
```

MCP 的 `replace_all_matching_properties` 无法正确生成变量引用（会转义为 `\\$var`），因此**硬编码是最可靠的方式**。

### 方式二：shadcn 主题令牌（仅用于 shadcn 组件）

shadcn 组件通过 `theme` 配置控制颜色：
```javascript
// 改变 shadcn 组件的品牌色
U("KSJ9L", {theme: {"1:Accent": "Violet", "1:Base": "Neutral", "1:Mode": "Light"}})
```

可用 Accent 值：Default/Red/Rose/Orange/Green/Blue/Yellow/Violet
可用 Base 值：Neutral/Gray/Stone/Zinc/Slate

### 项目颜色规范

| 用途 | 色值 |
|------|------|
| 品牌色（主按钮/编辑行高亮） | `#5c00cb` |
| 品牌色深色（警告/时间冲突） | `#3d0088` |
| 编辑行背景 | `#f8f4fc` |
| 页面背景 | `#fafafa` |
| 卡片背景 | `#ffffff` |
| 主文字 | `#1a1a1a` |
| 次要文字/时间码 | `#666666` |
| 占位文字/快捷键 | `#999999` |
| 边框 | `#e5e5e5` |
| 播放器背景 | `#000000` |
| 控制栏背景 | `#0a0a0a` |

---

## 图标系统

Pencil 的 `icon_font` 类型支持 6 种图标字体库，全部内建，无需外部引入。

### 支持的图标库

| 图标库 | `iconFontFamily` 值 | 图标数量 | 风格 | 推荐场景 |
|--------|-------------------|---------|------|---------|
| **Lucide** | `lucide` | 1700+ | 2px 描边、圆角统一 | **首选**，shadcn 默认配套 |
| Feather | `feather` | 300+ | 简洁线条 | 极简风格界面 |
| Material Symbols Outlined | `Material Symbols Outlined` | 2500+ | Google 标准 | 需要 Material 风格时 |
| Material Symbols Rounded | `Material Symbols Rounded` | 2500+ | Google 圆角 | 同上，更柔和 |
| Material Symbols Sharp | `Material Symbols Sharp` | 2500+ | Google 直角 | 同上，更锐利 |
| Phosphor | `phosphor` | 1200+ | 多字重可选 | 需要多种粗细变化时 |

**首选 Lucide**：1700+ 图标覆盖 36 个分类，与 shadcn/ui 组件风格一致，是产品界面最安全的选择。完整图标列表见 [lucide.dev/icons](https://lucide.dev/icons)。

### 常用分类

| 分类 | 典型图标 |
|------|---------|
| 导航 | `chevron-left`, `chevron-right`, `chevron-down`, `arrow-left`, `arrow-right` |
| 操作 | `plus`, `minus`, `search`, `filter`, `sort-asc`, `edit`, `trash-2`, `copy` |
| 状态 | `check`, `x`, `alert-circle`, `alert-triangle`, `info`, `loader-2` |
| 文件 | `file`, `folder`, `upload`, `download`, `paperclip`, `link` |
| 用户 | `user`, `users`, `user-plus`, `mail`, `phone` |
| 设置 | `settings`, `sliders-horizontal`, `bell`, `moon`, `sun` |
| 多媒体 | `play`, `pause`, `volume-2`, `mic`, `image`, `video` |

### 使用语法

```javascript
// 独立图标节点
icon=I(parent, {
  type: "icon_font",
  iconFontFamily: "lucide",
  iconFontName: "settings",
  width: 24,
  height: 24,
  fill: "#1a1a1a"
})

// 图标 + 文字组合（按钮内）
btn=I(parent, {type: "frame", layout: "horizontal", justifyContent: "center", alignItems: "center", gap: 8, ...})
btnIcon=I(btn, {type: "icon_font", iconFontFamily: "lucide", iconFontName: "plus", width: 16, height: 16, fill: "#FFFFFF", x: 0, y: 0})
btnText=I(btn, {type: "text", content: "添加", fontFamily: "Inter", fontSize: 14, fontWeight: "600", fill: "#FFFFFF", x: 0, y: 0})

// 图标按钮（shadcn 组件）
iconBtn=I(parent, {type: "ref", ref: "1:urnwK"})
// 替换图标（用 descendants）
iconBtn=I(parent, {type: "ref", ref: "1:urnwK", descendants: {"iconNodeId": {iconFontName: "settings"}}})

// Sidebar 导航项中的图标
item=I(sidebar, {type: "ref", ref: "1:jBcUh", descendants: {"iconId": {iconFontName: "dashboard"}, "labelId": {content: "Dashboard"}}})
```

### 图标尺寸约定

| 用途 | 尺寸 |
|------|------|
| 按钮内小图标 | 16px |
| 列表项/导航项图标 | 20px |
| 独立操作图标 | 24px |
| 空状态/大图标 | 48px |

### 图标颜色

| 场景 | `fill` 值 |
|------|----------|
| 主文字旁 | `#1a1a1a` |
| 次要文字旁 | `#666666` |
| 主按钮内 | `#FFFFFF` |
| 品牌色按钮内 | `#FFFFFF` |
| 危险操作 | `#ef4444` |
| 导航项选中 | 品牌色（如 `#5c00cb`） |

### 已知坑

1. **图标必须有 width/height** → 不设尺寸 = 不显示或默认 24px
2. **图标用 fill 控制颜色** → 不是 `color` 或 `stroke`
3. **图标名用 kebab-case** → `trash-2` 不是 `trash2`，`chevron-down` 不是 `chevronDown`
4. **组件 descendants 替换图标时只需 `iconFontName`** → 不需要重新声明 `type` 和 `iconFontFamily`

---

## 图形系统（Graphics）

官方文档完整定义了对象的视觉表现，包括 fill/stroke/effects。以下是 playbook 此前未覆盖的部分。

### 多 Fill 叠加

一个对象可以有多个 fill，按数组顺序从下到上层叠：

```javascript
U("nodeId", {
  fill: [
    { type: "color", color: "#5c00cb", blendMode: "normal" },
    { type: "gradient", gradientType: "linear", opacity: 0.3, rotation: 45,
      colors: [{ color: "#ffffff", position: 0 }, { color: "transparent", position: 1 }] }
  ]
})
```

### 渐变（Gradient）

```javascript
// 线性渐变
{
  type: "gradient",
  gradientType: "linear",
  rotation: 135,           // 逆时针角度，0° 向上
  center: { x: 0.5, y: 0.5 },  // 归一化到边界框
  size: { width: 1, height: 1 },
  colors: [
    { color: "#5c00cb", position: 0 },
    { color: "#3d0088", position: 1 }
  ]
}

// 径向渐变
{ type: "gradient", gradientType: "radial", ... }

// 角度渐变
{ type: "gradient", gradientType: "angular", ... }
```

### Image Fill

```javascript
{
  type: "image",
  url: "../../assets/hero.png",  // 相对于 .pen 文件的路径
  mode: "fit"  // "stretch" | "fill" | "fit"
}
```

### Stroke 完整属性

```javascript
{
  stroke: {
    align: "inside",        // "inside" | "center" | "outside"
    thickness: 1,           // 或 { top: 1, right: 2, bottom: 1, left: 2 }
    join: "round",          // "miter" | "bevel" | "round"
    miterAngle: 4,
    cap: "round",           // "none" | "round" | "square"
    dashPattern: [4, 4],    // 虚线
    fill: "#e5e5e5"
  }
}
```

### Effects（效果）

```javascript
// 毛玻璃模糊
{ effect: { type: "background_blur", radius: 8 } }

// 层模糊
{ effect: { type: "blur", radius: 4 } }

// 阴影（最常用）
{
  effect: {
    type: "shadow",
    shadowType: "outer",
    offset: { x: 0, y: 2 },
    spread: 0,
    blur: 8,
    color: "#00000033",  // 8 位 HEX = RGBA
    blendMode: "normal"
  }
}

// 内阴影
{ effect: [{ type: "shadow", shadowType: "inner", ... }, { type: "shadow", shadowType: "outer", ... }] }
```

多个 effect 按数组顺序应用。

### Blend Modes

20 种混合模式：`normal`, `darken`, `multiply`, `linearBurn`, `colorBurn`, `light`, `screen`, `linearDodge`, `colorDodge`, `overlay`, `softLight`, `hardLight`, `difference`, `exclusion`, `hue`, `saturation`, `color`, `luminosity`。日常原型几乎用不到，做特殊视觉效果时查阅。

---

## 文字进阶（Text）

### textGrowth（文字容器增长模式）

控制文字如何换行和容器如何增长。**不设 textGrowth 时 width/height 会被忽略**：

| 模式 | 行为 | 场景 |
|------|------|------|
| `auto` | 文字不换行，容器自动扩大 | 单行标题、按钮文字 |
| `fixed-width` | 固定宽度，文字自动换行，高度自动增长 | 正文、描述文字 |
| `fixed-width-height` | 宽高都固定，文字换行，超出可能溢出 | 卡片描述限制行数 |

```javascript
// 单行标题
{ type: "text", content: "标题", textGrowth: "auto", fontSize: 24, fontWeight: "700" }

// 自动换行的正文
{ type: "text", content: "这是一段很长的描述文字...", textGrowth: "fixed-width", width: 300, fontSize: 14 }

// 固定区域的描述
{ type: "text", content: "...", textGrowth: "fixed-width-height", width: 300, height: 60, fontSize: 14 }
```

### 其他文字属性

```javascript
{
  lineHeight: 1.5,        // 行高倍数，不设则用字体内置行高
  letterSpacing: 0.5,     // 字间距
  textAlign: "center",    // "left" | "center" | "right" | "justify"
  textAlignVertical: "middle",  // "top" | "middle" | "bottom"
  underline: true,
  strikethrough: true,
  fontStyle: "italic"
}
```

Rich text 支持 `content` 为 `TextStyle[]` 数组，同一段文字不同样式：

```javascript
{
  type: "text",
  content: [
    { content: "粗体", fontWeight: "700" },
    { content: "普通文字", fontWeight: "400" }
  ]
}
```

---

## 主题系统（Theme）进阶

### 解析规则

- 变量有多个主题值时，**最后一个满足当前主题配置的胜出**
- 每个主题轴的**第一个值是默认值**（如 `themes: { mode: ["light", "dark"] }` 默认 light）

```javascript
{
  "variables": {
    "bg": {
      "type": "color",
      "value": [
        { "value": "#FFFFFF", "theme": { "mode": "light" } },
        { "value": "#1a1a1a", "theme": { "mode": "dark" } }
      ]
    }
  },
  "themes": {
    "mode": ["light", "dark"]   // light 是默认值
  }
}
```

### 实体级主题覆盖

任意 Entity 可设 `theme` 属性，其下所有子节点继承该主题：

```javascript
// 整个页面是 light，但某一块用 dark 主题
{
  type: "frame",
  theme: { "mode": "dark" },  // 此 frame 下的所有节点使用 dark 主题
  fill: "$bg"                // 解析为 #1a1a1a
}
```

多轴主题：

```javascript
{ theme: { "mode": "dark", "spacing": "condensed" } }
```

shadcn 组件通过 `theme` 配置控制品牌色：

```javascript
U("nodeId", { theme: { "1:Accent": "Violet", "1:Base": "Neutral", "1:Mode": "Light" } })
```

---

## Entity 通用属性

所有 Entity（frame/rectangle/text 等）共享以下属性，此前 playbook 未完整列出：

| 属性 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `id` | string | 自动生成 | 文档内唯一，不可含 `/` |
| `name` | string | — | 显示名称，用于标识 |
| `reusable` | boolean | false | 标记为可复用组件 |
| `enabled` | bool/var | true | 是否启用 |
| `opacity` | num/var | 1 | 透明度 |
| `rotation` | num/var | 0 | 旋转角度（逆时针） |
| `flipX` | bool/var | false | 水平翻转 |
| `flipY` | bool/var | false | 垂直翻转 |
| `theme` | object | — | 主题配置 |
| `context` | string | — | 实体上下文 |
| `metadata` | object | — | 元数据 |
| `layoutPosition` | string | "auto" | `"auto"`（参与 flex 布局）或 `"absolute"`（绝对定位） |
| `clip` | bool/var | false | frame 裁剪溢出内容 |

### layoutPosition

```javascript
// 在 flex 容器中，某个子节点不参与布局，手动定位
{
  type: "frame",
  layout: "vertical",
  children: [
    { type: "text", content: "正常流" },
    { type: "text", content: "浮动覆盖", layoutPosition: "absolute", x: 100, y: 50 }
  ]
}
```

### clip

```javascript
// 溢出 frame 的内容被裁剪
{ type: "frame", width: 200, height: 100, clip: true, children: [...] }
```

---

## 其他节点类型

Playbook 主要使用 frame/rectangle/text/ref/icon_font，以下是其他类型的简要说明：

| 类型 | 用途 | 关键属性 |
|------|------|---------|
| `ellipse` | 椭圆/圆/环 | `innerRadius`（0=实心, 1=环）、`startAngle`、`sweepAngle` |
| `line` | 线条 | width/height 决定线长和方向 |
| `path` | SVG 路径 | `geometry`（SVG path 字符串）、`fillRule`（nonzero/evenodd） |
| `polygon` | 正多边形 | `polygonCount`（1=三角形, 2=方形...）、`cornerRadius` |
| `group` | 无背景的容器 | 类似 frame 但无 fill/border，只做分组 |
| `note` | 设计备注 | `content`，不参与渲染，仅设计阶段 |

Group vs Frame：Group 没有视觉表现（无 fill/stroke/cornerRadius），仅用于逻辑分组。Frame 是矩形且可有子节点。

---

## 外部引用（Document.imports）

```json
{
  "imports": {
    "shadcn": "../design/shadcn.lib.pen"
  }
}
```

- key 是短别名（如 `shadcn`），value 是相对路径
- 导入的 .pen 文件中所有 reusable 组件和 variables 在当前文件可用
- 实例化时用 `ref: "shadcn/1:VSnC2"` 或直接用 ID `"1:VSnC2"`（ID 文档内唯一即可）

Pencil 桌面通过 `pencil:shadcn.lib.pen` 协议引入，MCP 中会自动处理。手写 .pen 文件时用相对路径。

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

### 标准工作流（增量式）

```
1. batch_get 读取当前状态 → 2. 确认目标节点/属性 → 3. U()/I() 精确修改 → 4. get_screenshot 验证 → 5. 手动保存
```

- 每次只改一个区域（左栏/右栏/顶部栏），不要一次性改全屏
- 每批操作 ≤ 25 个，避免绑定变量过期
- 每批操作后立刻 `get_screenshot` 验证，发现问题立刻修

### 正确：增量修改

```javascript
// 读当前状态，找到要改的节点 ID
batch_get({nodeIds: ["existingNodeId"]})
// 用 U() 精确更新
U("existingNodeId", {fill: "#5c00cb"})
// 验证
get_screenshot({nodeId: "existingNodeId"})
```

### 错误：全删重建（操作不熟练的下策）

```javascript
// ❌ 不要这样做
D("WGbpF")  // 删除整个屏
// 重新创建所有内容...
```

### 需要全量重建的场景

只有在以下情况才全删重建：
- 文档被 `replace_all_matching_properties` 等破坏性操作彻底损坏
- 需要改变整体布局结构（横屏 → 竖屏）
- 首次创建

1. **先建框架**：用 `I(document, ...)` 创建主框架
2. **逐层填充**：在框架内用 `I("parentId", ...)` 添加内容
3. **每次操作后验证**：用 `get_screenshot` 看效果，用 `snapshot_layout` 查问题
4. **组件优先**：优先用 `ref` 引用 shadcn 组件，不要自己造轮子
5. **颜色用硬编码值**：MCP 变量引用有 bug，用 `#5c00cb` 等硬编码更安全
6. **小步迭代**：每批 25 个操作以内，不要一次性塞太多
7. **字体简洁**：用 `"Inter"`，不要用复杂字体栈

---

## 已知坑

1. **C (Copy) 操作有 API bug** → 用 `ref` 替代
2. **数字开头的 ID 必须加引号** → `I("1GmYb", {...})` 不能写成 `I(1GmYb, {...})`
3. **binding 不跨批次** → 每次 `batch_design` 都是新的作用域
4. **flexbox 下 x/y 无效** → 布局引擎会忽略，靠 gap/padding 控制间距
5. **fontFamily 'system-ui' 单独使用报错** → 用 `"Inter"` 或完整栈 `"Inter, system-ui, sans-serif"`
6. **R (Replace) 会丢失子节点** → 替换后需重新插入子内容
7. **按钮文字居中必须用 flexbox** → `layout: "none"` + 绝对定位 `x/y` 无法实现文字居中。所有按钮/图标按钮必须用 `layout: "horizontal"` + `justifyContent: "center"` + `alignItems: "center"`，子节点 x/y 归零。**这是最常见的视觉错误**。空状态页标题文字用 `textAlign: "center"` + 指定 `width`。
8. **多屏原型 Frame 默认叠加在 (0,0)** → 每个新 frame 的 x 坐标必须递增错开。公式：`x_N = (N-1) × (1440 + 40)`。不设 x 坐标 = 所有屏叠在一起看不到
9. **replace_all_matching_properties 是破坏性操作** → 会替换子树中 ALL 匹配属性，连锁污染。不要用。用 `U("nodeId", {...})` 精确更新
10. **变量引用在 replace_all_matching_properties 中会转义** → `$1:--foreground` 变成 `\\$1:--foreground`，渲染引擎不识别
11. **shadcn 库变量只读** → `1:--primary` 等是导入变量，`set_variables` 无法修改。要改品牌色只能改 shadcn frame 的 theme 配置
12. **Pencil 无自动保存** → 每次 `batch_design` 后必须手动 Cmd+S

---

## Pencil MCP `get_guidelines()` 设计指南

Pencil 提供 8 个主题的设计指南，通过 `get_guidelines(topic)` 调用。这些不是操作语法，而是**设计决策框架**——指导 AI 在做设计时遵循行业最佳实践。

### 可用主题

| 主题 | 用途 | 何时调用 |
|------|------|---------|
| `web-app` | 功能型产品 UI 的 16 条通用原则 | 做任何产品界面设计前 |
| `landing-page` | 营销落地页设计系统 | 做官网、营销页时 |
| `mobile-app` | 移动端屏幕组成规范 | 做移动端原型时 |
| `design-system` | 用已有组件组装屏幕的模式 | 用 shadcn 组件搭建页面时（**最常用**） |
| `slides` | 演示文稿 20 种布局模板 | 做 PPT/Keynote 时 |
| `code` | 从 .pen 到 React 的转换流程 | 原型确定后，开始写代码时 |
| `table` | 手写表格结构规则 | 没有预设表格组件时 |
| `tailwind` | Tailwind v4 实现指南 | 写代码阶段 |

### web-app 核心原则（摘录）

做产品界面时必须遵守的高层原则：

| 编号 | 原则 | 含义 |
|------|------|------|
| W1 | Purpose First | 每屏一个核心问题、一个主要操作 |
| W2 | Dominant Region | 每屏一个视觉焦点，避免等权重的布局 |
| W3 | Progressive Disclosure | 先展示核心，高级操作按需展开 |
| W4 | Recognition Over Recall | 减少记忆负担，操作在需要时才出现 |
| W5 | System Status | 每个数据区域定义：Loading / Empty / Error / Success / Permission 五种状态 |
| W6 | Action Hierarchy | 一个 primary action，次级操作降视觉权重，破坏性操作独立区分 |
| W7 | Spatial Logic | 一个主导轴向，优先两分区，用留白分隔而非装饰线 |
| W8 | Constraint Over Decoration | 不服务于导航/理解/决策/操作的元素不应存在 |
| W9 | Entity Integrity | 实体（用户/记录/文档）必须显示名称、状态、关键元数据、明确操作 |
| W10 | Responsiveness | 层级必须在所有断点存活，移动端单列为主 |

### design-system 组件组装模式（关键）

#### Slot 使用模式

```javascript
// 插入组件后，用 slot 路径插入内容
sidebar=I(page, {type: "ref", ref: "1:PV1ln", height: "fill_container"})
item=I(sidebar+"/contentSlotId", {type: "ref", ref: "1:qCCo8", descendants: {"iconId": {iconFontName: "dashboard"}, "labelId": {content: "Dashboard"}}})

// 不需要的 slot 标记为 disabled
U(card+"/actionsSlotId", {enabled: false})
```

#### Card 三段式结构

Card 组件通常有 header / content / actions 三个 slot：

```javascript
card=I(container, {type: "ref", ref: "1:pcGlv", width: 480})
// Header: 用 R() 替换为自定义内容
newHeader=R(card+"/headerSlotId", {type: "frame", layout: "vertical", gap: 4, padding: 24, width: "fill_container", children: [
  {type: "text", content: "标题", fontSize: 18, fontWeight: "600"},
  {type: "text", content: "描述文字", fontSize: 14, fill: "#666666"}
]})
// Content: 在 slot 内插入
input=I(card+"/contentSlotId", {type: "ref", ref: "1:1415a", width: "fill_container"})
// Actions: 右对齐按钮组
U(card+"/actionsSlotId", {gap: 12, justifyContent: "end", padding: 24})
```

#### 页面布局模板

```javascript
// A: Sidebar + Content（Dashboard）
screen=I(document, {type: "frame", name: "Dashboard", layout: "horizontal", width: 1440, height: "fit_content(900)"})
sidebar=I(screen, {type: "ref", ref: "1:PV1ln", height: "fill_container"})
main=I(screen, {type: "frame", layout: "vertical", width: "fill_container", height: "fill_container(900)", padding: 32, gap: 24})

// B: Header + Content
screen=I(document, {type: "frame", layout: "vertical", width: 1200, height: "fit_content(800)"})
header=I(screen, {type: "frame", layout: "horizontal", width: "fill_container", height: 64, padding: [0, 24], alignItems: "center", justifyContent: "space_between"})
content=I(screen, {type: "frame", layout: "vertical", width: "fill_container", height: "fit_content(736)", padding: 32, gap: 24})

// C: Two-Column
cols=I(content, {type: "frame", layout: "horizontal", width: "fill_container", height: "fill_container(900)", gap: 24})
mainCol=I(cols, {type: "frame", layout: "vertical", width: "fill_container", gap: 24})
sideCol=I(cols, {type: "frame", layout: "vertical", width: 360, gap: 24})

// D: Card Grid
grid=I(container, {type: "frame", layout: "horizontal", width: "fill_container", gap: 16})
card=I(grid, {type: "ref", ref: "1:pcGlv", width: "fill_container"})
```

#### 按钮层级

| 优先级 | 变体 | 用途 |
|--------|------|------|
| 1 | Primary/Default | 主操作（Save, Submit, Create） |
| 2 | Secondary | 替代操作 |
| 3 | Outline | 三级操作（Cancel, Back） |
| 4 | Ghost | 内联操作 |
| 5 | Destructive | 删除、危险操作 |

**对齐约定**：
- Card/Modal 中的按钮：右对齐（`justifyContent: "end"`）
- 表单提交按钮：右对齐
- Destructive + Cancel：Cancel 在左，Destructive 在右

#### 间距参考

| 场景 | Gap | Padding |
|------|-----|---------|
| 屏幕区块 | 24-32 | — |
| 卡片网格 | 16-24 | — |
| 表单字段（纵向） | 16 | — |
| 按钮组 | 12 | — |
| 卡片内部 | — | 24 |
| 按钮内部 | — | [10, 16] |
| 页面内容区 | — | 32 |

#### 图标使用

Lucide 为首选（与 shadcn 风格一致）。完整图标库、尺寸约定、颜色规范见上方"图标系统"章节。

```javascript
// Lucide 图标（推荐）
icon=I(container, {type: "icon_font", iconFontFamily: "lucide", iconFontName: "settings", width: 24, height: 24})

// 组件内替换图标
descendants: {"iconNodeId": {iconFontName: "search"}}
```

### code 指南：从 .pen 到 React 的流程

```
Step 1: 组件分析 → 读取 .pen 中所有 ref 实例，记录每个组件出现次数和属性覆盖
Step 2: 逐个提取 → batch_get 获取完整组件树，一个接一个处理（不要批量）
Step 3: React 创建 → .tsx + TypeScript interface + Tailwind 类
Step 4: 验证 → get_screenshot 对比设计稿与渲染结果，像素级对齐
Step 5: 集成 → 组装到页面框架，验证 fill_container → flex-1 转换
```

**关键转换规则**：
- `fill_container` → `flex-1`（flex 容器内）或 `w-full` / `h-full`
- `fit_content` → `w-fit` / `h-fit`
- 颜色 → `bg-[var(--color-name)]`，不硬编码 hex
- 字体 → 在 `@layer base` 定义 `.font-primary` 类，不用 `font-[var(--font-primary)]`
- 多个 `fill_container` 子元素 → 每个都需要 `flex-1`

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
