# 项目质量规范 (Quality Assurance)

## 1. 框架基础定义

| 关键项 | 定义 |
|:---|:---|
| **框架版本** | Laravel 10.x + PHP 8.2 |
| **扫描根** | 由技能输入中的 `scan_roots` 决定，默认聚焦业务模块目录 |
| **Strict Mode** | 推荐启用 `declare(strict_types = 1);`，作为质量目标值 |
| **PSR** | 推荐遵循 PSR-12 编码规范 |
| **PHPStan** | 推荐配置 PHPStan 并持续提升规则等级 |
| **Pint** | 推荐使用 Laravel Pint 或等效格式化工具 |

## 2. 目录结构规范

### 2.1 推荐模块结构
推荐模块（`{scan_root}/{module}`）包含以下核心目录：
- `src/` (源码)
- `tests/` (测试)
- `configurations/` (配置)
- `resources/` (资源)

目录缺失不必然意味着失败，但应在报告中结合模块职责评估风险。

### 2.2 命名空间映射
- `modules/user/src` -> `namespace User;`
- `modules/user/tests` -> `namespace User\Tests;`

## 3. 编码规范 (Coding Standards)

### 3.1 命名约定
- **Class**: `PascalCase` (e.g., `UserController`)
- **Method**: `camelCase` (e.g., `getUserInfo`)
- **Variable**: `camelCase` (e.g., `isAvailable`)
- **Constant**: `UPPER_CASE` (e.g., `MAX_RETRY`)
- **Interface**: 以 `Interface` 结尾
- **Trait**: 以 `Trait` 结尾

### 3.2 特殊文件命名
- **Event**: `*Event.php`
- **Listener**: `*Listener.php`
- **Job**: `*Job.php`
- **Policy**: `*Policy.php`
- **Resource**: `*Resource.php`

### 3.3 路由与控制器
- **Controller**: 建议以 `Controller` 结尾
- **Routes**:
    - 推荐位置: `src/Http/Routes/*.php`
    - 建议避免在路由文件中使用 `namespace` 参数
    - 命名约定可按项目现状评估，不做硬性失败项

### 3.4 国际化 (i18n)
- **Validation**: 建议存在 `resources/lang/zh/validation.php`，并尽量与 `en` 目录保持同步
- **Keys**: 建议使用 snake_case
- **风险等级**: i18n 缺失默认记为 `warning`，除非它已直接影响运行时行为

## 4. 风险信号 (Risk Signals)

以下模式在生产代码中应重点关注：

1. **Debug Functions**:
   - `dd(...)`
   - `dump(...)`
   - `var_dump(...)`
   - `print_r(...)`
2. **Unsafe Code**:
   - `eval(...)`
   - `exec(...)`（除非存在明确封装与边界控制）
3. **Legacy**:
   - 使用 `@deprecated` 标记的方法/类但没有替代说明

这些问题应根据出现位置和影响范围分成 `high` / `medium` / `low`，而不是一律视为阻断。

## 5. AI Analysis Rules (Regex Patterns)

| 检查项 | 推荐 Regex 模式 | 说明 |
|:---|:---|:---|
| **Strict Types** | `^declare\s*\(\s*strict_types\s*=\s*1\s*\);` | 目标值；除 view/config 外优先检查源码文件 |
| **Forbidden Debug** | `(dd|dump|var_dump|print_r)\s*\(` | 扫描所有 `.php` 文件 |
| **Namespace** | `namespace\s+[A-Z][a-zA-Z0-9\\]*;` | 验证命名空间格式 |
| **Route Prefix** | `['\"]prefix['\"]\s*=>\s*['\"][a-z0-9\-\/]+['\"]` | 提取路由前缀 |
| **Class Name** | `class\s+[A-Z][a-zA-Z0-9]*` | 验证类名 PascalCase |

## 6. 快速检查清单 (Checklist)

- [ ] **Strict Types**: `src/` 下的重要 PHP 文件是否启用严格模式？
- [ ] **No Debug**: 是否残留 `dd()`、`dump()` 等高风险调试代码？
- [ ] **Config Depth**: `configurations/` 下是否出现难以维护的深层嵌套？
- [ ] **I18n**: `resources/lang/zh` 和 `en` 是否明显失衡？若缺失，是否已影响用户体验？
- [ ] **Naming**: 是否存在明显偏离约定且影响理解或维护的命名问题？
