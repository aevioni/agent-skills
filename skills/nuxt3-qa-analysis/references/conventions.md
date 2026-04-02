# 项目规范参考

> 来源：CLAUDE.md + README.md，适用于本项目（Nuxt 3 + Vue 3 H5）

## 单文件长度限制

- 组件或页面 `.vue` 文件不应超过 **500 行**
- 超过 500 行时，应将独立的 UI 区块拆分为子组件，放入 `components/` 对应子目录
- 超过 600 行视为 high 风险，500~600 行视为 medium 风险

## 单文件组件结构

组件必须按以下顺序组织，不得颠倒：

```vue
<template>...</template>
<script setup lang="ts">...</script>
<style lang="scss">...</style>
```

**违规示例：**
- `<script>` 在 `<template>` 之前
- 使用 `<style scoped>` 但缺少 `lang="scss"`（可能需要核实项目是否统一要求 scss）
- 使用 Options API `export default {}` 而非 `<script setup>`

## 命名规范

### 组件名称与存放位置

- 所有组件统一存放在 `components/` 目录下，不得分散到其他目录
- 目录名使用 **kebab-case**，Nuxt 自动导入时将路径各段转换为 PascalCase 后拼接为组件名：
  - `components/alpha-beta/index.vue` → `<AlphaBeta />`
  - `components/game/card-list/index.vue` → `<GameCardList />`
  - `components/base/icon-font/index.vue` → `<BaseIconFont />`
- 文件名：统一使用 **kebab-case** 或 `index.vue`，**禁止** PascalCase 文件名
  - 正确：`index.vue`、`card-item.vue`
  - 违规：`GameCard.vue`、`CardList.vue`
- 模板中引用必须使用 PascalCase，不使用 kebab-case 形式（`<AlphaBeta />`，非 `<alpha-beta />`）

### 组件属性

- 模板中属性必须使用 kebab-case，如 `game-id`、`show-loading`
- 禁止驼峰属性：`gameId`（错误）→ `game-id`（正确）

### 文件/目录命名

- 页面文件（`pages/`）：遵循 Nuxt 文件路由规范，通常 kebab-case
- Composable：camelCase，如 `useGameList.ts`
- Store：camelCase，如 `app.ts`、`user.ts`
- 服务文件（`utils/services/`）：camelCase，按域分目录

## 图标使用

- **禁止**使用 `<i />` 标签引入图标（即使带 class）
- **必须**使用 `<KrIconFont />` 组件，例如：

```vue
<!-- 错误 -->
<i class="iconfont icon-home"></i>

<!-- 正确 -->
<KrIconFont name="home" />
```

## onMounted / onActivated 内的冗余检查

`onMounted` 和 `onActivated` 本身只在客户端运行，内部无需再判断 `import.meta.client`：

```typescript
// 错误：冗余检查
onMounted(() => {
    if (import.meta.client) {
        doSomething()
    }
})

// 正确：直接执行
onMounted(() => {
    doSomething()
})
```

## 顶层 Composable 调用限制

Nuxt 不允许在模块顶层调用 composable（即在 `utils/` 等非 Vue 组件/composable 上下文中调用）：

```typescript
// 错误：utils 文件顶层调用 composable
// utils/someHelper.ts
const config = useRuntimeConfig()  // 会在 SSR 时报错

// 正确：在 composable 或组件中使用
// composables/useHelper.ts
export function useHelper() {
    const config = useRuntimeConfig()
    return { config }
}
```

## 代码风格（ESLint 配置）

- 缩进：4 空格
- 引号：单引号
- 语句末尾：无分号
- 框架：ESLint + Nuxt 预设

## API 请求规范

### 使用 useAppFetch（优先）

`useAppFetch` 是推荐的 API 请求方式，自动处理：
- 认证头（Authorization）
- 自定义头（x-os、x-app-source、x-type）
- 请求去重（防止重复并发请求）
- SSR/CSR 兼容

```typescript
// 正确
const { data, pending, error } = await useAppFetch('/api/endpoint')

// 避免直接使用（除非有充分理由）
const data = await $fetch('/api/endpoint')
```

### 服务层组织

- 路径：`utils/services/{domain}/{feature}.ts`
- 按业务域分目录（mwp、model、v2 等）
- 每个文件导出具名方法
- 响应类型使用 `RespResult<T>`（包含 status、message、data）

```typescript
// utils/services/mwp/game.ts
export function getGameList(): Promise<RespResult<Game[]>> {
    return useAppFetch('/api_mwp/game/list')
}
```

## Pinia Store 规范

- 使用 Options API 模式（state / getters / actions），不使用 Setup Store
- 通过 `pinia-plugin-persistedstate` 持久化到 localStorage
- 不在 action 外部直接修改 state

```typescript
// stores/app.ts
export const useAppStore = defineStore('app', {
    state: () => ({
        latestGames: [] as Game[],
    }),
    getters: {
        hasGames: (state) => state.latestGames.length > 0,
    },
    actions: {
        updateLatestGame(game: Game) {
            this.latestGames = [game, ...this.latestGames].slice(0, 10)
        },
    },
    persist: true,
})
```

## 环境与配置

- 环境文件位于 `env/` 目录（`.env.dev`、`.env.test`、`.env.prod`、`.env.dev-prod`）
- 运行时配置通过 `nuxt.config.ts` 的 `runtimeConfig` 暴露
- 公共配置使用 `runtimeConfig.public.*`，敏感配置使用 `runtimeConfig.*`（仅服务端）
