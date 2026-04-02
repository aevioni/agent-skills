# QA 检查项清单

## 维度 A：组件规范

### A1 单文件组件结构顺序（high）
- [ ] 检查是否所有 `.vue` 文件都遵循 `<template>` → `<script setup lang="ts">` → `<style lang="scss">` 顺序
- 搜索方式：读取文件，检查三个根节点的出现顺序
- 违规标记：`high`（结构混乱影响可读性和团队协作）

### A2 组件命名与存放位置（medium）
- [ ] 组件必须存放在 `components/` 目录下
- [ ] 目录名和文件名统一 kebab-case，禁止 PascalCase 文件名（如 `GameCard.vue`）
- [ ] 模板引用使用 PascalCase（`<AlphaBeta />`，非 `<alpha-beta />`）
- 搜索方式：`Glob(components/**/*.vue)` 后检查文件名是否含大写字母（`index.vue` 除外）
- 命名规则详见 `conventions.md` → 组件名称与存放位置

### A3 属性命名（medium）
- [ ] 组件属性在模板中是否为 kebab-case
- 搜索方式：`Grep` 组件标签上的属性，检查是否有 camelCase 属性（如 `gameId=`）
- 注意：原生 HTML 属性（`class`、`id`、`style`）不在此范围

### A4 图标用法（medium）
- [ ] 是否存在 `<i class=` 或 `<i ` 形式的图标引用
- 搜索命令：`Grep("<i ", pages/ components/)`
- 违规示例：`<i class="iconfont icon-home"></i>`
- 正确示例：`<KrIconFont name="home" />`

### A5 单文件长度（medium）
- [ ] 组件或页面单文件（`.vue`）是否超过 500 行
- 搜索方式：读取 `Glob(pages/**/*.vue)` 和 `Glob(components/**/*.vue)` 结果，对每个文件检查行数
- 违规标记：超过 500 行应考虑拆分为子组件，降低单文件复杂度
- 提示：优先关注超过 600 行的文件作为 high 风险，500~600 行作为 medium

---

## 维度 B：SSR 兼容性

### B1 utils 中顶层 Composable 调用（high）
- [ ] `utils/` 目录下是否有文件在模块顶层调用 composable（如 `useRuntimeConfig()`、`useState()`、`useNuxtApp()`）
- 搜索命令：`Grep("use[A-Z]", utils/)` 后筛选顶层调用（非函数体内）
- 原因：Nuxt 要求 composable 只能在 Vue 组件或 composable 函数内调用，顶层调用在 SSR 时会报错

### B2 import.meta.server/client 滥用（medium）
- [ ] 检查 `import.meta.server` 和 `import.meta.client` 的使用场景是否合理
- 搜索命令：`Grep("import.meta.server|import.meta.client", pages/ components/ composables/)`
- 评估：在 `onMounted` 内使用 → low（冗余）；在 composable/utils 顶层无条件使用 → medium

### B3 服务端/客户端代码分离（medium）
- [ ] 只能在客户端运行的代码（window、document、localStorage）是否有适当的 guard
- 搜索命令：`Grep("window\.|document\.|localStorage", composables/ utils/)`
- 检查是否在插件文件命名中使用了 `.client.ts` / `.server.ts` 约定

### B4 Hydration Mismatch 风险（medium）
- [ ] 服务端/客户端渲染内容是否可能不一致（如依赖 `Date.now()`、随机数、`window` 对象）
- 重点检查 `pages/` 中直接使用这些 API 的代码

---

## 维度 C：状态管理（Pinia）

### C1 Store 模式（medium）
- [ ] Store 是否使用 Options API 模式（`defineStore('name', { state, getters, actions }`）
- [ ] 是否存在 Setup Store 写法（可能与项目规范不一致）
- 搜索命令：`Glob(stores/*.ts)` 后读取检查

### C2 持久化配置（low）
- [ ] 需要持久化的 Store 是否配置了 `persist: true` 或细粒度持久化
- [ ] 是否有敏感数据（token 等）被意外持久化到 localStorage

### C3 State 直接修改（medium）
- [ ] 是否存在在 action 外部通过 `store.xxx = value` 直接修改 state 的写法
- 搜索方式：检查组件中对 store 实例属性的直接赋值

### C4 Store 间依赖（low）
- [ ] Store 间是否存在循环依赖
- [ ] 跨 Store 调用是否通过 action 而非直接访问

---

## 维度 D：API 调用模式

### D1 useAppFetch 使用（high）
- [ ] 是否有组件/页面绕过 `useAppFetch` 直接使用 `$fetch` 或 `useFetch`
- 搜索命令：`Grep("useFetch\|\\$fetch", pages/ components/ composables/)`
- 注意：`useAppFetch` 本身内部使用 `useFetch` 是正常的，需排除 `composables/useAppFetch.ts` 本身

### D2 服务层组织（medium）
- [ ] `utils/services/` 是否按域分目录（mwp/、model/、v2/ 等）
- [ ] 服务方法是否有 TypeScript 返回类型
- [ ] 是否存在 API 调用散落在组件内而非服务层

### D3 响应类型（low）
- [ ] API 响应是否使用 `RespResult<T>` 或等价的类型定义
- [ ] 是否存在 `any` 类型的响应处理

### D4 请求去重（low）
- [ ] 是否有场景中同一请求被并发触发多次（可通过 `useAppFetch` 的去重机制解决）

---

## 维度 E：代码风格

### E1 格式规范（low）
- [ ] 缩进是否为 4 空格（非 2 空格、非 Tab）
- [ ] 是否使用单引号（非双引号）
- [ ] 是否有多余的分号
- 优先通过 `pnpm lint` 检查，无需手动逐文件审查

### E2 TypeScript 类型覆盖（medium）
- [ ] 组件 props 是否有类型定义（`defineProps<{...}>()`）
- [ ] Composable 返回值是否有类型
- [ ] Store state 是否有 TypeScript 类型
- 搜索命令：`Grep("defineProps({|: any", pages/ components/)`

### E3 调试残留（medium）
- [ ] 是否有 `console.log`、`debugger`、未删除的 `vConsole` 调用
- 搜索命令：`Grep("console\.log|debugger", pages/ components/ composables/ stores/)`
- 注意：`cons.ts` 等日志工具文件排除在外

### E4 注释质量（low）
- [ ] 是否有大量注释掉的代码块
- [ ] TODO/FIXME 注释是否有跟进计划

### E5 页面级别 any 类型（medium）
- [ ] `pages/` 下的 `.vue` 文件中是否存在显式或隐式的 `any` 类型
- 搜索命令：`Grep(": any|as any|<any>|any\[\]", pages/)`
- 违规示例：`const data: any = await fetchSomething()`、`(event as any).target`
- 建议：页面级别的 `any` 类型应当移除，替换为精确的 TypeScript 类型或 `unknown`，防止类型错误在运行时才暴露
- 违规标记：`medium`（直接影响类型安全，但通常不导致立即崩溃）

---

## 维度 F：测试质量

### F1 单元测试覆盖率（medium）
- [ ] 运行 `pnpm test:coverage` 检查当前覆盖率报告
- [ ] 整体行覆盖率是否达到 80% 以上
- [ ] 核心工具函数（`utils/`）、composable（`composables/`）是否有对应测试文件
- 搜索命令：`Glob(tests/**/*.test.ts)` 枚举已有测试文件，与 `utils/`、`composables/` 文件数量对比
- 建议：单元测试覆盖率应达到 80% 以上；优先补全业务逻辑复杂的 composable 和工具函数；页面组件可通过 E2E 测试补充覆盖
- 违规标记：覆盖率低于 60% 标记 `high`；60%~80% 标记 `medium`；80% 以上视为达标
