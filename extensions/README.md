# Extensions

扩展口。当 SOP 流程稳定后，将最佳实践填入对应目录。

## 已调研的社区方案（2026-04-18）

### BMAD-METHOD（45k ⭐）

- **定位**：AI 驱动的企业级敏捷开发框架，34+ workflow
- **可借鉴**：Create/Edit/Validate 三模式、Story 自动从 PRD/Architecture/UX 提取上下文、sprint-status.yaml 追踪
- **不采用**：太重（61 commands、12-14 步/workflow），不适合个人开发者
- **集成状态**：已吸收其 Create/Edit/Validate 思想到 Phase 4 实现规则

### cc-sdd（gotalab，Kiro 风格）

- **定位**：Spec-Driven Development for AI coding agents，17 skills，支持 8 个 AI 编程工具
- **可借鉴**：
  - **Spec = 合同边界**（已采纳到 WORKFLOW.md 核心原则）
  - **Boundary / Depends 注解**（已采纳到 Story 模板）
  - **Agents write spec, humans review contract**（已采纳到 Phase 3→4 流程）
  - **Planning Audit Gate**（已采纳到 Phase 3 末尾）
- **集成状态**：核心理念已采纳

### spec-driven-workflow（liatrio-labs，79 ⭐）

- **定位**：4 步 Markdown 流程：spec → tasks → impl → validate
- **可借鉴**：Context Verification Marker（emoji 标记防跑偏）、Planning Audit Gate 强制审批
- **集成状态**：Audit Gate 已采纳，emoji 标记待评估

### ViperJuice/claude-code-template

- **定位**：Claude Code 多语言项目模板，25 个子 Agent，TDD 强制执行
- **可借鉴**：Git worktree 隔离并行开发、Agent 角色分工
- **集成状态**：暂不采纳（偏团队场景，个人开发者用不到）

---

## 默认设计规范

### shadcn/ui（默认，必选）

即使项目 PRD 或 ARCHITECTURE.md 中没有提及，**也默认使用 shadcn/ui 作为设计系统**。

- 为什么：行业事实标准，112k+ stars，7 万+项目在用。Radix 底层保证无障碍，零运行时开销。
- 安装：`npx shadcn@latest init` + `npx shadcn@latest add [component]`
- 详细规则：见 `extensions/shadcn-ui.md`

### SaaS Boilerplate（推荐，用于官网/SaaS 项目）

- 为什么：基于 shadcn/ui 的完整 SaaS 模板，含 Landing Page + Dashboard + 用户管理
- 地址：[ixartz/SaaS-Boilerplate](https://github.com/ixartz/SaaS-Boilerplate)（7k ⭐）
- 适用：需要快速出官网或 SaaS 产品的场景

---

## 已调研的社区方案（2026-04-18）

### 测试工具集成

- **Playwright**: [待集成 — E2E 测试配置模板]
- **Chromatic**: [待评估 — 视觉回归测试]
- **[其他]**: [待补充]

### 工具链集成

- **Storybook**: [待集成 — 组件文档]
- **[其他]**: [待补充]

## 添加新扩展的规则

1. 在此 README 中记录扩展名称、用途、集成方式、调研日期
2. 在 `extensions/` 目录下创建对应的配置模板或指南
3. **默认扩展**（如 shadcn/ui）：所有项目自动应用，不需要用户确认
4. **可选扩展**（如 Playwright、Chromatic）：仅在用户明确需要时引入
5. 不要直接改 SOP 核心流程，扩展是默认值的补充，不是流程的变更
