# hq-scrum 工作流程

## 核心原则

**Spec = 合同边界**。Design Spec 不是写给自己看的备忘录，是人和 AI 之间的合同：
- 合同内（实现细节）：AI 自由发挥
- 合同外（边界约定）：人审边界决策，不审逐行实现

> "Boundaries are not overhead. They are what lets you move freely inside while protecting the outside."

**Spec 的粒度**：一个 Story 应该能在数小时或数天内完成。太大就拆。

**零安装，对话式驱动**。不装 CLI，不配 61 个 commands。把 scaffold 复制到项目，打开 Claude Code 说「按流程开始」，AI 全程驱动。人在关键 gate 做审批，不逐行写文档。

## 总览

```
Phase -1: 品牌   → .impeccable.md（视觉宪法）
Phase 0:  需求   → PRD.md + DESIGN_BRIEF.md（含 UX 发现 + 竞品基准）
Phase 1:  验证   → Sprint 0 原型（Pencil GUI 设计，真实 UI 截图）
          ↓ UX 评审 + Visual Diff Check Gate
Phase 2:  架构   → ARCHITECTURE.md + tokens.css
Phase 3:  规划   → specs/sprint-N/*.md（Story 含 Design Spec）
          ↓ Planning Audit Gate（人审合同边界）
Phase 4:  实现   → src/ + tests/（Constraints 作为硬性约束）
Phase 5:  验证   → Show & Tell（真实用户验证核心任务）
```

每一阶段的产出是下一阶段的输入。不允许跳过阶段。

---

## Phase -1: 品牌（视觉方向）

**触发**：项目需要品牌调性（营销页、官网、C 端产品）。内部工具/后台可跳过。
**目标**：明确「这个产品长什么样」——品牌色、视觉调性、情感氛围
**产出**：`.impeccable.md`

### 步骤

1. 执行 `/impeccable teach`
2. 回答品牌定义问题：
   - 品牌名是什么？
   - 受众是谁？（B 端决策者、C 端消费者、内部员工？）
   - 视觉参考：哪些品牌/产品的视觉方向是你认可的？
   - 情感调性：高端？亲和？专业？极简？活力？
3. 生成 `.impeccable.md`，包含品牌色、字体偏好、视觉关键词
4. 用户确认品牌方向

### 为什么需要

没有品牌方向，原型就是灰色的卡片堆——功能正确但没有灵魂。`.impeccable.md` 是后续所有设计和代码的「视觉宪法」，Phase 1 的 Pencil 设计和 Phase 4 的代码实现都从中读取方向。

---

## Phase 0: 需求（PRD + Design Brief）

**触发**：用户描述了一个产品想法
**目标**：明确做什么、为谁做、不做什么，并产出设计指令书
**产出**：`PRD.md` + `DESIGN_BRIEF.md`

### 步骤

#### 1. UX 发现（`/shape`）

PRD 定义了"做什么"，但没有定义"怎么做感觉对"。在动手写 PRD 之前，先跑一次 `/shape [产品名]`。

`/shape` 会发起一场 5-10 个问题的结构化 UX 访谈：
- 用户是谁？在什么场景下使用？什么心态？
- 显示什么数据？最小/典型/最大量级？
- 空状态、错误状态、加载状态分别是什么？
- 这个页面应该让用户感觉什么？（高效/安心/有趣/高端？）
- 有什么绝对不应该做的？

访谈结束后会产出一份 **Design Brief**，不是代码，是"思考"——指导后续原型和实现的决策依据。

#### 2. 填写 PRD

基于 UX 洞察，生成 `PRD.md`，必须包含：
- 目标用户
- 核心用户路径（不超过 3 条）
- 非目标（明确不做什么）
- 约束（技术、时间、平台）

#### 3. 竞品基准

找 3-5 个同类型真实产品，截图 + 分析：
- 他们的信息架构是什么？
- 视觉调性是什么感觉？
- 有什么值得借鉴的交互模式？

**为什么需要竞品基准**：不看竞品就设计，等于闭门造车。竞品基准不是为了抄袭，而是为了知道「行业标准长什么样」，避免做出远低于行业水平的设计。

#### 4. 生成 Design Brief

汇总以上内容，生成 `DESIGN_BRIEF.md`，作为 Phase 1 Pencil AI agent 的上下文输入。包含：
- 产品概述（来自 PRD）
- 用户画像与场景（来自 `/shape`）
- 品牌方向（来自 `.impeccable.md`）
- 页面类型声明：Landing Page（营销）还是 Tool/Editor（功能型）
- 竞品参考（截图 + 关键设计模式）
- 核心用户路径（需要在原型中走通的流程）

### 验证

- 核心用户路径能否在纸面上走通？不能走通说明需求没想清楚，回到步骤 1
- 竞品基准是否覆盖了直接竞品和间接竞品？
- DESIGN_BRIEF 能否让一个不了解产品的人快速理解「做什么 + 长什么样」？

---

## Phase 1: 验证（Sprint 0 原型）

**触发**：PRD + DESIGN_BRIEF 已确认
**目标**：在写代码前验证核心交互路径和视觉体验
**产出**：`specs/sprint-0/0.1-prototype.md` + `specs/sprint-0/prototype.pen`

### 工具交接说明

本阶段需要在 **Claude Code** 和 **Pencil GUI** 之间切换。Claude Code 会在每个阶段结束时明确告诉你下一步做什么，不需要你记流程。

**Phase 1 交互流程**：
```
Claude Code:  "Phase 0 完成。现在打开 Pencil 桌面应用..."
              （给出 Pencil 操作指引 + DESIGN_BRIEF.md 内容）
              "完成后保存为 specs/sprint-0/prototype.pen，回到这里告诉我"
              
你：          在 Pencil 中设计，满意后保存

你：          "保存好了"
              
Claude Code:  自动跑评审（/critique + Microcopy + Visual Diff）
              "评审发现 3 个问题：..."
              
你：          确认修改 / 回 Pencil 调整
              
Claude Code:  "Visual Diff 通过。进入 Phase 2。"
```

### 步骤

#### 1. AI 辅助设计评审（设计前加载专家思维）

**在打开 Pencil 之前**，AI 加载 `docs/design-constraints.md` 中的专家规范，提取为 6 个检查维度，作为后续设计的「UX 总监视角」：

| 维度 | 检查点 |
|------|--------|
| 克制（C0.x） | 一个页面一种 primary action？多余元素能删则删？ |
| 间距（S1.x） | 相关元素 ≤ 12px，不相关 ≥ 24px？同层间距一致？ |
| 层次（H2.x） | 只有一个视觉焦点？字号只取阶梯值？字重只 400 和 600-700？ |
| 形状（R3.x） | 圆角只取 6/8/12/16/9999？边框 1px？有阴影不要边框？ |
| 色彩（C4.x） | 一个品牌色？文字 3 级灰度？不用纯黑？ |
| 可用性（U6.x） | 导航 ≤ 7 项？点击区域 ≥ 44px？ |

这不是事后检查，是**设计时的实时决策辅助**。AI 在 Pencil 设计过程中主动提示 PM：
- 「这个页面有两个填充色按钮，违反了 C0.1，建议第二个改成 outline」
- 「当前卡片间距是 14px，不是 4 的倍数，建议改为 16px」
- 「导航有 9 项，超过了 U6.1 的 7 项上限，建议分组」

**PM 不需要记住规则**，AI 自动加载并提示，PM 只需判断「要不要改」。

#### 2. 在 Pencil GUI 中对话式设计

**本阶段核心原则：设计在 GUI，不在 MCP。**

**Claude Code 交接动作**：
- Claude 将 DESIGN_BRIEF.md 的内容格式化输出给用户
- 告诉用户：「打开 Pencil → 新建文档 → 粘贴以下内容到 AI agent → 开始设计 → 满意后保存为 `specs/sprint-0/prototype.pen` → 回来告诉我」

**用户操作**：
1. 打开 Pencil 桌面应用，新建文档
2. 将 Claude 输出的 DESIGN_BRIEF 内容粘贴到 Pencil AI agent 对话框
3. 在 Pencil GUI 中对话式设计，边看画布边调整
4. 满意后，Cmd+S 保存为 `specs/sprint-0/prototype.pen`（保存到项目目录）
5. 回到 Claude Code，说「保存好了」
4. 此阶段 Claude Code 等待你操作，不参与 Pencil 内的设计
5. 设计满意后，**Cmd+S 保存**为 `specs/sprint-0/prototype.pen`
6. **回到 Claude Code**，告诉 Claude「原型保存好了」
7. Claude 自动执行后续评审（步骤 3-9），完成后把结果报告给你

**Claude Code 在此阶段的职责**：等你返回原型后，自动运行 Microcopy 评审、专家规范检查、/critique、Visual Diff Check。不干预 Pencil 内的设计过程。

打开 Pencil 桌面应用，新建文档。把 `DESIGN_BRIEF.md` 的内容粘贴到 Pencil 内置 AI agent 对话框中，作为上下文。

**强制使用真实数据占位**，不用 Lorem Ipsum：
- 标题文字用 PRD 中定义的实际文案长度范围（最短/最长）
- 列表用 PRD 中定义的典型数据量（最少 1 条、典型 20 条、最大 100+ 条）
- 空状态必须展示一次（「暂无数据」场景）
- 极端情况必须展示一次（超长标题、极短描述）

**为什么用真实数据**：Lorem Ipsum 看起来「还行」，但真实数据长度会暴露间距不足、文字溢出、布局挤压等问题。

AI agent 会基于 Brief 生成初始设计方案。你在实时画布上查看效果，通过对话指令调整：
- 「Hero 区域再大气一些」
- 「按钮颜色换成品牌色」
- 「卡片间距拉大到 32px」
- 「这个板块太满了，删掉」

**AI 在此过程中持续加载专家规范**，每完成一个页面的初稿，自动运行 6 维度检查并列出发现的问题和修改建议。

**优先使用 `design/shadcn.lib.pen` 中的组件**。通过 `ref` 引用 shadcn 组件（Button、Card、Input、Dialog、Sidebar 等），保证设计一致性。

#### 3. Microcopy 评审（文案即体验）

原型初稿完成后，AI 审查所有界面文案：

| 检查项 | 错误示例 | 正确示例 |
|--------|---------|---------|
| 按钮文字是否描述具体行为？ | "提交" | "创建你的第一个项目" |
| 空状态是否提供下一步？ | "暂无数据" | "暂无数据，点击上传第一张图片" |
| 错误提示是否告诉用户怎么做？ | "操作失败" | "网络连接超时，请检查网络后重试" |
| 文案语气是否与品牌调性一致？ | 官方腔、技术术语 | 用户语言、自然表达 |
| 是否有未翻译的占位文字？ | "Text here" | 实际中文文案 |

文案是 UX 的灵魂。一个按钮叫"立即开始"vs"注册"vs"创建你的第一个项目"，体验完全不同。

#### 4. 验证组件库可用（强制 Gate，不可跳过）

`scaffolds/project/design/shadcn.lib.pen` 已复制到项目，但 .pen 文件必须通过 Pencil 编辑器打开并保存后，MCP 才能正确读取其内容。**必须先验证再使用。**

验证步骤：
1. 在 Pencil 桌面应用中打开 `design/shadcn.lib.pen`，Cmd+S 保存
2. 用 `batch_get` 搜索 `design/shadcn.lib.pen` 的 reusable 组件
3. **如果返回非空** → 组件可用，继续下一步，用 `ref` 引用 shadcn 组件
4. **如果返回空** → 组件库不可读，**暂停 Phase 1，报告给用户**，不要自行跳过或用原生组件替代

#### 5. 保存原型

Cmd+S 保存为 `specs/sprint-0/prototype.pen`。然后用 CLI 确保文件持久化：

```bash
expect <<'EOF'
set timeout 30
spawn pencil interactive -i "specs/sprint-0/prototype.pen" -o "specs/sprint-0/prototype.pen"
expect "pencil >"
send "save()\r"
expect "pencil >"
send "exit()\r"
expect eof
EOF
```

#### 6. 原型走查

通过 `get_screenshot` 渲染截面图，在浏览器/查看器中真实走查核心路径：
- 用户要完成什么？
- 第一步点哪里？
- 系统怎么反馈？
- 出错怎么办？

#### 7. 自动化 UX 评审（`/critique`）

原型截图出来后，**必须**跑 `/critique [页面描述]`。它会：
- 检测 AI Slop（紫色渐变、玻璃拟态、通用卡片网格等）
- 评分 Nielsen 10 条启发式规则
- 检查认知负荷（每个决策点可见选项是否 >4）
- 输出 3-5 个优先修复项

`/critique` 针对的是已完成页面，检查的是 UX 逻辑，不是视觉品质。

#### 8. Visual Diff Check（视觉质量 Gate，不可跳过）

**AI 辅助对比**：AI 将原型截图与 Phase 0 收集的竞品基准截图并排分析，自动识别差距点：
- 排版层级：竞品的大标题多大？我们的够不够突出？
- 间距密度：竞品是紧凑还是宽松？我们的一致吗？
- 圆角风格：竞品按钮圆角大小？我们的匹配吗？
- 品牌色使用：竞品如何用品牌色？我们是否过度或不足？
- 信息密度：竞品一屏展示多少信息？我们的平衡吗？

然后由 PM 回答三个定性问题：
- 这个设计放在竞品旁边，看起来是一个档次的吗？
- **Landing Page 标准**：是否有品牌感？是否有视觉冲击力？是否有情感共鸣？
- **Tool/Editor 标准**：是否清晰？是否高效？是否不杂乱？信息层级是否一目了然？

如果回答「否」→ 回到 Pencil GUI 调整，不要进入 Phase 2。

**为什么需要 AI 辅助的 Visual Diff Check**：`/critique` 只能检查 UX 逻辑（层级、信息架构、认知负荷），不能判断视觉品质。一个 UX 逻辑完美的设计可能在视觉上依然粗糙。缺乏经验的 PM 可能看不出具体差距，AI 负责「找出差异点」，PM 负责「判断是否接受」——让 AI 做 PM 的眼睛。

#### 9. 记录并调整 / 迭代循环

记录发现的问题和调整方案，调整 PRD（如果需要）。

**如果评审发现问题**：
- Claude 列出问题清单 + 修改建议
- 告诉用户：「回 Pencil 调整以下 X 处，保存后回来」
- 用户回到 Pencil 修改 → 保存 → 回来
- Claude 重新跑评审，直到通过

**如果全部通过**：
- Claude 报告：「Phase 1 完成。所有检查通过。进入 Phase 2。」

### 验证

- 至少一条核心路径在真实 UI 截图上能完整走通
- 6 维度专家检查无重大问题（或 PM 已确认接受）
- Microcopy 评审通过（所有文案提供行为指引，非占位文字）
- `/critique` UX 逻辑评分无明显缺陷
- Visual Diff Check 通过（AI 辅助对比竞品，PM 确认为同一档次）
- 原型已在 Pencil 中渲染可见，不是文字想象

**为什么需要 Sprint 0**：Pencil 生成的真实截面图成本极低（几分钟对话式出图），但能暴露纯文字想不到的间距、排版、色彩对比度等问题。

### 工具参考

- Pencil MCP 操作手册见 `docs/pencil-playbook.md`，所有操作已逐项测试验证
- 设计约束规范见 `docs/design-constraints.md`，Phase 1 设计阶段和 Phase 4 代码阶段全程加载

---

## Phase 2: 架构

**触发**：Sprint 0 原型验证通过
**目标**：技术选型 + 项目骨架 + Design Tokens
**产出**：`ARCHITECTURE.md` + `design/tokens.css`

### 步骤

1. 技术选型（框架、状态管理、样式方案、测试工具）
2. 记录为什么选，不选什么
3. 定义 `design/tokens.css`（Design Tokens）：
   - 品牌色（来自 `.impeccable.md`）
   - 字号阶梯
   - 间距系统
   - 圆角系统
4. 定义目录结构
5. 初始化项目骨架（用 `scaffolds/project/` 复制）

### 架构文档必须包含

- 技术栈 + 选型理由（为什么，不只选什么）
- 目录结构约定
- 状态管理方案
- 样式方案（Tailwind / CSS-in-JS / 其他）
- 测试策略（用什么测、测什么、Done 定义）
- Design Tokens 定义（`tokens.css`）
- 已知的技术债务或待决策项

---

## Phase 3: 规划（Specs）

**触发**：架构已确认
**目标**：拆解 Sprint + Story，每个 Story 自带 Design Spec
**产出**：`specs/sprint-N/M.N-story-name.md`

### Story 模板

每个 Story 文件必须包含以下结构（见 `templates/story.md`）：

```markdown
# M.N Story 标题

Status: ready-for-dev

## User Story
作为 [角色]，
我想要 [行为]，
以便 [收益]。

## Design Spec
- 布局：[引用 prototype.pen 中的页面或组件]
- 组件：[用到哪些组件]
- 状态矩阵：[default / hover / active / disabled / loading]
- 交互：[点击、动画、反馈]

## 验收标准
- [ ] 逻辑测试：vitest
- [ ] UI 测试：真实浏览器点击 + 人眼视觉确认（对比 prototype.pen 截图）
- [ ] Design Spec 中所有状态已实现

## Dev Notes
- 技术约束
- 引用的架构决策
- 测试标准
```

### 关键规则

1. **Design Spec 内嵌在 Story 文件里**，不建独立的 design 文档制造双份
2. **状态矩阵是必须的**，不写状态矩阵的 Story 不能进入 ready-for-dev
3. **验收标准分两级**：
   - Must：所有 Story — vitest + 人眼视觉确认
   - Should：核心用户路径 — + Playwright E2E

### 进度追踪

用 `sprint-status.yaml` 追踪每个 Story 的状态：
```yaml
sprint-1:
  1.1-video-player: ready-for-dev    ← Story 模板已写好，可以开发
  1.2-waveform: in-progress          ← 正在开发
  1.3-playback: done                 ← 所有验收通过
```

### Planning Audit Gate（规划审计门）

Phase 3 写完所有 Specs 后，**进入实现前**必须经过此门。不可跳过。

目的：AI 生成的 Specs 可能有边界不清、依赖不诚实、规格过大的问题。人审合同边界，不审实现细节。

审计清单：

- [ ] 每个 Story 的 Design Spec 包含完整的状态矩阵
- [ ] 每个 Story 的粒度合理（能在数小时至数天内完成）
- [ ] Story 之间的依赖关系已明确标注（`_Depends:_`）
- [ ] 没有两个 Story 修改同一组文件（边界冲突）
- [ ] 核心用户路径的 Story 已标记为高优先级
- [ ] 空状态、Loading 状态在每个相关 Story 中都有定义

如果审计不通过：打回 Phase 3 修改，不进入实现。
如果审计通过：将所有 Story 标记为 `ready-for-dev`。

---

## Phase 4: 实现

**触发**：Story 状态为 `ready-for-dev`
**目标**：将 Pencil 原型翻译为代码 + 测试，直到 Story 的验收标准全部通过
**产出**：`src/` + `tests/`

**Design Constraints 全程生效。** 设计阶段（Phase 1）作为决策参考，代码阶段（Phase 4）作为硬性约束。

### 执行规则

1. **读取原型**：用 `batch_get` 读取 `prototype.pen` 结构，用 `get_screenshot` 查看真实效果
2. **加载约束**：读取 `CLAUDE.md` 中的设计约束章节 + `design/tokens.css`
3. **类型先行**：先定义 `src/types/` 中的数据结构
4. **逐像素翻译**：将 Pencil 原型翻译为代码，严格遵守：
   - 颜色只引用 tokens.css 变量，不硬编码
   - 间距只取 4px 倍数
   - 字号只取阶梯值
   - 圆角只取定义值
5. **组件规范**：每个组件 `ComponentName.tsx` + `ComponentName.test.tsx` 同级
6. **Design Token 单一来源**：颜色、字号、间距只定义在 `design/tokens.css`，代码只引用不重写
7. **不重复造轮子**：开发中发现组件已存在于 `design/components/`，直接复用
8. **Story 完成标准**：
   - 所有 Tasks/Subtasks 勾选
   - 所有 Acceptance Criteria 通过
   - 逻辑测试通过
   - 人眼视觉确认（启动 dev server 在浏览器里看，对比 prototype.pen 截图）

### 设计精细化（Impeccable，按需使用）

当 Story 的代码实现完成后，可在提交前做一轮设计抛光：

| 场景 | 命令 | 作用 |
|------|------|------|
| 整体视觉抛光 | `/polish [页面描述]` | 对齐、间距、排版、色彩、交互综合优化 |
| 排版问题 | `/typeset [页面描述]` | 字体层级、字重、字距、可读性 |
| 色彩单调 | `/colorize [页面描述]` | 为灰度过度的界面增加战略色彩 |
| 布局对齐 | `/layout [页面描述]` | 对齐、间距、视觉节奏 |

**规则**：Impeccable 是针对性修改，不是重写。每次只调一个维度，审 diff 后再决定下一步。不满意的修改直接回退。

### 发现新组件时

当 Story 需要用到未定义的组件：
1. 先写 `design/components/ComponentName.md`（状态矩阵）
2. 再写 `src/components/ComponentName.tsx`
3. 后续 Story 需要同组件时，直接引用已有定义

### 代码审查清单（每个 Story 完成时自检）

- [ ] 没有硬编码颜色值（只用 design/tokens.css 的变量）
- [ ] 图标尺寸统一（检查是否遵循已有规范）
- [ ] 空状态已处理
- [ ] Loading 状态已处理
- [ ] 交互有视觉反馈（hover、active、focus）
- [ ] 测试覆盖了核心逻辑
- [ ] 视觉与 prototype.pen 截图一致

---

## Phase 5: 验证（Show & Tell）

**触发**：Sprint 所有 Story 状态为 `done`
**目标**：真实用户验证，不是团队内部自嗨演示
**产出**：验证记录

### 步骤

1. 启动 dev server
2. 让一个**不熟悉这个功能的人**（或你自己换角色）完成核心任务
3. 观察：
   - 能否在 2 分钟内完成核心任务？
   - 在哪里卡住了？
   - 哪里觉得不舒服？
4. 记录问题

### 自动化设计评审（Impeccable，可选）

如果项目安装了 Impeccable，在人眼验证之前先跑一轮自动化评审：

```bash
/critique [页面描述]    # UX 评审：层级、信息架构、认知负荷、情感共鸣
/audit [页面描述]       # 技术评审：无障碍、性能、主题、响应式、反模式
```

两个命令都会输出量化评分 + 具体问题 + 修复建议。评审结果作为 Show & Tell 的补充输入，不替代人眼验证。

### 判定标准

| 结果 | 条件 | 行动 |
|------|------|------|
| **Pass** | 核心任务在 2 分钟内完成，无阻塞 | 进入下一 Sprint |
| **Fix** | 能完成但有明显卡顿 | 修问题，不新增功能，修完再验证 |
| **Fail** | 无法完成核心任务 | 回炉，重新设计交互，不讨论修修补补 |

### 为什么 Show & Tell 不能省

SubtitleScribe 项目的教训：功能齐全、测试通过、但"界面不可用"。因为所有验证都是"测试能不能跑"，不是"人能不能用"。

---

## Kill Switch

任何阶段发现方向错误，直接回退，不讨论沉没成本：

- PRD 阶段：重写好过改
- Sprint 0：发现核心路径走不通，改 PRD 或放弃
- 实现阶段：Show & Tell Fail，重新设计交互，不在原代码上补丁

**规则**：回退是流程的一部分，不是失败。

---

## Story 级纠错机制

Kill Switch 是阶段级的核按钮。Story 级别的纠错需要更精细，不炸掉整栋楼。

### 纠错状态机

Story 文件的状态不只是 `ready-for-dev` / `in-progress` / `done`，还有：

| 状态 | 含义 | 谁能发起 | 下一步 |
|------|------|---------|--------|
| `spec-revision-needed` | Spec 合同本身有问题，需要修改 | 用户或 AI | 用户改 Spec → Planning Audit Gate 再审 |
| `impl-revision-needed` | 实现不符合 Spec，Spec 本身没问题 | 用户或 AI | AI 改代码 → 重新验收 |
| `blocked` | 被其他 Story 或技术障碍阻塞 | AI | 解除阻塞后恢复为 `in-progress` |

### 触发条件

**spec-revision-needed**（合同错了）：
- 用户审 Story 时发现 Design Spec 的交互走不通
- 实现过程中发现 Spec 的组件定义自相矛盾
- 规划审计门通过但用户后来发现边界定义不对

操作：
1. AI 在 Story 文件的 Dev Agent Record 中记录 `## Revision Notes`：为什么需要改 Spec、改了什么
2. Story 状态改为 `spec-revision-needed`
3. 用户修改 Spec（Design Spec 部分），不改代码
4. 修改完成后，重新经过 Planning Audit Gate（只审改过的 Story）
5. 审过之后恢复为 `ready-for-dev`（如果已经实现过，则改为 `impl-revision-needed`）

**impl-revision-needed**（实现错了，合同没问题）：
- 逻辑测试通过但人眼视觉确认时发现不对
- Show & Tell 发现某个 Story 的交互体验差
- 代码审查清单有未通过项

操作：
1. AI 在 Story 文件的 Dev Agent Record 中记录 `## Revision Notes`：发现了什么视觉/交互问题
2. Story 状态从 `done` 改回 `impl-revision-needed`
3. AI 修正代码，不修改 Spec
4. 重新走验收流程：逻辑测试 → 人眼视觉确认 → 代码审查清单

**blocked**（被阻塞）：
- 依赖的 Story 还没完成（`_Depends:_` 标注的 Story 状态不是 `done`）
- 技术障碍（如某个库不支持预期行为）

操作：
1. AI 在 Dev Agent Record 中记录阻塞原因
2. Story 状态改为 `blocked`
3. 如果是依赖问题：先完成被依赖的 Story
4. 如果是技术障碍：升级到用户决策——换方案或调整 Spec

### 纠错后的验证

每个纠错路径都有闭环验证：

| 纠错类型 | 验证方式 | 关闭条件 |
|---------|---------|---------|
| spec-revision | Planning Audit Gate 再审 | 审计通过，状态回到 ready-for-dev |
| impl-revision | 逻辑测试 + 人眼视觉确认 | 验收通过，状态回到 done |
| blocked | 阻塞原因消除 | 恢复为 in-progress，继续实现 |

### 为什么需要 Story 级纠错

SubtitleScribe 的教训：发现 UI 不可用时，已经是 Sprint 3。之前所有 Story 都标记为"测试通过"，但没人说过"这个 Spec 本身就有问题"。没有 `spec-revision-needed` 状态，就只能默默往下走，直到 Show & Tell 炸掉。

有了这个机制，任何一个 Story 在任何一个阶段发现问题，都有明确的回退路径，不是"全做完再说"。
