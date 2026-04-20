---
name: sop-rules
description: hq-scrum SOP 流程规则。使用此 SOP 的项目必须遵守。
type: feedback
---

**初始化**：当用户说「hq-scrum 在 XXX 路径」或「初始化 hq-scrum」：
1. 复制该路径下 `scaffolds/project/` 全部内容到当前项目根目录（`cp -r`）
2. 复制完成后，从 Phase 0 开始

---

## Phase 执行引擎

每个 Phase 的完成标准达到后，才能进入下一个。用户确认后推进。

### Phase 0 — 需求（填充 PRD.md）
1. 读取用户描述的产品想法
2. 按 PRD.md 模板提问补全：目标用户、核心路径（≤3 条）、非目标、约束
3. 填写 PRD.md
4. **等待用户确认 PRD 内容**，确认后进入 Phase 1

### Phase 1 — Sprint 0 原型
1. **执行 `/shape [产品名]`** — UX 访谈，产出 Design Brief
2. 确认 Pencil MCP 可用（调用 `get_editor_state` 不报错）
3. **先保存**：`/new` → Cmd+S 保存为 `specs/sprint-0/prototype.pen`
4. 浏览 shadcn 组件库：`batch_get` 查看 `design/shadcn.lib.pen`
5. 基于 PRD + Design Brief，用 `ref` 引用 shadcn 组件生成原型
6. 原型完成后执行 `/critique [页面描述]` — UX 评审
7. **等待用户确认原型走查通过**，确认后进入 Phase 2

### Phase 2 — 架构（填写 ARCHITECTURE.md）
1. 基于 PRD 填写技术栈表格（每项必须写为什么选）
2. 确认目录结构、状态管理方案、样式方案、测试策略
3. **等待用户确认架构**，确认后进入 Phase 3

### Phase 3 — Story Specs
1. 将核心路径拆分为 Sprint Story
2. 每个 Story 包含 Design Spec（状态矩阵：Loading/Empty/Error/Success/Permission）
3. **Planning Audit Gate：等待人审合同边界**，通过后进入 Phase 4

### Phase 4 — 实现
1. 逐个 Story 实现
2. 可用 Impeccable 抛光：`/polish` `/typeset` `/colorize` `/layout`
3. 每个 Story 完成标准：逻辑测试通过 + 人眼视觉确认
4. Sprint 结束后进入 Phase 5

### Phase 5 — Show & Tell
1. 执行 `/audit` 技术检查
2. 真实用户验证核心任务，不是自嗨演示
3. 回退是流程的一部分，发现方向错直接回退

---

**知识来源**：本文件（CLAUDE.md）内联了设计约束和 Pencil 操作速查表，是日常工作的主要参考。`docs/pencil-playbook.md` 和 `docs/design-constraints.md` 是完整参考文档，需要深入查某个细节时翻阅。

**设计组件库**：`design/shadcn.lib.pen` 是 shadcn/ui 的完整组件库（.pen 格式）。在 Pencil 中通过 import 引入后，可用 `ref` 引用所有组件（Button、Card、Dialog、Sidebar 等）。AI 可通过 `batch_get` 浏览组件结构，`get_screenshot` 查看真实渲染效果。

**hq-scrum SOP 规则 — 不可跳过：**

- Phase 0（PRD）→ Phase 1（Sprint 0 Pencil 原型）→ Phase 2（架构）→ Phase 3（Specs）→ Phase 4（实现）→ Phase 5（Show & Tell）
- 不允许跳过 Phase 直接写代码
- Sprint 0 原型必须通过 Pencil 生成真实 UI 截图，不可用 ASCII 代替
- Story 必须包含 Design Spec（状态矩阵）才能进入 ready-for-dev
- 每个 Story 完成标准：逻辑测试 + 人眼视觉确认
- Sprint 结束必须 Show & Tell，不是自嗨演示
- 回退是流程的一部分，发现方向错直接回退，不讨论沉没成本

---

## 工具前置条件

进入 Phase 1 之前必须确认以下工具可用：

### Pencil MCP

- **Pencil 桌面应用**（[pencil.dev](https://pencil.dev)）必须正在运行
- Claude Code 必须通过 MCP 连接到 Pencil（MCP Server 配置在 settings.json 中）
- 验证方法：在 Claude Code 中调用 Pencil MCP 工具，如 `get_editor_state`，不报错即为连通
- **如果 Pencil 不可用**：用纸面回答 6 个 UX 发现问题（见 WORKFLOW.md Phase 1 步骤 1），然后继续 Phase 2。但 Sprint 0 必须在 Phase 4 实现前用 Pencil 补上

### Impeccable Design Skill（必须）

- 初始化时已内置于 `.agents/skills/impeccable/`
- Phase 1 第一步自动执行 `/shape`，原型完成后自动执行 `/critique`
- Phase 4/5 按需使用 `/polish` `/typeset` `/colorize` `/layout` `/audit`
- 每次只调一个维度，审 diff 后再决定下一步

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

### 组件优先原则（必须遵守）

**所有 Pencil 设计必须优先使用 `design/shadcn.lib.pen` 中的组件。**

- 开始设计前，先用 `batch_get` 浏览 shadcn 组件库，找到匹配的组件
- 用 `ref` 引用 shadcn 组件（Button、Card、Dialog、Sidebar、Input、Tabs、Alert、Badge 等），不要自己从零画
- 只在 shadcn 没有对应组件时，才手动创建自定义组件
- 用 `get_screenshot` 查看组件渲染效果，确保视觉一致

### 先保存再操作

**第一步永远是保存，不保存不做任何操作。**

`/new` 创建的临时文档每次重新序列化时会重新分配节点 ID。如果 `batch_design` 操作发生回滚，临时文档的 ID 会变化，之前记录的 ID 全部失效。**必须先 Cmd+S 保存为 `.pen` 文件，再开始 `batch_design`**。保存到磁盘后 ID 就固定了。

**文件路径约定**：
- 所有 Sprint 原型：`specs/sprint-N/prototype.pen`（N 为 Sprint 编号，0、1、2...）
- 设计库/组件库：`design/components/xxx.lib.pen`

所有 `.pen` 文件保存在项目根目录下的对应路径，不要散落在任意位置。

### 核心操作

| 操作 | 语法 | 注意 |
|------|------|------|
| Insert | `I(parent, props)` | parent 用字符串 ID |
| Update | `U(nodeId, props)` | 跨批次用节点 ID，binding 不跨批 |
| Delete | `D(nodeId)` | 删除节点及其子树 |
| Replace | `R(nodeId, props)` | 会丢子节点，需重新插入 |
| Copy | 有 bug，用 `ref` 替代 | `I(parent, {type: "ref", ref: "sourceId"})` |

### 关键规则

- **先保存再操作**：`/new` 临时文档不保证 ID 稳定性，Cmd+S 保存为 `.pen` 后才能开始 `batch_design`
- **新建 Frame 必须错开 x 坐标**：每个新 frame 的 `x` 必须等于前一个 frame 的 `x + 宽度 + 40px`。不设置 = 所有 frame 叠加在 `(0,0)` 互相遮挡看不到。公式：`x_N = (N-1) × (1440 + 40)`
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

## Pencil 与 Impeccable 的分工

| 阶段 | 工具 | 输入 | 输出 |
|------|------|------|------|
| Phase 1 Sprint 0 原型 | Pencil MCP | PRD 文字/ASCII + 设计约束 | `.pen` 文件 + 真实 UI 截图 |
| Phase 4 实现后抛光 | Impeccable | 已有 HTML/组件代码 + `.impeccable.md` | 代码 diff（针对性修改） |
| Phase 5 验证评审 | Impeccable | 已有页面 | 评分报告 + 技术检查 |

Pencil 定方向（原型验证），Impeccable 做收尾（代码抛光）。
