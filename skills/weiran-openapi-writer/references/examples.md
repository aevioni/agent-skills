# Weiran OpenAPI Writer Examples

这份文件只放风格样例。优先学习“什么时候用”，其次才是“代码长什么样”。

## 1. POST + RequestBody(ref)

适用场景：
- 明确写接口
- 已有专用 Request 类
- body 契约清晰

```php
#[OA\Post(
    path: '/api/web/system/v1/auth/login',
    summary: '登录',
    tags: ['System'],
    requestBody: new OA\RequestBody(
        required: true,
        content: new OA\JsonContent(ref: AuthLoginRequest::class)
    ),
    responses: [
        new OA\Response(
            response: 200,
            description: '登录成功',
            content: new OA\JsonContent(ref: AuthLoginResponseBody::class)
        )
    ]
)]
public function login(...)
```

不要用于：
- 纯 query 接口
- 输入来源不明确的接口

---

## 2. GET + Parameters

适用场景：
- 列表、详情、校验、纯查询
- 参数来自 query
- 不存在 body 契约

```php
#[OA\Get(
    path: '/api/web/content/v1/content/detail',
    summary: '内容详细',
    tags: ['Content'],
    parameters: [
        new OA\Parameter(
            name: 'id',
            description: '内容 ID',
            in: 'query',
            required: true,
            schema: new OA\Schema(type: 'integer')
        ),
    ],
    responses: [
        new OA\Response(
            response: 200,
            description: 'success',
            content: new OA\JsonContent(ref: ContentContentDetailResponseBody::class)
        )
    ]
)]
public function detail(...)
```

不要用于：
- 实际从 body 读取字段的写接口

---

## 3. Multipart 上传

适用场景：
- 明确文件上传
- 已有上传 Request 类
- media type 为 `multipart/form-data`

```php
#[OA\Post(
    path: '/api/web/system/v1/upload/image',
    summary: '图片上传',
    requestBody: new OA\RequestBody(
        required: true,
        content: new OA\MediaType(
            mediaType: 'multipart/form-data',
            schema: new OA\Schema(ref: UploadImageRequest::class)
        )
    ),
    tags: ['System'],
    responses: [
        new OA\Response(
            response: 200,
            description: '图片上传',
            content: new OA\JsonContent(ref: BaseResponseBody::class)
        )
    ]
)]
public function image(...)
```

注意：
- 如果同一接口还支持 base64 或其他模式，应标记风险，不要假装只有一种输入方式。

---

## 4. Inline RequestBody

适用场景：
- 没有专用 Request 类
- 但写接口 body 字段少且明确
- 字段来源稳定

```php
#[OA\Post(
    path: '/api/web/system/v1/auth/renew',
    summary: '续期',
    requestBody: new OA\RequestBody(
        required: true,
        content: new OA\JsonContent(properties: [
            new OA\Property(property: 'device_id', description: '设备 ID, 参考 header x-id', type: 'string'),
            new OA\Property(property: 'device_type', description: '设备类型, 参考 header x-os', type: 'string'),
        ])
    ),
    tags: ['System'],
    responses: [
        new OA\Response(
            response: 200,
            description: '操作成功',
            content: new OA\JsonContent(ref: BaseResponseBody::class)
        )
    ]
)]
public function renew(...)
```

不要用于：
- 字段太多
- 来源不稳定
- 实际还依赖 header/middleware 契约但没有明确记录

---

## 5. Inline Response

适用场景：
- 没有现成 `*ResponseBody`
- 返回字段固定且很清晰
- 可以稳定抽象

```php
#[OA\Get(
    path: '/api/web/version/v1/version/latest',
    summary: '版本检测',
    tags: ['Version'],
    parameters: [
        new OA\Parameter(
            name: 'version',
            description: '版本号',
            in: 'query',
            required: true,
            schema: new OA\Schema(type: 'string')
        ),
    ],
    responses: [
        new OA\Response(
            response: 200,
            description: '获取版本成功',
            content: new OA\JsonContent(
                properties: [
                    new OA\Property(property: 'download_url', description: '下载地址', type: 'string'),
                    new OA\Property(property: 'description', description: '描述', type: 'string'),
                    new OA\Property(property: 'version', description: '版本', type: 'string'),
                    new OA\Property(property: 'is_upgrade', description: '是否需要强制更新', type: 'string'),
                ]
            )
        )
    ]
)]
public function latest(...)
```

不要用于：
- 返回结构分支很多
- 字段依赖运行时条件明显变化

---

## 6. Fallback to BaseResponseBody

适用场景：
- 没有专用 `ResponseBody`
- 无法稳定提炼响应结构
- 已排除更具体表达方式

```php
#[OA\Post(
    path: '/api/web/system/v1/auth/logout',
    summary: '退出登录',
    tags: ['System'],
    responses: [
        new OA\Response(
            response: 200,
            description: '操作成功',
            content: new OA\JsonContent(ref: BaseResponseBody::class)
        )
    ]
)]
public function logout(...)
```

使用前必须确认：
- 没有更具体的现成 schema
- 控制器返回结构确实不适合稳定抽象
- 没有被你忽略的 success envelope 分支
