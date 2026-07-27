# 任务编号与项目约定

> 阶段 2 产出 dev-tasks.md、阶段 4 写产出物时读本文件。

## 任务编号与产出物命名（强制）

任务统一使用 **`X.Y`** 格式编号（X=任务组, Y=组内子任务，从 1 起始）。禁止 T1、0.1、波次1、T5R 等非标格式。

编号在阶段 2 产出 dev-tasks.md 时确定，全流程所有产出物文件名必须引用同一编号：

| 产出物 | 文件名 |
|--------|--------|
| progress.md 任务行 | `{X.Y} {模块名} · {任务标题}` |
| plan | `.task/plans/{模块名}-{X.Y}.md` |
| done 报告 | `.task/done/{模块名}-{X.Y}.md` |
| CR 报告 | `.task/review/{模块名}-{X.Y}.md` |

## 项目约定

- **单体仓库**：默认已在项目仓库根目录。产出物存于 `{讨论根目录}{域}/{需求名}/.task/`（目录结构见 `protocols/templates.md`）
- **需求级参考文档**：`{讨论根目录}{域}/{需求名}/docs/`，用户在 `/devops-workflow start` 时放入（业务说明、接口文档、原始需求材料等），阶段 2 分析和阶段 4 编码时 agent 会读取
- **前置检查（强制）**：见 `protocols/preflight.md`，所有检查项在命令执行前自动逐项验证
- **工作范围**：每个任务在 dev-tasks.md 中标注工作范围目录（一个或多个），由阶段 2 analyst 确定。executor / code-reviewer / verifier 均以此为工作边界，不假设特定目录结构
- **架构文档** `docs/{模块名}/`（overview/business/contracts/flows）+ `docs/cross-module.md`（或 `docs/cross-service-guide.md`），由 `arch-analyzer` 产出；缺失则先提示跑 arch-analyzer
- **验收基线（强制）**：按项目 `.claude/rules/` 中定义的测试命令执行模块级单测通过 + 按项目规则执行语法/编译检查通过
- **静态分析**：是否纳入 DoD/验收由项目 `.claude/rules/` 决定，workflow 不强制
- **单仓库单分支**：所有模块共用一条 feature 分支，无独立分支/PR；具体命令按项目配置
