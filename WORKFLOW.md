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
Phase 0: 需求   → PRD.md
Phase 1: 验证   → Sprint 0 原型（Pencil 真实 UI 截图，验证核心路径和视觉体验）
Phase 2: 架构   → ARCHITECTURE.md
Phase 3: 规划   → specs/sprint-N/*.md（Story 含 Design Spec）
  ↓ Planning Audit Gate（人审合同边界）
Phase 4: 实现   → src/ + tests/（每个 Story 独立完成）
Phase 5: 验证   → Show & Tell（真实用户验证核心任务）
```

每一阶段的产出是下一阶段的输入。不允许跳过阶段。

---

## Phase 0: 需求（PRD）

**触发**：用户描述了一个产品想法
**目标**：明确做什么、为谁做、不做什么
**产出**：`PRD.md`

### 步骤

1. 用户描述需求
2. Claude Code 按 PRD 模板生成 `PRD.md`，必须包含：
   - 目标用户
   - 核心用户路径（不超过 3 条）
   - 非目标（明确不做什么）
   - 约束（技术、时间、平台）
3. 用户确认 PRD

### 验证

- 核心用户路径能否在纸面上走通？不能走通说明需求没想清楚，回到步骤 1

---

## Phase 1: 验证（Sprint 0 原型）

**触发**：PRD 已确认
**目标**：在写代码前验证核心交互路径和视觉体验
**产出**：`specs/sprint-0/0.1-prototype.md` + `specs/sprint-0/prototype.pen`

### 步骤

1. 基于 PRD 的核心路径，使用 Pencil MCP 生成每个关键页面的 .pen 原型（使用默认设计规范）
2. 通过 `get_screenshot` 渲染截面图，在浏览器/查看器中真实走查核心路径：
   - 用户要完成什么？
   - 第一步点哪里？
   - 系统怎么反馈？
   - 出错怎么办？
3. 记录发现的问题和调整方案
4. 调整 PRD（如果需要）

### 验证

- 至少一条核心路径在真实 UI 截图上能完整走通
- 发现至少一个之前没想到的交互或视觉体验问题
- 原型已在 Pencil 中渲染可见，不是文字想象

**为什么需要 Sprint 0**：SubtitleScribe 项目的教训——写完 120 个测试后发现 UI 不可用。ASCII 原型发现不了间距、排版、色彩对比度等视觉问题，Pencil 生成的真实截面图成本极低（几分钟对话式出图），但能暴露 ASCII 发现不了的问题。

### Pencil MCP 操作规范

> 完整操作手册见 `docs/pencil-playbook.md`，所有操作已逐项测试验证。

### Impeccable 设计技能（可选增强）

> 如果项目已安装 Impeccable（`npx skills add pbakaus/impeccable`），在此阶段运行 `/impeccable teach` 建立设计上下文。生成的 `.impeccable.md` 将品牌定义、受众画像、视觉参考沉淀为项目级文件，后续所有设计命令自动读取。

---

## Phase 2: 架构

**触发**：Sprint 0 原型验证通过
**目标**：技术选型 + 项目骨架
**产出**：`ARCHITECTURE.md` + 项目骨架文件

### 步骤

1. 技术选型（框架、状态管理、样式方案、测试工具）
2. 记录为什么选，不选什么
3. 定义目录结构
4. 初始化项目骨架（用 `scaffolds/project/` 复制）

### 架构文档必须包含

- 技术栈 + 选型理由（为什么，不只选什么）
- 目录结构约定
- 状态管理方案
- 样式方案（Tailwind / CSS-in-JS / 其他）
- 测试策略（用什么测、测什么、Done 定义）
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
- 布局：[ASCII 原型或设计稿引用]
- 组件：[用到哪些组件]
- 状态矩阵：[default / hover / active / disabled / loading]
- 交互：[点击、动画、反馈]

## 验收标准
- [ ] 逻辑测试：vitest
- [ ] UI 测试：真实浏览器点击 + 人眼视觉确认
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
**目标**：写代码 + 测试，直到 Story 的验收标准全部通过
**产出**：`src/` + `tests/`

### 执行规则

1. **类型先行**：先定义 `src/types/` 中的数据结构
2. **组件规范**：每个组件 `ComponentName.tsx` + `ComponentName.test.tsx` 同级
3. **Design Token 单一来源**：颜色、字号、间距只定义在 `design/tokens.css`，代码只引用不重写
4. **不重复造轮子**：开发中发现组件已存在于 `design/components/`，直接复用
5. **Story 完成标准**：
   - 所有 Tasks/Subtasks 勾选
   - 所有 Acceptance Criteria 通过
   - 逻辑测试通过
   - 人眼视觉确认（启动 dev server 在浏览器里看）

### 发现新组件时

当 Story 需要用到未定义的组件：
1. 先写 `design/components/ComponentName.md`（状态矩阵）
2. 再写 `src/components/ComponentName.tsx`
3. 后续 Story 需要同组件时，直接引用已有定义

### 设计精细化（Impeccable，可选）

当 Story 的代码实现完成后，如果项目安装了 Impeccable，可在提交前做一轮设计抛光：

| 场景 | 命令 | 作用 |
|------|------|------|
| 整体视觉抛光 | `/polish [页面描述]` | 对齐、间距、排版、色彩、交互综合优化 |
| 排版问题 | `/typeset [页面描述]` | 字体层级、字重、字距、可读性 |
| 色彩单调 | `/colorize [页面描述]` | 为灰度过度的界面增加战略色彩 |
| 缺少动效 | `/animate [页面描述]` | 微交互、过渡动画、悬停反馈 |
| 增加视觉层次 | `/bolder [页面描述]` | 强化对比度和视觉层级 |
| 过度设计 | `/quieter [页面描述]` | 削减装饰，回归克制 |

**规则**：Impeccable 是针对性修改，不是重写。每次只调一个维度，审 diff 后再决定下一步。不满意的修改直接回退。

### 代码审查清单（每个 Story 完成时自检）

- [ ] 没有硬编码颜色值（只用 design/tokens.css 的变量）
- [ ] 图标尺寸统一（检查是否遵循已有规范）
- [ ] 空状态已处理
- [ ] Loading 状态已处理
- [ ] 交互有视觉反馈（hover、active、focus）
- [ ] 测试覆盖了核心逻辑

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
