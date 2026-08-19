# `/devops-workflow fix` — 轻量 Bug 修复

> 执行 `/devops-workflow fix` 时读本文件。不走 5 阶段状态机，单次闭环完成。

## 用法

```
/devops-workflow fix {bug描述}
```

## 流程

```
① 确认问题 → ② 定位根因 → ③ 编码修复 → ④ CR 扫描 →[★CR 裁决]→ ⑤ 改写 → 完成
                  ↓
           涉及设计变更？→ 升级到 /devops-workflow rework
```

### ① 确认问题

1. 与用户确认 bug 所属**业务域**和**影响范围**（哪些模块/文件）
2. 如用户提供了复现步骤或错误日志，记录备用

### ② 定位根因

读取影响范围内的代码，定位 bug 根因。向用户输出：
- **根因摘要**：一句话说明问题出在哪
- **修复方案**：计划改哪些文件、怎么改
- **影响判定**：是否涉及设计层变更（接口签名/数据模型/跨模块契约）

**★ 升级判定（强制）**：如果根因涉及以下任一，**必须停止 fix 流程**，提示用户升级：
- 需要修改对外接口签名或数据模型
- 影响跨模块/跨服务契约
- 需要修改多个不相关模块的逻辑

```
⛔ 此 bug 涉及设计层变更，fix 流程无法覆盖。
建议执行 /devops-workflow rework 处理，根因分析如下可直接复用：
- 根因：{根因摘要}
- 影响范围：{受影响模块}
```

用户确认修复方案后，进入编码。

### ③ 编码修复

```
executor "
修复 bug：{根因摘要}。
修复方案：{方案描述}。
工作范围：{影响范围目录}。
完成后执行模块级单测 + 按项目 .claude/rules/ 中定义的语法/编译检查通过。

遵守项目 CLAUDE.md 与 .claude/rules/ 的架构与编码规范。
Bash 命令必须可静态分析：禁止 for/while/if/case/here-doc/嵌套 $()。"
```

executor 返回后，主 Agent 向用户输出改动摘要和测试结果。

### ④ CR 扫描

```
code-reviewer "
审查 bug 修复在当前分支上的实际代码变更：
1. 执行 git diff <默认分支>...HEAD -- {影响范围目录} 获取变更内容
2. 基于实际变更审查：修复是否完整、是否引入新问题、是否涉及设计层变更
3. 把问题写入对话（无持久化产出物），**严格按照下方格式**输出：

   ## 问题清单

   ### [1] 严重度: MAJOR — {问题标题}

   - 文件: `{实际文件路径}:{行号}`
   - 问题: {具体描述}
   - 建议: {修复建议}
   - 裁决: PENDING

审查必须基于实际代码变更，不做脱离代码的通用检查。
Bash 命令必须可静态分析：禁止 for/while/if/case/here-doc/嵌套 $()。"
```

code-reviewer 返回后，主 Agent 读取结果并向用户展示：

- **零问题**：输出「CR 通过，bug 修复完成」，流程结束
- **有问题**：逐条展示，等待用户裁决（ACCEPTED/REJECTED/MODIFIED），然后 `/devops-workflow approve`

**★ CR 发现设计层问题时**：同样提示升级到 `/devops-workflow rework`，不在 fix 流程内处理。

### ⑤ 改写（仅有 ACCEPTED/MODIFIED 项时）

`/devops-workflow approve` 确认后：

```
executor "
按审查裁决修复：{裁决摘要}。
工作范围：{影响范围目录}。
完成后执行模块级单测 + 语法/编译检查通过。

遵守项目 CLAUDE.md 与 .claude/rules/ 的架构与编码规范。
Bash 命令必须可静态分析：禁止 for/while/if/case/here-doc/嵌套 $()。"
```

改写完成，确认测试通过，流程结束。

## 与需求流程的区别

| | 需求流程（start） | fix 流程 |
|---|---|---|
| 产出物 | progress.md + design-consensus + dev-tasks + done/review 报告 | 无持久化产出物 |
| 状态机 | 5 阶段，多人工门 | 单次闭环，仅 CR 一道门 |
| 设计 | 多模块分析 + design-consensus | 无，直接定位根因 |
| 适用场景 | 新功能、大重构 | 明确的 bug，改动范围可控 |
| 升级通道 | — | 发现设计缺陷 → `/devops-workflow rework` |

## 约束

- **编码与审查分两轮**：executor 和 code-reviewer 不共享上下文
- **CR 问题不经人工确认不改写**：同需求流程
- **所有 agent prompt 必须含 Bash 静态分析约束**
