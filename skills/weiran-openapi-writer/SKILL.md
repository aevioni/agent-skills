---
name: weiran-openapi-writer
description: 为 Weiran Framework / Laravel 的 API 控制器生成或 dry-run 审查 OpenAPI（Swagger）PHP Attributes 文档。只要用户提到给 Weiran 接口补 Swagger、OpenAPI、OA Attributes、接口文档、控制器文档，检查某个 API 接口文档是否正确，或基于 QA 报告继续深挖某个接口的文档风险，就应优先使用本技能。它专注接口级证据链分析与文档建模，适用于路由驱动、Request/ResponseBody 复用、Header 契约识别、上传接口与多种输入来源分析，不适用于项目级代码质量、测试覆盖或架构总评。
---

# Weiran OpenAPI Writer

你是一名熟悉 Weiran Framework 与 Laravel 的 API 文档开发者。你的任务是基于真实 route、controller、request、response 结构，为接口生成或 dry-run 审查 OpenAPI PHP 8 Attributes。

目标不是尽快拼出一段 `#[OA\...]`，而是给出一个可审计、可验证、可复核的结果。

## 工作模式

### 生成模式
适用于用户明确要你补全或生成 OpenAPI Attributes。

### Dry-run 模式
适用于用户要求只读分析、试跑技能、评估文档方案、不修改代码。

在 dry-run 模式下：
- 不编辑任何文件
- 不插入任何 Attribute
- 仍然完整执行分析、规划、验证流程
- 最终只输出证据、草稿、风险与建议

如果用户明确说“不要修改代码”，必须使用 dry-run 模式。

## 输入补充

- 可接收来自 `weiran-project-qa-analysis` 的候选接口、风险项或文件线索作为起点
- 但无论输入来源如何，仍必须重新完成 route、prefix、输入来源、Request / Response 的证据链验证
- 不得因为上游 QA 报告已经给出初步判断，就跳过本技能要求的分析、规划与验证流程

## 输出契约

每次执行都必须输出以下 6 个部分：

### 1. 目标接口
- Controller::method
- 模块
- 当前模式：dry-run / 生成

### 2. 路由证据
- route 文件
- route method
- route path
- prefix 来源
- 完整 path

### 3. 输入来源判定
- query / body / multipart / header / mixed transport
- 判定依据

### 4. Request / Response 发现结果
- 找到哪些候选类
- 最终采用哪个
- 为什么采用

### 5. 拟生成草稿
- imports
- `#[OA\...]`

### 6. 风险与未确认项
- 已验证项
- 未验证项
- 是否需要人工确认

如果这 6 部分没有完整输出，就不算完成。

## 工作主线

你必须按以下顺序工作：

1. 从 route 开始确认目标接口
2. 读取 `RouteServiceProvider` 提取 prefix 与完整 path
3. 分类真实输入来源
4. 发现并评估 Request / Response 候选
5. 先输出生成计划，再给 OA 草稿
6. 做源码语义验证
7. 明确风险与未确认项

不要从命名直接猜，不要跳过证据链，不要只输出最终 `#[OA\...]`。

## 阶段规则

### 阶段 1：确认 route 与 path

永远先从 route 开始。搜索：
- `[modules|vendor/weiran]/**/src/Http/Routes/api*.php`

以 route 中定义的 Controller::method 为准，再读取对应模块的 `RouteServiceProvider.php` 确认 prefix，最后拼出完整 path。

约束：
- Controller namespace 中必须包含 `Api`
- 如果不包含，立即停止并说明当前技能仅支持 API 控制器

如果遇到 `Route::any`：
- 不要机械照搬
- 必须说明为什么最终选 GET / POST
- 如果判断不稳，立即停止并要求人工确认

### 阶段 2：识别真实输入来源

你必须检查：
- 方法签名参数
- `input()` / `$request->input()`
- `file()` / `Request::file()`
- `x_header()` / header
- path / query / body / multipart

然后给出明确分类：
- `query parameters`
- `json requestBody`
- `multipart/form-data`
- `mixed transport`
- `header-derived contract`

规则：
- 不能因为看到了 Request 类，就跳过真实输入分析
- 关键 header 不能漏
- 同一字段若既可能来自 body 又可能来自 file，标记为 `mixed transport`

### 阶段 3：发现 Request / Response

#### Request 查找顺序
1. 方法签名
2. 顶部 `use`
3. 方法体中的 `app(FooRequest::class)` / `resolve(...)`
4. 模块内真实 `*Request.php`
5. 最后才是命名推导

#### Response 查找顺序
1. 顶部 `use` 的 `*ResponseBody`
2. 模块内 `OpenApi` 目录
3. 模块内相关 `ApiV1` 目录
4. 最后才是命名推导

#### 选择规则
- 有明确 Request 类且契约匹配：优先复用
- query 接口：优先 `parameters`
- 写接口且字段来自 body：优先 `requestBody`
- 上传接口：优先 `multipart/form-data`
- 返回结构固定且明确：允许内联 response schema
- 只有真的无法稳定表达时，才允许 fallback 到 `BaseResponseBody`

重要：
`BaseResponseBody` 不是默认答案。只有当你证明没有更具体的已有 schema，且控制器返回结构也无法稳定总结时，才允许 fallback。

### 阶段 4：生成前计划

在输出最终 `#[OA\...]` 之前，先给出计划：

- imports
- OA verb
- path
- tags
- `parameters` 还是 `requestBody`
- request schema 来源
- response schema 来源
- fallback 理由（如果有）

如果计划说不清楚，就不要生成。

### 阶段 5：验证

你必须检查：
- `route method == OA verb`
- `prefix + route path == OA path`
- 参数位置与真实读取方式一致
- 引用类真实存在
- header 契约没有遗漏
- 没有错误地提前 fallback
- response 与控制器真实行为没有明显冲突

如果当前是 dry-run：
- 不修改文件
- 给出草稿和验证结论
- 说明建议继续做哪些 validate / lint / UI 预览

如果当前是生成模式：
- 生成后也要给出验证结论，而不是只贴代码

## 高风险场景

遇到以下情况必须格外谨慎：

- `Route::any`
- 关键输入来自 header
- multipart + 普通字段混合
- 同一字段支持 file / base64 / string 多模式
- 同一接口有多种成功响应结构
- 方法签名不是具体 Request，但方法体里动态解析了 Request
- 返回结构稳定，却有人想直接 fallback 到 `BaseResponseBody`

### Mixed Transport + Alternate Response 特殊规则

当接口同时满足以下任一条件时，不要继续把它当作普通 `multipart/form-data` 或普通 `json requestBody` 接口处理：

1. 同一字段既可能来自 `$request->file(...)` / `Request::file(...)`，也可能来自 `$request->input(...)` / `input(...)`
2. 接口通过显式分支支持多种输入模式，例如 `form` / `base64` / `file` / `string`
3. 接口存在非标准成功响应包裹，例如普通 `Resp::success(...)` 之外，还存在编辑器、自定义前端或第三方协议专用 JSON 结构
4. 接口在不同分支下返回的 success / error payload 结构明显不同，且无法用一个稳定 schema 清晰表达

命中以上条件时，你必须：

- 明确标记该接口为 `mixed transport`、`alternate response envelopes` 或两者同时存在
- 输出具体证据：哪些代码路径走 file，哪些代码路径走 body，哪些代码路径走特殊响应
- 不要直接给出“看起来已经完整”的最终 OpenAPI 文档
- 优先输出风险说明、待确认项，以及你认为可选的建模方案
- 如果无法稳定收敛为一个准确的 request/response 契约，立即 STOP，并要求人工确认

### Mixed Transport 场景的默认决策

如果你发现同一字段既可来自 file 又可来自 input，默认不要把它视为“纯 multipart”。

你必须先判断：

- 这是否是同一个接口的两种正式输入协议
- 是否应拆成两个文档化场景
- 是否必须用人工确认来决定最终文档策略

如果无法稳定判断，停止生成。

### Alternate Response Envelope 场景的默认决策

如果同一接口存在多个 success envelope，例如：

- 标准 `Resp::success(...)`
- 特定调用方专用结构（如编辑器、自定义 SDK、第三方平台协议）
- 不同条件下字段层级明显不同的 JSON

则不能默认 fallback 到 `BaseResponseBody`，也不能假装这些分支不存在。

你必须：

- 记录每种响应结构的触发条件
- 判断是否存在“主响应”与“特殊响应”
- 如果无法稳定选定主文档表达方式，停止并要求人工确认

复杂场景下继续读取 `references/checklist.md`。

需要参考风格模板时继续读取 `references/examples.md`。

## 必须停止并人工确认

出现以下任一情况时，停止：

- route 找不到
- prefix 找不到
- `Route::any` 无法稳定判断
- 有多个 Request 候选且无法确认
- 有多个 Response 候选且无法确认
- 有多种 success response 无法稳定建模
- 输入来源混合且无法稳定归类
- 同一字段既支持 `file()` 又支持 `input()`
- 接口通过显式分支支持 `form` / `base64` / 其他多输入模式
- 存在编辑器、自定义客户端或第三方协议专用返回结构
- 普通成功响应与特殊成功响应无法被一个稳定 schema 准确覆盖
- 你只能通过忽略某个分支，才能勉强生成一个“统一”的 OpenAPI 文档
- header / middleware 契约明显重要但无法准确表达
- fallback 到 `BaseResponseBody` 缺少充分证据

## 成功标准

只有当以下条件同时满足时，才算成功：

- 已确认 route、prefix、path、verb
- 已完成输入来源分类
- 已找到并说明 Request / Response 的依据
- 已给出可复核草稿
- 已完成语义验证
- 已明确风险与未确认项
