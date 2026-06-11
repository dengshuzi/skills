# 业务上下文管理

## 作用

要件定义 Grill 的质量依赖上下文管理，但当前还没有展开完整知识库建设方案。

## 最小要求

至少要能支持下面几类信息：

- 术语表
- 模块边界说明
- 当前讨论对象的现状说明
- 既存需求或既存行为依据
- 本次讨论的 `context-pack.md`

## 实施要求

### 1. 能支撑业务识别

AI 至少要能判断：

- 当前在讨论哪个业务域
- 当前涉及哪个模块
- 当前属于哪类变更

### 2. 能支撑事实与假设区分

至少要区分：

- 文档事实
- 用户确认
- AI 假设
- 待确认问题

### 3. 能支撑会话持续推进

每次 Grill 后，至少应能更新：

- 已确认事实
- 被否定的假设
- 当前未决问题
- 下一步最关键问题

## 目录

目录结构：

```text
AI/
  README.md
  docs/
    00-blueprint/
    10-v-model-framework/
    20-knowledge-context/
    30-requirement-definition/
    40-platform-view/
    90-reference/
  knowledge/
    glossary/
    domains/
      <业务域>/
        modules/
          <模块名>/
            current-state.md
            rules.md
            interfaces.md
            known-issues.md
            change-history.md
            requirements/
            designs/
            tests/
            defects/
  projects/
    <项目或改修ID>/
      context-pack.md
      grill-session.md
      requirement-draft.md
      open-questions.md
      traceability.md
```

## 输出关系

当前这部分只需要服务 `要件定义`：

- 输入：一句话需求描述 + 现有业务资料
- 中间层：`context-pack.md`
- 输出：`requirement-draft.md` 与 `next-phase-input.md`
