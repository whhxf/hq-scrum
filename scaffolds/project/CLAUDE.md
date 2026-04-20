---
name: sop-rules
description: hq-scrum SOP 流程规则。使用此 SOP 的项目必须遵守。
type: feedback
---

**SOP 完整流程见 `WORKFLOW.md`**。本文件只保留初始化规则和不可跳过的红线。

**初始化**：当用户说「hq-scrum 在 XXX 路径」或「初始化 hq-scrum」：

1. **先读规则**：读取本文件 + `WORKFLOW.md` 中的 Phase 流程，不要凭记忆操作
2. 确认目标路径存在 `scaffolds/project/` 目录
3. **逐文件检查，只复制不存在的文件，绝不覆盖任何已有文件**：
   - 使用 `cp -n`（no-clobber）或逐个检查文件是否存在后再复制
   - 禁止使用 `cp -r` 或 `cp -rf`（会无条件覆盖）
   - `CLAUDE.md`、`PRD.md`、`ARCHITECTURE.md`、`WORKFLOW.md` 已存在 → 直接跳过
   - 任何目录已存在 → 只复制该目录下不存在的文件
4. 复制完成后，**从第一个未完成的 Phase 开始**：
   - PRD 已填写 → 跳过 Phase 0，直接从 Phase 1 开始
   - PRD 未填写 → 从 Phase 0 开始
5. 向用户报告初始化结果（复制了哪些文件、跳过了哪些、当前从哪个 Phase 开始）

---

**不可跳过的红线**（详细说明见 WORKFLOW.md）：

- Phase -1（品牌）→ Phase 0（需求）→ Phase 1（原型）→ Phase 2（架构）→ Phase 3（规划）→ Phase 4（实现）→ Phase 5（验证）
- 不允许跳过 Phase 直接写代码
- Sprint 0 原型必须通过 Pencil GUI 生成真实 UI 截图
- Story 必须包含 Design Spec（状态矩阵）才能进入 ready-for-dev
- 每个 Story 完成标准：逻辑测试 + 人眼视觉确认
- Sprint 结束必须 Show & Tell
- 回退是流程的一部分，发现方向错直接回退

---

**Pencil MCP 速查**（日常操作参考）：

- 先 Cmd+S 保存再操作（`/new` 临时文档 ID 不稳定）
- 新建 Frame 错开 x 坐标：`x_N = (N-1) × 1480`
- 不要用 `replace_all_matching_properties`
- 不要全删重建：`batch_get` 读状态 → `U()` 精确更新 → `I()` 增量添加
- 自动保存用 `expect` + `pencil interactive`（见 WORKFLOW.md）

### 工具交接协议

Phase 1 涉及工具切换。遵循以下协议，主动告诉用户下一步做什么：

1. **Phase 0 完成后**：输出 Pencil 操作指引 + DESIGN_BRIEF.md 核心内容 → 告诉用户「去 Pencil 设计，保存后回来」
2. **用户说「保存好了」**：读取 prototype.pen → 跑 /critique + Microcopy + Visual Diff → 报告结果
3. **发现问题**：列出问题清单 → 告诉用户「回 Pencil 调整 X 处，保存后再回来」
4. **全部通过**：告诉用户「Phase 1 完成，进入 Phase 2」

不要让等用户来猜。每个阶段结束，给出明确的下一步指令。

常用 shadcn 组件 ID：

| Sidebar | `1:PV1ln` | Button | `1:VSnC2` | Card | `1:pcGlv` | Dialog | `1:OtykB` |
|---------|-----------|--------|-----------|------|-----------|--------|-----------|
| Badge | `1:UjXug` | Input | `1:fEUdI` | Tabs | `1:PbofX` | Alert | `1:QyzNg` |
