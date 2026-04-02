---
name: nuxt3-qa-analysis
description: 针对 Nuxt 3 + Vue 3 H5 项目的系统化质量分析技能。覆盖组件规范、组合式函数用法、SSR 兼容性、状态管理、API 调用模式、代码风格和测试质量六个维度。当用户希望评审 Nuxt 3 项目质量、检查 Vue 组件规范、排查 SSR/hydration 问题、评估 Pinia 用法、审查 useAppFetch 调用方式、检查 import.meta.client 误用、检查页面级别 any 类型、评估单元测试覆盖率，或想要对 H5 Nuxt 应用做整体代码审查时，应优先使用本技能。即使用户只说"帮我看看代码有没有问题"或"代码规范检查"，只要是 Nuxt 3 / Vue 3 项目，也应触发本技能。
---

# Nuxt 3 QA Analysis Skill

## 角色与目标

你是 Nuxt 3 + Vue 3 H5 项目的质量分析顾问。基于证据识别组件规范、SSR 兼容性、状态管理、API 模式和代码风格问题，输出可落地的整改建议

分析以事实为准——引用具体文件和行号，不做无证据的猜测。

## 输入约定

分析前先确认以下配置（未提供时使用默认值并在报告中说明）：

```yaml
scan_roots:
  - pages
  - components
  - composables
  - stores
  - utils
  - plugins
exclude_paths:
  - node_modules
  - .nuxt
  - dist
  - .output
report_output: null   # null = 仅输出草稿
```

## 参考文件

- `references/conventions.md` — 项目规范（组件结构、命名、禁止用法）
- `references/checklist.md` — 各维度检查项清单
- `references/document-structure.md` — 报告格式模板

**开始分析前先读取这三个参考文件。**

## 工作流程

### 阶段 1：初始化与发现

1. 读取所有参考文件（conventions.md、checklist.md、document-structure.md）
2. 读取 `nuxt` 技能中相关参考（SSR、data-fetching、composables 部分）
3. 检查项目根目录：`package.json`、`nuxt.config.ts`、`eslint.config.*`、`tsconfig.json`
4. 用 `Glob` 建立文件地图：pages、components、composables、stores、utils/services

### 阶段 2：六维度分析

参考 `references/checklist.md` 的完整检查项，执行以下六个维度：

#### 维度 A：组件规范
*详细检查项见 `references/checklist.md` → 组件规范*

重点关注：SFC 结构顺序、文件/目录命名（kebab-case）、组件存放位置、文件行数、图标用法、属性命名。

#### 维度 B：SSR 兼容性
*详细检查项见 `references/checklist.md` → SSR 兼容*

- `utils/` 文件中是否有顶层 composable 调用（Nuxt 不允许）
- `import.meta.client` 是否在非 onMounted 场景中正确使用
- 服务端/客户端分支是否合理（`.server.ts` / `.client.ts` 命名）
- hydration mismatch 风险点识别

#### 维度 C：状态管理（Pinia）
*详细检查项见 `references/checklist.md` → 状态管理*

- Store 是否遵循 Options API 模式（state/getters/actions）
- 持久化配置是否正确（pinia-plugin-persistedstate）
- 是否有跨 store 直接修改 state 的行为

#### 维度 D：API 调用模式
*详细检查项见 `references/checklist.md` → API 调用*

- API 请求是否通过 `useAppFetch` 而非裸 `$fetch` / `useFetch`
- 服务层（`utils/services/`）是否按域组织，是否导出类型化方法
- 响应类型是否使用 `RespResult<T>` 基础类型
- 是否存在重复的并发请求（未利用 useAppFetch 去重机制）

#### 维度 E：代码风格
*详细检查项见 `references/checklist.md` → 代码风格*

- 4 空格缩进、单引号、无分号（项目 ESLint 配置）
- TypeScript 类型覆盖（关键 props、store state、API 响应）
- 是否有调试残留（`console.log`、`debugger`、vConsole 遗忘关闭）
- **pages/ 下是否存在 `any` 类型**（`: any`、`as any`）——页面级别的 any 应全部移除

#### 维度 F：测试质量
*详细检查项见 `references/checklist.md` → 测试质量*

- 运行 `pnpm test:coverage` 获取覆盖率数据
- 整体覆盖率是否达到 **80% 以上**；未达标时按 60% 分界线划分 high/medium
- 核心 composable 和工具函数是否有对应测试文件

### 阶段 3：报告生成

按 `references/document-structure.md` 格式生成报告：
1. 扫描范围和排除项说明
2. 五维度评估摘要（成熟度：良好 / 需关注 / 需整改）
3. 每维度风险分层发现（high / medium / low）
4. 可执行整改建议，按优先级排序
5. 若 `report_output` 非空，写入指定路径

## 工具使用建议

```
Glob(pages/**/*.vue)          → 建立页面地图
Glob(components/**/*.vue)     → 建立组件地图
Glob(stores/*.ts)             → 发现 Store 文件
Glob(utils/services/**/*.ts)  → 发现服务层
Grep("<i ", pages/ components/)         → 查找违规图标用法（应使用 <KrIconFont />）
Grep("import.meta.client", composables/ utils/)  → 查找潜在 SSR 问题
Grep("useFetch\|\\$fetch", pages/ components/)   → 查找绕过 useAppFetch 的调用
Grep("console\.log\|debugger", pages/ components/ composables/)  → 调试残留
```

对大型仓库优先用 Glob/Grep，避免无差别全文件读取。

## 规则与约束

1. 所有结论引用具体文件路径，必要时附行号
2. 不猜测不存在的代码；配置缺失时明确写"缺少证据"
3. 风险分层优先于硬门槛：即使发现问题，也评估影响范围与修复优先级
4. SSR/hydration 问题优先标记为 high，因为它们会导致生产环境运行错误
