# Weiran Framework 单元测试规范

## 1. 核心配置要求

### 1.1 基础环境
- **测试框架**: PHPUnit 10.x 或兼容版本
- **配置文件**: 优先读取根目录 `phpunit.xml`
- **补充发现**: 同时扫描 `{scan_root}/*/tests/**/*Test.php` 以发现真实测试资产

### 1.2 配置检查重点
- `testsuites` 是否声明了实际测试入口
- `source` / `coverage` 是否覆盖了核心业务代码，而不是只覆盖局部模块
- 测试环境变量是否合理（如 `APP_ENV=testing`）

## 2. 测试组织建议

### 2.1 目录结构
测试目录不要求严格镜像 `src/`，但应满足以下目标：
- 能从命名或目录上看出测试对象
- 能被 `phpunit.xml` 或补充 glob 发现
- 不因随意堆叠而影响维护

### 2.2 命名规范

| 对象类型 | 推荐规则 | 示例 |
|:---|:---|:---|
| **测试类** | 以 `Test` 结尾 | `UserActionTest` |
| **测试方法** | 以 `test` 开头，或使用 PHPUnit Attributes | `testHandleWithInvalidData` |
| **辅助方法** | 避免以 `test` 开头 | `buildFakeUser` |

## 3. 覆盖率与测试发现

### 3.1 发现顺序
1. 读取 `phpunit.xml`，提取 testsuite、directory、file、source、coverage 信息
2. 扫描 `{scan_root}/*/tests/**/*Test.php`
3. 对比“配置声明的测试范围”与“仓库实际存在的测试资产”

### 3.2 覆盖率判断原则
- 若存在 coverage 配置或报告，优先使用现有证据
- 若无法获得真实覆盖率，只做粗略估算，并明确说明估算依据
- 覆盖率目标来自 `references/scoring.md`，不要在本规范中写死唯一阈值

## 4. 风险分层规则

### 4.1 High Risk
- 测试文件中存在 `dd()`, `dump()`, `var_dump()`
- 测试方法缺少断言或等效验证
- 存在被注释掉的测试入口（如 `// public function test...`）
- `phpunit.xml` 与真实测试资产严重脱节，导致核心测试无法被发现

### 4.2 Medium Risk
- 测试继承基类不统一或上下文准备不清晰
- coverage `source` 范围明显过窄
- 目录结构偏移较大，增加维护成本

### 4.3 Low Risk
- AAA 结构不够明显
- 未使用 Faker 或测试数据复用方式较弱
- 命名可读性一般，但不影响执行

## 5. 代码质量建议

### 5.1 AAA 模式
推荐测试方法保持 Arrange / Act / Assert 结构，但不要为了形式化而忽略真实可读性。

### 5.2 优先检查项
1. 断言是否存在
2. 是否有调试残留
3. 测试入口是否会被发现
4. 是否继承合理的测试基类

## 6. AI 分析检查清单 (Checklist)

- [ ] **Config**: 根目录 `phpunit.xml` 是否存在，且 testsuite 定义基本合理？
- [ ] **Discovery**: `phpunit.xml` 中声明的测试范围与 `{scan_root}/*/tests` 的真实文件分布是否一致？
- [ ] **Naming**: 测试文件名是否以 `Test.php` 结尾？
- [ ] **Inheritance**: 测试类是否继承项目测试基类或等效基类？
- [ ] **Assertions**: 测试方法中是否包含 `$this->assert*` 或等效验证？
- [ ] **Debug**: 是否存在 `dd()`、`dump()`、注释掉的测试代码？
