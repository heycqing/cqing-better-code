# 改进 #1：文档代码同步原子性

> 状态：已落地（2026-06-25）
> 来源项目：某 B 端权限管理项目（已脱敏）
> 影响规则层级：P1
> 已同步至：`rule-governance.md`、`workflow.md`、来源项目 `CLAUDE.md`、memory

---

## 问题描述

### 现象

某项目运行期间，以下接口变更只更新了代码，`backend-api.md` 未同步：

| 变更 | 代码 | 文档 |
|------|------|------|
| `queryUserList` 入参从 `name/nickname/email` 改为 `keyword` | ✅ 已改 | ❌ 未更新 |
| 新增 `query-operation-logs-type` 接口 | ✅ 已调用 | ❌ 未登记 |
| 接入 `dept-user-list`（org.ts） | ✅ 已实现 | ❌ 未登记 |

### 根因

**文档更新被隐性地当作"可以之后再补"的次要工作。**

整个执行期间，从来没有一个时机让"文档没更新"感觉不完整——代码跑了，build 通过了，会话结束了，一切正常。没有任何东西报错或阻塞。

结果：`backend-api.md` 在第一次接口变更发生时就开始漂移，逐渐从"单一真相来源"退化成"初始快照"。

---

## 解决方案

### 核心重定义

把"更新文档"从一个独立步骤变成**变更定义的一部分**：

```
旧定义：一次变更 = 改代码  （文档是附带的，可以之后补）
新定义：一次变更 = 改代码 + 改文档  （两者缺一，变更未完成）
```

类比 git commit：没有 commit message 就没有 commit，这不是规则，是定义。

### 文档先于代码

为了让"不更新文档"感觉不完整而非只是"违规"，改变操作顺序：

```
旧顺序：改代码 → （可能）改文档
新顺序：改文档 → 改代码
```

当文档是前置条件时，文档自然不会被跳过。

### 三种变更来源的具体操作

| 来源 | 旧流程 | 新流程 |
|------|--------|--------|
| 后端通知接口变化 | 直接改 service | 先改 backend-api.md → 再改 service |
| 会话中决定换方案 | 直接改代码 | 先改规格文档 → 再改代码 |
| 自己写代码接入新接口 | 直接写 createApi 调用 | 先在 backend-api.md 登记 → 再写调用 |

### 会话收尾机械核验

在会话结束的 checklist 中加入机械核验步骤（不依赖记忆，不依赖判断）：

```bash
# 找出项目内所有接口调用的 URL（示例，按实际项目路径调整）
grep -r "url:" src/ --include="*.ts" -h | grep -oP "'[^']*'"

# 逐一在 backend-api.md 核对
# 找不到 = 文档未同步 = 补文档后才能关会话
```

---

## 落地情况

### 已更新的文件

| 文件 | 更新内容 |
|------|----------|
| 来源项目 `CLAUDE.md` §2 | 新增「变更原子性」P1 规则 |
| 来源项目 `CLAUDE.md` §4 | 新增会话收尾 grep 核验步骤 |
| `cqing-better-code/governance/rule-governance.md` | P1 层新增「文档代码同步」规则 |
| `cqing-better-code/workflow.md` | §四 变更原子性协议 + §八 标准 spec 文件图谱 |
| memory `feedback-rule-governance.md` | 新增最容易忽视的 P1 规则说明 |

### 适用范围

任何使用以下模式的项目均适用：
- 有独立的后端 API 需要前端接入
- 有 spec 文件作为接口契约
- 使用 AI 辅助编码（AI 不知道"口头确认"了什么）

---

## 未来可能的延伸

1. **自动化核验**：在 pre-commit hook 中加入 grep 核验，让违反成为 git 级别的阻断
2. **接口登记模板**：在 IDE snippet 中预置 createApi 调用的模板，强制带上 backend-api.md 的引用
3. **PR checklist**：在 PR 描述模板中加入"backend-api.md 已同步"的 checkbox

---

## 命名规范

本改进命名为 `doc-sync`，后续迭代使用 `doc-sync-v2`、`doc-sync-v3`。
跨项目改进统一放在 `cqing-better-code/improvements/` 下。
