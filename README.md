# hq-scrum

通过对话式 AI 完成的前端开发 SOP。适用于任何项目，从个人工具到产品迭代。

## 一句话

**Spec = 合同边界。PRD → Sprint 0 → 架构 → Specs（含合同）→ 实现 → Show & Tell**

每个阶段产出落到固定文件位置，后续阶段直接读，不靠记忆。AI 写 Spec，人审合同。

## 核心理念

- **合同边界**：Design Spec 是人和 AI 之间的合同。合同内 AI 自由实现，合同外人审边界决策。
- **零安装**：不装 CLI，不配 commands。cp -r scaffold 到项目，打开 Claude Code 说「按流程开始」。
- **视觉验收**：Sprint 0 原型 + 人眼视觉确认 + Show & Tell。不只看测试能不能跑。

## 快速开始

### 新项目初始化

1. **复制 scaffold**：
   ```bash
   cp -r /path/to/hq-scrum/scaffolds/project/* /path/to/your/project/
   ```
   复制后新项目根目录包含：
   - `CLAUDE.md` — SOP 规则（LLM 入口，自动加载）
   - `PRD.md` — 需求模板
   - `ARCHITECTURE.md` — 架构模板
   - `design/shadcn.lib.pen` — shadcn/ui 组件库
   - `design/tokens.css` — 设计 Token
   - `docs/` — Pencil 操作手册 + 设计约束
   - `specs/` — Sprint 原型目录
   - `.agents/skills/impeccable/` — Impeccable 设计 skill

2. **在 Claude Code 中打开新项目**

3. **告诉 Claude Code**：「按 hq-scrum 流程，从 Sprint 0 开始」

4. **Claude Code 自动执行**：
   - Phase 0：填充 PRD.md
   - Phase 1：`/shape` UX 访谈 → Pencil 原型（用 shadcn 组件）→ `/critique` UX 评审
   - Phase 2：填写 ARCHITECTURE.md
   - Phase 3：编写 Story Specs
   - Phase 4：编码实现
   - Phase 5：Show & Tell 验证

## 目录结构

```
hq-scrum/
├── README.md                    ← 本文件
├── WORKFLOW.md                  ← 6 个 Phase 的详细步骤、产出、验证标准
├── templates/                   ← Story、sprint-status、UX Spec 模板
└── scaffolds/
    └── project/                 ← 复制到新项目的完整骨架
        ├── CLAUDE.md            ← SOP 规则（LLM 入口）
        ├── PRD.md               ← 需求模板
        ├── ARCHITECTURE.md      ← 架构模板
        ├── design/              ← 设计资产
        │   ├── shadcn.lib.pen   ← shadcn/ui 组件库
        │   ├── tokens.css       ← 设计 Token
        │   └── components/      ← 项目自定义组件
        ├── docs/                ← 参考文档
        │   ├── pencil-playbook.md
        │   └── design-constraints.md
        ├── specs/               ← Sprint 原型
        │   └── sprint-0/
        │       └── 0.1-prototype.md
        ├── .agents/skills/      ← 项目级 skill
        │   └── impeccable/      ← Impeccable 设计 skill
        └── src/                 ← 代码目录
```

## 社区方案对比（2026-04-18）

| 方案 | Stars | 适用场景 | 有无 UI 验收 | 学习成本 |
|------|-------|---------|-------------|---------|
| **BMAD-METHOD** | 45k | 企业级团队 | 无 | 61 commands |
| **cc-sdd** | — | 多 spec 并行 | 无 | 17 skills |
| **spec-driven-workflow** | 79 | 中小团队 | 无 | 4 commands |
| **hq-scrum** | 0 | 个人开发者 | **有** | 零安装 |

社区方案都聚焦"功能和架构的正确性"，**无一管理 UI 视觉质量**。这是 hq-scrum 的独特价值。

## 版本

v0.1.0 — 初始版本，吸收了 BMAD-METHOD、cc-sdd、spec-driven-workflow 的最佳实践
