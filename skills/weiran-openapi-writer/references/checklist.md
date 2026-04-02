# Weiran OpenAPI Writer Checklist

这份清单用于复杂场景或 dry-run 场景下的详细复核。主技能文件定义主流程，这里定义更细的验收标准。

## 阶段 1：路由与 Path 验收

必须确认：

- route 文件已定位
- route method 已确认
- route path 已确认
- Controller::method 与 route 一致
- `RouteServiceProvider` 中 prefix 来源已确认
- 完整 path = prefix + route path

复核问题：

- 当前 path 是否可追溯到源代码
- 当前 OA verb 是否可追溯到 route method
- 若 route 为 `any`，是否给出了充分的行为依据

## 阶段 2：输入来源验收

必须检查：

- 方法签名
- `input()` / `$request->input()`
- `file()` / `Request::file()`
- `x_header()`
- path/query/body/multipart

复核问题：

- 输入位置是否与控制器真实行为一致
- 是否遗漏 header 契约
- 是否误把 body 字段写成 query parameters
- 是否存在 mixed transport

## 阶段 3：Request 发现验收

查找顺序是否遵循：

1. 方法签名
2. 顶部 `use`
3. 容器解析
4. 模块内真实文件
5. 命名推导

复核问题：

- 当前 Request 类是否真的是接口契约来源
- 是否只是命名猜测
- 没有专用 Request 时，是否给出了合理的 inline body 方案

## 阶段 4：Response 发现验收

必须检查：

- 已有 `*ResponseBody`
- 模块 `OpenApi` 目录
- 模块相关 `ApiV1` 目录
- 控制器返回的稳定字段
- 是否存在多个 success envelope

复核问题：

- 是否过早 fallback 到 `BaseResponseBody`
- 是否忽略了更具体的返回结构
- 是否存在分支响应没有被记录

## 阶段 5：生成计划验收

生成前必须已有计划表，至少包括：

- imports
- OA verb
- path
- tags
- request 方案
- response 方案
- fallback 理由

复核问题：

- 计划是否足够让人工 review
- 计划是否能被追溯到前面阶段证据
- 是否仍存在关键不确定项未标注

## 阶段 6：语义验证验收

必须逐项检查：

- `route method == OA verb`
- `prefix + route path == OA path`
- 参数位置与真实读取方式一致
- 引用类真实存在
- header 契约未遗漏
- response 与真实行为没有明显冲突
- fallback 有证据

## 高风险场景专项清单

### Route::any
- 是否解释了 GET / POST 的选择依据
- 若无依据，是否停止

### Header 契约
- 是否存在 `x_header(...)`
- 是否明确写入风险或契约说明

### Mixed transport
- 同一字段是否支持 body / file / base64 多模式
- 是否误简化为单一 requestBody
- 如果同一字段既支持 `file()` 又支持 `input()`，是否已明确标记为 `mixed transport`
- 是否评估过该接口应继续文档化、拆成多个场景，还是直接停止并人工确认

### Multipart + scalar
- 除文件外，是否还有关键标量字段
- 是否一起进入 schema 设计

### Alternate response envelopes
- 是否存在普通 `Resp::success(...)` 之外的专用响应结构
- 是否存在编辑器、自定义客户端或第三方协议专用 JSON 包裹
- 是否记录了每种响应结构的触发条件
- 如果无法用一个稳定 schema 准确覆盖，是否停止并人工确认

### 多成功响应
- 是否存在不同 success envelope
- 是否需要人工确认或更复杂 schema

### 动态解析 Request
- 方法体是否通过容器解析真实 Request
- 是否把这个线索纳入最终结论

### 过早 fallback
- 是否已经排除所有更具体 schema
- 控制器返回是否其实足够稳定

## Dry-run 最终输出模板

建议 dry-run 最终输出固定为：

1. 目标接口
2. 路由与 Prefix 证据
3. 输入来源判定
4. Request / Response 发现结果
5. 拟生成草稿
6. 风险与未确认项

## 高风险上传接口默认决策

如果一个上传接口同时满足以下任一条件：

- 同一字段既可能来自 `file()`，也可能来自 `input()`
- 通过显式分支支持 `form` / `base64` / 其他多输入模式
- 存在普通成功响应之外的编辑器或协议专用 JSON 响应

则默认不要把它视为“普通 multipart 接口”。

复核时必须确认：

- 是否已明确标记为 `mixed transport`
- 是否已明确标记存在 `alternate response envelopes`
- 是否错误地 fallback 到 `BaseResponseBody`
- 是否只能通过忽略某个分支，才能勉强拼出统一文档

如果以上任一问题无法被稳定解决，默认结论应为：停止并人工确认，而不是继续自动生成最终 OpenAPI 文档。

## OpenAPI 工具链建议

如果环境允许，后续建议执行：

1. 生成 OpenAPI spec
2. validate / lint
3. 必要时 bundle
4. Swagger UI / Swagger Editor 预览

dry-run 中不一定执行，但应明确建议。
