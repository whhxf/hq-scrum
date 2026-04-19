---
name: sop-rules
description: hq-scrum SOP 流程规则。使用此 SOP 的项目必须遵守。
type: feedback
---

**hq-scrum SOP 规则 — 不可跳过：**

- Phase 0（PRD）→ Phase 1（Sprint 0 Pencil 原型）→ Phase 2（架构）→ Phase 3（Specs）→ Phase 4（实现）→ Phase 5（Show & Tell）
- 不允许跳过 Phase 直接写代码
- Sprint 0 原型必须通过 Pencil 生成真实 UI 截图，不可用 ASCII 代替
- Pencil 操作手册见 `docs/pencil-playbook.md`，包含全部操作语法、shadcn 组件 ID 速查、布局模式和已知坑
- 设计约束规范见 `docs/design-constraints.md`，将 PRD 文字/ASCII 翻译为视觉规则（CRAP 原则 + 间距系统 + 颜色/字体/阴影约束 + 语义提取规则）
- Story 必须包含 Design Spec（状态矩阵）才能进入 ready-for-dev
- 每个 Story 完成标准：逻辑测试 + 人眼视觉确认
- Sprint 结束必须 Show & Tell，不是自嗨演示
- 回退是流程的一部分，发现方向错直接回退，不讨论沉没成本
