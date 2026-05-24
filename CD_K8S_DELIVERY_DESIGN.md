# K8s CD 流程设计

## 1. 总览图

```mermaid
flowchart LR
  A[("Artifactory / Image Registry<br/>image digest / Helm chart<br/>SBOM / scan report")]
  A --> B["阶段一<br/>Release Candidate<br/>选择已验证产物"]
  B --> C["阶段二<br/>Deploy to Dev<br/>自动部署开发环境"]
  C --> D["阶段三<br/>Promote to Test<br/>测试环境验证"]
  D --> E["阶段四<br/>Promote to Staging<br/>预发环境验证"]
  E --> F{"生产发布审批"}
  F --> G["阶段五<br/>Progressive Delivery<br/>生产灰度 / 金丝雀 / 蓝绿"]
  G --> H["阶段六<br/>Post-deploy Verification<br/>健康检查 / 指标 / 日志 / 告警"]
  H --> I{"是否健康？"}
  I -- "是" --> J["Release Complete<br/>发布完成"]
  I -- "否" --> K["Rollback<br/>自动或手动回滚"]

  Git[("GitOps Repo<br/>环境配置 / Helm values / Kustomize overlays")] --> Argo["Argo CD<br/>持续同步 / Diff / Drift 检测"]
  Argo --> C
  Argo --> D
  Argo --> E
  Argo --> G
```

## 2. 阶段说明

| 阶段 | 用什么 | 做什么 | 目的 |
| --- | --- | --- | --- |
| 阶段一：Release Candidate | Artifactory、Image Registry、SBOM、扫描报告 | 选择 CI 已验证的镜像 digest 和 chart | 只发布可信产物 |
| 阶段二：Deploy to Dev | Argo CD、Helm/Kustomize、K8s Dev Namespace | 自动部署开发环境 | 快速验证部署配置 |
| 阶段三：Promote to Test | Argo CD、自动化测试、API Test、E2E Test | 晋级到测试环境并跑验证 | 验证功能和接口行为 |
| 阶段四：Promote to Staging | Argo CD、Staging Cluster、生产相似配置 | 部署预发环境，跑冒烟和回归 | 尽量贴近生产验证 |
| 阶段五：Progressive Delivery | Argo Rollouts、Istio/Nginx Ingress、Prometheus | 生产灰度、金丝雀、蓝绿发布 | 降低生产发布风险 |
| 阶段六：Post-deploy Verification | Prometheus、Grafana、Loki/ELK、Alertmanager | 看健康检查、错误率、延迟、日志、告警 | 判断发布是否成功 |
| Rollback | Argo CD、Helm rollback、kubectl rollout undo | 回滚到上一个稳定版本 | 快速止损 |

## 3. 推荐环境

```mermaid
flowchart LR
  Dev["Dev<br/>开发联调<br/>自动部署"] --> Test["Test / QA<br/>测试验证<br/>自动晋级或手动触发"]
  Test --> Staging["Staging<br/>预发环境<br/>接近生产配置"]
  Staging --> Prod["Prod<br/>生产环境<br/>审批 + 灰度发布"]
```

推荐环境设计：

- Dev：自动部署，允许频繁变更。
- Test / QA：跑 API、E2E、回归测试。
- Staging：尽量贴近生产，用于发布前验证。
- Prod：必须审批，使用灰度、金丝雀或蓝绿发布。

## 4. GitOps 方式

推荐用 GitOps 管 CD，不建议直接在流水线里到处执行 `kubectl apply`。

```mermaid
flowchart LR
  A["更新环境配置<br/>image tag / image digest / values"] --> B["提交到 GitOps Repo"]
  B --> C["Argo CD 检测变更"]
  C --> D["同步到 K8s"]
  D --> E["状态回写<br/>Synced / Healthy / Degraded"]
```

GitOps Repo 里放：

- Helm chart。
- Helm values。
- Kustomize overlays。
- 每个环境的 image digest。
- ConfigMap / Secret 引用。
- HPA、Ingress、Service、Deployment 等 K8s 配置。

好处：

- 发布动作可审计。
- 环境状态可追踪。
- 配置变更可 Review。
- 集群漂移可以被 Argo CD 发现。
- 回滚可以通过 Git revert 或 Argo rollback 完成。

## 5. 生产发布策略

```mermaid
flowchart TD
  A["Prod Deploy Start"] --> B["Canary 5%"]
  B --> C{"指标健康？"}
  C -- "是" --> D["Canary 25%"]
  D --> E{"指标健康？"}
  E -- "是" --> F["Canary 50%"]
  F --> G{"指标健康？"}
  G -- "是" --> H["Full Rollout 100%"]
  C -- "否" --> R["Rollback"]
  E -- "否" --> R
  G -- "否" --> R
```

推荐灰度指标：

- Pod 是否 Ready。
- HTTP 5xx 是否升高。
- P95 / P99 延迟是否升高。
- CPU / Memory 是否异常。
- 核心业务指标是否异常。
- Error log 是否明显增加。

## 6. CD Quality Gate

CD 也应该有门禁，不是把镜像推上去就结束。

```mermaid
flowchart TD
  A["部署动作开始"] --> B{"部署是否成功？"}
  B -- "否" --> R["阻塞 / 回滚"]
  B -- "是" --> C{"健康检查是否通过？"}
  C -- "否" --> R
  C -- "是" --> D{"冒烟测试是否通过？"}
  D -- "否" --> R
  D -- "是" --> E{"监控指标是否正常？"}
  E -- "否" --> R
  E -- "是" --> F["允许晋级下一环境或完成发布"]
```

CD 门禁关注：

- K8s rollout 是否成功。
- Pod 是否 Ready。
- Service 是否可访问。
- 冒烟测试是否通过。
- 错误率和延迟是否正常。
- 告警是否触发。
- 灰度阶段是否满足晋级条件。

## 7. 和 CI 的关系

```mermaid
flowchart LR
  CI["CI<br/>构建 / 测试 / 扫描"] --> A[("Artifact Repo<br/>镜像 / chart / 报告")]
  A --> CD["CD<br/>选择可信产物并发布"]
  CD --> K8S["K8s<br/>Dev / Test / Staging / Prod"]
```

CI 负责证明这个产物“能不能被交付”。

CD 负责证明这个产物“能不能稳定运行在目标环境里”。

## 8. 最终推荐

```text
CI 产出可信制品，
CD 只发布可信制品，
GitOps 管环境状态，
Argo CD 执行同步，
生产使用灰度发布，
指标异常立即回滚。
```

