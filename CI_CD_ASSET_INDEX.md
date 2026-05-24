# CI/CD 资产索引

这份文档是总目录，用来把 CI/CD 的关键决策沉淀成资产。

## 1. 资产清单

| 主题 | 记录位置 | 作用 |
| --- | --- | --- |
| Git Flow 与 CI/CD 映射 | [GIT_FLOW_CICD_MAPPING.md](/Users/d/Documents/GIT_FLOW_CICD_MAPPING.md) | 记录分支模型如何影响 CI/CD |
| CI 门禁设计 | [CODE_QUALITY_GATE_DESIGN.md](/Users/d/Documents/CODE_QUALITY_GATE_DESIGN.md) | 记录本地 hook、远端门禁、并行扫描、资产归档 |
| K8s CD 设计 | [CD_K8S_DELIVERY_DESIGN.md](/Users/d/Documents/CD_K8S_DELIVERY_DESIGN.md) | 记录环境映射、GitOps、灰度、回滚 |
| AI / AIOps 结合 | [AIOPS_CICD_INTEGRATION_DESIGN.md](/Users/d/Documents/AIOPS_CICD_INTEGRATION_DESIGN.md) | 记录 AI 在 CI/CD 中的辅助位置和边界 |

## 2. 当前约定顺序

```text
Git Flow
  -> CI 触发点
  -> CD 环境映射
  -> 发布审批和回滚
```

## 3. 当前已形成的核心结论

- CI/CD 必须先跟 Git Flow 对齐。
- CI 负责证明制品可信。
- CD 负责证明制品可稳定交付。
- 产物、报告、SBOM、扫描结果要进入资产库。
- AI 只能辅助分析和建议，不能绕过门禁。

## 4. 这份索引的用途

- 作为后续扩展的入口。
- 作为讨论和评审的统一引用点。
- 作为 CI/CD 设计的资产总目录。

