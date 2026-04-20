# Architecture: {{project_name}}

---

## 技术栈

<!-- Phase 2 架构确认后填写。每个技术必须写为什么选 -->

| 层级 | 技术 | 为什么选 | 不选什么 |
|------|------|---------|---------|
| 框架 | | | |
| 样式 | | | |
| 状态管理 | | | |
| 测试 | | | |
| 构建 | | | |

---

## 目录结构

```
project/
├── CLAUDE.md              ← 项目约定
├── PRD.md                 ← 需求
├── ARCHITECTURE.md        ← 本文件
├── specs/                 ← Sprint Story 文件（.pen 原型也在此目录，与 Story 同名）
│   └── sprint-N/
├── design/                ← 设计资产
│   ├── tokens.css         ← 设计 Token（单一来源）
│   ├── shadcn.lib.pen     ← shadcn/ui 完整组件库（可 ref 引用）
│   └── components/        ← 可复用组件规范（含 .lib.pen 设计库）
├── src/                   ← 代码
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   ├── types/
│   └── store/
└── tests/
    └── e2e/               ← UI 验证测试
```

---

## 状态管理方案

<!-- 怎么管数据流，为什么这么选 -->

---

## 样式方案

<!-- CSS 方案、Token 管理方式、组件样式约定 -->

- Design Token 单一来源：`design/tokens.css`
- 代码只引用 token，不硬编码颜色值
- 组件规范写在 `design/components/`

---

## 测试策略

<!-- 用什么测、测什么、Done 定义 -->

| 层级 | 工具 | 测什么 | Done 定义 |
|------|------|--------|----------|
| 单元 | vitest | 组件逻辑、hooks、utils | 全部通过 |
| UI | 真实浏览器 | 视觉确认、核心路径 | 人眼确认 |
| E2E | Playwright（可选） | 核心用户路径 | 全部通过 |

---

## 已知技术债务

<!-- 暂时不做，后面补 -->

-
