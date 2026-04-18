---
name: shadcn-ui-default
description: shadcn/ui 作为默认设计系统。即使项目 PRD/架构没提，也默认使用此方案。
type: feedback
---

**shadcn/ui 是默认设计系统，不可跳过。**

原因：
- 行业事实标准，112k+ stars，7 万+项目在用
- 质量稳定，Radix 底层保证无障碍
- 零运行时开销，直接生成 Tailwind 代码
- 组件覆盖所有常见交互场景（Button、Dialog、Input、Select、Toast、DropdownMenu 等）

安装：`npx shadcn@latest init` + `npx shadcn@latest add [component]`

品牌定制规则：
- 品牌色通过 CSS 变量注入（`--background`、`--primary` 等）
- 圆角统一使用 `lg`（0.5rem）
- 字体用项目已有字体栈
- 不使用默认 dark 模式，跟随项目 token 定义

此规范是默认值，不是建议。除非用户明确要求其他方案，否则一律用 shadcn/ui。
