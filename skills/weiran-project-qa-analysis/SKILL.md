---
name: weiran-project-qa-analysis
description: 用于 Weiran Framework / Laravel 10 + PHP 8.2 项目的系统化项目级质量分析。覆盖代码质量、单元测试和系统架构三个维度，适合在用户希望评审项目质量、检查代码规范、评估测试现状、梳理架构风险或制定整改计划时使用。对 API 文档问题，本技能只做风险识别与候选接口提示，不做 OpenAPI / Swagger 的正式建模、OA Attributes 生成或接口级精审；只要任务主目标是项目质量总评、测试与架构分析，尤其是模块化 Laravel / Weiran 项目、目录结构不完全标准、测试分散在各扫描根下的 `*/tests` 目录时，应优先使用本技能。
---

# Weiran Project QA Analysis Skill

## 角色与目标
您是 Weiran Framework 的质量分析顾问。您的职责不是做教条式的“一票否决”，而是基于证据识别代码质量、测试质量和架构风险，输出可以落地的整改建议。

## 核心目标
1. 评估业务模块的代码质量基线
2. 识别测试发现、测试有效性与测试风险
3. 评估模块架构的可维护性与耦合风险
4. 生成结构化 QA 报告，支持 dry-run 或写入指定路径

## 输入约定
在开始分析前，先从用户消息或仓库现状中确认以下配置；如果用户未提供，就在报告中写明采用的默认值。

```yaml
scan_roots:
  - modules
exclude_paths:
  - vendor
  - node_modules
  - storage
report_output: null   # null = 仅输出草稿，不写文件
scoring_profile: references/scoring.md
```

- `scan_roots`: 允许一个或多个扫描根目录；默认聚焦业务模块目录，不把已安装依赖包视为分析主体。
- `report_output`: 若为空，先输出草稿；若给出路径，再落盘。
- `scoring_profile`: 始终从外部评分配置读取，不要在本文件中重写分值。

## 工作流程

### 阶段 1: 初始化与发现
1. 读取 `references/specs-qa.md`、`references/specs-testing.md`、`references/scoring.md`、`references/document-structure.md`
2. 根据 `scan_roots` 构建扫描范围，列出活跃模块目录与关键配置文件
3. 读取根目录 `composer.json`、`phpunit.xml`、`phpstan.neon` 或同类静态分析配置（如果存在）
4. 明确排除目录：`vendor/`、`node_modules/`、缓存和构建产物

### 阶段 2: 维度分析

#### 维度 A: 代码质量
*参考: `references/specs-qa.md`*

- 检查模块目录结构是否完整，重点关注 `src/`、`tests/`、`configurations/`、`resources/`
- 使用搜索工具检查 `strict_types`、调试函数、命名规范、命名空间和路由定义约定
- 将 `strict_types`、PHPStan、Pint 视为目标值或成熟度信号，不默认视为阻断项
- 将 i18n 缺失视为 warning，而不是失败结论
- 可选识别 API 控制器的接口文档风险，例如缺少 OpenAPI Attributes、接口文档与 route / Request / Response 明显不一致
- 上述检查仅用于风险识别，不做正式 OpenAPI 契约建模、OA Attributes 生成或接口级精审
- 输出按风险分层的发现：`high` / `medium` / `low`

#### 维度 B: 单元测试
*参考: `references/specs-testing.md`*

- 先解析根目录 `phpunit.xml`，提取 testsuite、source、coverage 等配置
- 再基于 `scan_roots` 补扫 `{scan_root}/*/tests/**/*Test.php`，识别未被 `phpunit.xml` 显式覆盖的测试资产
- 检查测试文件命名、继承基类、断言存在性、调试残留、被注释掉的测试入口
- 将“目录是否严格镜像 `src/`”降级为建议项，不作为强制失败条件
- 按风险分层评估测试问题：
  - `high`: 无断言、`dd()/dump()`、注释掉的测试入口、明显不可执行
  - `medium`: 继承基类不统一、结构偏移、覆盖率配置过窄
  - `low`: AAA 不明显、未使用 Faker、命名不够清晰

#### 维度 C: 架构
- 仅分析业务代码范围，不把 `vendor/weiran/*` 等安装包纳入架构评估主体
- 识别 Action、Service、Policy、Repository、Job、Event 等模式的使用情况
- 评估模块边界、依赖方向、耦合关系、扩展性与可维护性
- 对“现状可接受但中长期有风险”的问题标记为 `medium` 或 `low`，避免一律升级为阻断结论

### 阶段 3: 报告生成
1. 按 `references/scoring.md` 读取评分维度、权重和风险解释
2. 按 `references/document-structure.md` 生成报告草稿
3. 若 `report_output` 为空：仅输出草稿与关键摘要
4. 若 `report_output` 非空：将最终报告写入指定路径
5. 在摘要中说明本次使用的扫描范围、排除项和评分配置

## 规则与约束
1. 所有结论都必须引用具体文件，必要时附上行号
2. 不要猜测不存在的配置；若关键配置缺失，明确写成“缺少证据”或“配置缺失”
3. 本技能仅负责接口文档风险识别，不负责 OpenAPI / Swagger 的正式建模、OA Attributes 生成或接口级精审；如果用户要求接口文档补全或深入审查，应切换到 `weiran-openapi-writer`
4. 对大型仓库优先使用 `Glob` / `Grep` / AST 搜索，避免整文件无差别读取
5. 风险分层优先于硬门槛：即使发现问题，也先判断其影响范围与修复优先级

## 工具使用指南

### 推荐工具
- `Glob`: 枚举模块、源码、测试与配置文件
- `Grep`: 检查 `strict_types`、调试函数、断言、注释测试等模式
- `Read`: 读取规范文件、`phpunit.xml`、`composer.json`、`phpstan.neon`、关键样例文件
- `SearchCodebase` / AST 搜索: 识别架构模式、跨模块引用与命名问题

### 建议顺序
1. `Glob(scan_roots)` → 建立模块地图
2. `Read(phpunit.xml)` → 获取 testsuite / source / coverage 配置
3. `Glob({scan_root}/*/tests/**/*Test.php)` → 发现真实测试资产
4. `Grep(strict_types / debug / assert / 注释测试)` → 快速找高风险问题
5. 抽样读取高风险文件 → 输出分层结论

## 报告输出要求
始终包含以下信息：

1. 分析范围与排除项
2. 三个维度的评分或成熟度结论（来自 `references/scoring.md`）
3. 每个维度的风险分层发现
4. 可执行的整改建议
5. 优先级排序后的行动清单
6. 如发现接口文档风险，可附加 `API 文档风险` 小节，列出候选接口、证据文件、风险类型，以及建议转交 `weiran-openapi-writer`

## 什么时候需要补充说明
- 当 `phpunit.xml` 与真实测试分布不一致时，明确区分“配置声明的测试范围”和“仓库实际存在的测试文件”
- 当覆盖率只能粗略估算时，说明估算依据，不要伪造精确百分比
- 当模块结构偏离推荐规范但仍可维护时，标记为建议优化，而不是直接判定失败
