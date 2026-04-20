---
name: impeccable
description: Impeccable Design — 18 个设计命令，覆盖 UX 发现、原型评审、代码抛光、色彩、排版、布局优化
---

# Impeccable Design Skill

一套针对 AI 生成的前端界面的设计改进工具。18 个命令，每个只调一个维度，审 diff 再决定下一步。

## 安装

```bash
npx skills add pbakaus/impeccable
```

安装后以下命令可用：

## Phase 1 — 原型阶段

### `/shape [功能描述]`
结构化 UX 访谈。问 5-10 个问题（用户是谁/场景/数据量级/空状态/情绪预期/禁忌），产出一份 Design Brief，指导后续原型方向。

### `/critique [页面描述]`
UX 评审。检测 AI Slop（紫色渐变、玻璃拟态、通用卡片网格等）、评分 Nielsen 10 条启发式规则、检查认知负荷（决策点可见选项 >4）、输出 3-5 个优先修复项。

## Phase 4 — 实现后抛光

### `/polish [页面描述]`
对齐、间距、一致性综合优化。

### `/typeset [页面描述]`
字体层级、字重、字距、可读性。

### `/colorize [页面描述]`
为灰度过度的界面增加战略色彩。

### `/layout [页面描述]`
布局、间距、视觉节奏优化。

### `/clarify [页面描述]`
优化界面文案、错误提示、微文案、标签。

## Phase 5 — 验证评审

### `/audit [页面描述]`
技术检查：无障碍（a11y）、性能、主题、响应式设计。

## 其他命令

| 命令 | 作用 |
|------|------|
| `/distill` | 精简设计，删除不必要复杂度 |
| `/bolder` | 放大视觉冲击力 |
| `/quieter` | 降低视觉强度，减少刺激 |
| `/delight` | 添加愉悦细节和惊喜感 |
| `/animate` | 添加有意义的动画和微交互 |
| `/overdrive` | 极限视觉效果，超越常规 |
| `/optimize` | 诊断和修复 UI 性能（加载、渲染、动画、图片、bundle） |
| `/adapt` | 适配不同屏幕尺寸和设备 |
| `/onboard` | 优化 onboarding 流程、空状态、首次体验 |
| `/harden` | 提升界面韧性：错误处理、i18n、文字溢出 |
| `/extract` | 提取可复用组件、设计 token、模式 |
| `/normalize` | 对齐设计系统，确保一致性 |
| `/teach-impeccable` | 建立项目设计上下文，生成 `.impeccable.md` |

## 使用原则

1. 每次只调一个维度，不要同时运行多个命令
2. 审 diff 后再决定下一步
3. Impeccable 是针对性修改，不是重写
4. 原型阶段用 `/shape` + `/critique`，实现后用 `/polish` + `/typeset` + `/colorize`
