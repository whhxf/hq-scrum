---
name: sop-rules
description: hq-scrum SOP 流程规则。使用此 SOP 的项目必须遵守。
type: feedback
---

**hq-scrum SOP 规则 — 不可跳过：**

- Phase 0（PRD）→ Phase 1（Sprint 0 Pencil 原型）→ Phase 2（架构）→ Phase 3（Specs）→ Phase 4（实现）→ Phase 5（Show & Tell）
- 不允许跳过 Phase 直接写代码
- Sprint 0 原型必须通过 Pencil 生成真实 UI 截图，不可用 ASCII 代替
- Story 必须包含 Design Spec（状态矩阵）才能进入 ready-for-dev
- 每个 Story 完成标准：逻辑测试 + 人眼视觉确认
- Sprint 结束必须 Show & Tell，不是自嗨演示
- 回退是流程的一部分，发现方向错直接回退，不讨论沉没成本

---

## 设计约束（Phase 1 起生效）

以下约束从 Sprint 0 原型开始适用，所有 Pencil 设计和实现阶段代码必须遵守。

### 间距（S1.x）

| 规则 | 约束 |
|------|------|
| S1.1 | 所有间距只取 4px 的整数倍：4, 8, 12, 16, 20, 24, 32, 48, 64 |
| S1.2 | 相关元素间距 ≤ 12px：图标和文字、标题和副标题、标签和输入框 |
| S1.3 | 不相关元素间距 ≥ 24px：两个卡片之间、功能区域之间 |
| S1.6 | 同层级多个元素间距必须一致，不混用 12/16/20 |

### 排版（H2.x）

| 规则 | 约束 |
|------|------|
| H2.1 | 每屏只有一个视觉焦点（主按钮 > 次按钮 > 其他所有） |
| H2.2 | 字号只取阶梯值：11, 12, 13, 14, 16, 18, 20, 24, 28, 36 |
| H2.3 | 字重只使用 2 种：normal(400) 和 bold(600-700) |
| H2.4 | 品牌色只用于 primary action |
| H2.5 | 次级操作用 outline 或纯文字，不用灰色填充 |

| 用途 | 字号 | 字重 |
|------|------|------|
| 页面大标题 | 28-36px | 700 |
| 区块标题 | 20-24px | 600-700 |
| 卡片标题 | 16-18px | 600 |
| 正文内容 | 14-16px | 400 |
| 辅助信息 | 12-13px | 400 |
| 时间戳/标签 | 11-12px | 400 |

### 形状（R3.x）

| 规则 | 约束 |
|------|------|
| R3.1 | 圆角只有 5 种：6px(按钮)、8px(卡片)、12px(模态)、16px(徽章)、9999px(头像) |
| R3.7 | 边框粗细统一 1px |
| R3.8 | 有阴影的元素不需要边框，有边框的元素不需要阴影 |

### 色彩（C4.x）

| 规则 | 约束 |
|------|------|
| C4.1 | 整个产品只有一个品牌色 |
| C4.3 | 文字只用 3 级灰度 |
| C4.4 | 背景只用 2 种：页面背景 + 卡片背景 |
| C4.7 | 任何时刻屏幕上最多出现 2 种有彩色 |
| C4.8 | 禁止使用纯黑 `#000000` 作为文字色 |

### 克制（C0.x）

| 规则 | 约束 |
|------|------|
| C0.1 | 一个页面只有一种 primary action |
| C0.2 | 每屏只让用户思考一件事 |
| C0.3 | 如果一个元素可以被忽略而不影响核心流程，删掉它 |

---

## Pencil 原型速查（Phase 1 Sprint 0）

### 核心操作

| 操作 | 语法 | 注意 |
|------|------|------|
| Insert | `I(parent, props)` | parent 用字符串 ID |
| Update | `U(nodeId, props)` | 跨批次用节点 ID，binding 不跨批 |
| Delete | `D(nodeId)` | 删除节点及其子树 |
| Replace | `R(nodeId, props)` | 会丢子节点，需重新插入 |
| Copy | 有 bug，用 `ref` 替代 | `I(parent, {type: "ref", ref: "sourceId"})` |

### 关键规则

- 节点 ID 以数字开头必须加引号：`I("1GmYb", {...})`
- flexbox 布局下子节点 x/y 无效，靠 gap/padding 控制间距
- 按钮文字居中必须用 flexbox：`layout: "horizontal"` + `justifyContent: "center"`
- 多屏原型 Frame 的 x 坐标必须错开：`x_N = (N-1) × (1480)`
- **不要用** `replace_all_matching_properties`：破坏性操作，连锁污染
- **不要**全删重建：用 `batch_get` 读状态 → `U()` 精确更新 → `I()` 增量添加
- 每次 `batch_design` 后必须 Cmd+S 手动保存

### 常用 shadcn 组件 ID

| 组件 | ID |
|------|-----|
| Sidebar | `1:PV1ln` |
| Button/Default | `1:VSnC2` |
| Button/Secondary | `1:e8v1X` |
| Button/Outline | `1:C10zH` |
| Button/Destructive | `1:YKnjc` |
| Icon Button | `1:urnwK` |
| Card | `1:pcGlv` |
| Badge/Default | `1:UjXug` |
| Input/Default | `1:fEUdI` |
| Input Group | `1:1415a` |
| Checkbox | `1:ovuDP` |
| Switch | `1:c8fiq` |
| Tabs | `1:PbofX` |
| Dialog | `1:OtykB` |
| Alert/Default | `1:QyzNg` |
| Sidebar Item/Default | `1:jBcUh` |
| Sidebar Item/Active | `1:qCCo8` |

### 图标（Lucide）

Pencil 支持 6 种图标字体，**首选 Lucide**（1700+ 图标，shadcn 默认配套）。

```javascript
// 独立图标
icon=I(parent, {type: "icon_font", iconFontFamily: "lucide", iconFontName: "settings", width: 24, height: 24, fill: "#1a1a1a"})

// 组件内替换图标
descendants: {"iconNodeId": {iconFontName: "search"}}
```

| 用途 | 尺寸 |
|------|------|
| 按钮内小图标 | 16px |
| 列表项/导航项 | 20px |
| 独立操作图标 | 24px |
| 空状态/大图标 | 48px |

图标名用 kebab-case（`trash-2` 不是 `trash2`），必须有 width/height。完整图标列表见 [lucide.dev/icons](https://lucide.dev/icons)。

其他支持的图标库：Feather、Material Symbols (Outlined/Rounded/Sharp)、Phosphor。

---

## 设计技能增强（Phase 4/5）

项目初始化时安装 Impeccable Design Skill，在 Phase 4/5 阶段用于自动设计评审和视觉抛光：

```bash
npx skills add pbakaus/impeccable
```

安装后运行 `/impeccable teach` 建立项目设计上下文（品牌、受众、视觉参考），生成 `.impeccable.md`。

| 阶段 | 命令 | 作用 |
|------|------|------|
| Phase 4 实现后 | `/polish [页面描述]` | 对齐、间距、排版、色彩综合优化 |
| Phase 4 实现后 | `/typeset [页面描述]` | 字体层级、字重、字距、可读性 |
| Phase 4 实现后 | `/colorize [页面描述]` | 为灰度过度的界面增加战略色彩 |
| Phase 5 验证前 | `/critique [页面描述]` | UX 评审：层级、信息架构、认知负荷 |
| Phase 5 验证前 | `/audit [页面描述]` | 技术评审：无障碍、性能、响应式 |

Impeccable 是针对性修改，不是重写。每次只调一个维度，审 diff 后再决定下一步。

---

## Pencil 与 Impeccable 的分工

| 阶段 | 工具 | 输入 | 输出 |
|------|------|------|------|
| Phase 1 Sprint 0 原型 | Pencil MCP | PRD 文字/ASCII + 设计约束 | `.pen` 文件 + 真实 UI 截图 |
| Phase 4 实现后抛光 | Impeccable | 已有 HTML/组件代码 + `.impeccable.md` | 代码 diff（针对性修改） |
| Phase 5 验证评审 | Impeccable | 已有页面 | 评分报告 + 技术检查 |

Pencil 定方向（原型验证），Impeccable 做收尾（代码抛光）。
