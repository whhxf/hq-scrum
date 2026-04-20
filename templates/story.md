# {{epic_num}}.{{story_num}}: {{story_title}}

Status: ready-for-dev

<!--
  合同边界说明（Planning Audit Gate 审这些）：
  - _Boundary_: 此 Story 负责的文件/组件边界
  - _Depends_: 依赖的其他 Story 或组件
  - 合同内：AI 自由实现
  -合同外：人审边界决策
-->

_Boundary_: [此 Story 负责的文件/组件范围]
_Depends_: [依赖的 Story 编号，无则写 None]

---

## User Story

作为 [角色]，
我想要 [行为]，
以便 [收益]。

---

## Design Spec

<!-- 必填。不写状态矩阵的 Story 不能进入 ready-for-dev -->

### 布局

<!-- 引用 prototype.pen 中的页面和组件路径。说明关键区域的位置关系 -->

原型文件：`specs/sprint-N/prototype.pen`
对应 Frame：[Frame 名称或 ID]

### 组件清单

| 组件 | 用途 | 是否复用 |
|------|------|----------|
| Button | 操作按钮 | design/components/Button.md |
| [新组件] | [说明] | 新建 |

### 状态矩阵

<!-- 每个交互组件必须定义以下状态 -->

#### [组件名]

| 状态 | 背景 | 文字 | 边框 | 光标 | 动画 |
|------|------|------|------|------|------|
| default | | | | | |
| hover | | | | | |
| active | | | | | |
| disabled | | | | | |
| loading | | | | | |
| focus | | | | | outline: brand |

### 空状态

<!-- 数据为空时显示什么 -->

### Loading 状态

<!-- 数据加载时显示什么 -->

---

## Acceptance Criteria

<!-- 从 PRD Epic 中提取，必须可验证 -->

1. [具体条件]
2. [具体条件]

---

## Tasks / Subtasks

- [ ] Task 1（AC: #）
  - [ ] 定义类型 `src/types/`
  - [ ] 写组件 `src/components/`
  - [ ] 写测试 `*.test.tsx`
  - [ ] 人眼视觉确认
- [ ] Task 2（AC: #）
  - [ ] Subtask

---

## Dev Notes

<!-- 从 ARCHITECTURE.md 中提取的技术约束 -->

- 技术约束：
- 样式方案：
- 测试标准：

### References

- PRD: `PRD.md`
- Architecture: `ARCHITECTURE.md`
- Design Tokens: `design/tokens.css`
- Prototype: `specs/sprint-N/prototype.pen`
- Design Brief: `DESIGN_BRIEF.md`

---

## Dev Agent Record

### Agent Model Used

{{agent_model}}

### Completion Notes

-

### File List

-

### Change Log

| Date | Change | Author |
|------|--------|--------|
| | | |
