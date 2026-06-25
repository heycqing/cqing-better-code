# 改进 #4：功能状态定义与进度文件即时更新

> 状态：已落地（2026-06-25）
> 来源项目：某 B 端权限管理项目（已脱敏）
> 影响规则层级：P0（假 passing）+ P1（进度文件更新时机）
> 已同步至：`governance/rule-governance.md`、`workflow.md`

---

## 问题描述

### 现象 1：功能状态机缺少中间态

`feature_list.json` 只有三个状态：`not_started` / `in_progress` / `passing`。

当 UI 完成但后端依赖未就绪时，没有合适的状态可以用：
- 标 `passing`：不诚实，后端 TODO 还在
- 留 `in_progress`：不准确，前端工作已完成
- 结果：两种情况都出现过，feature_list 失去了作为"单一真相来源"的价值

### 现象 2：进度文件更新滞后

`claude-progress.md` 和 `feature_list.json` 被要求在"会话结束时"更新。但实际上：
- 很多会话在任务中途就结束了，来不及走完收尾 checklist
- 下一次会话开始时，进度文件和实际代码状态之间已经有偏差
- 复盘时发现：已完成的工作在进度文件里仍然显示为未完成

### 根因

**状态机设计不完整**：二元（做了/没做）无法描述"前端完成但外部依赖未就绪"这个真实存在的中间状态。

**更新时机设计有误**：绑定到"会话结束"而非"状态变化"。状态变化是随时发生的，会话结束是偶然的时机。

---

## 解决方案

### 新增状态：`pending-backend`

扩展 feature_list.json 的状态机，新增 `pending-backend` 状态：

| 状态 | 含义 |
|------|------|
| `not_started` | 未开始 |
| `in_progress` | 正在执行 |
| `pending-backend` | 前端/UI 工作已完成，等待后端依赖交付 |
| `passing` | 所有依赖就绪，完整验证通过，无残留 TODO |

**`pending-backend` 的使用条件：**
- UI/前端逻辑已实现并通过 build 验证
- 功能的 mock 接口已按四件套纪律建立
- 对应的后端 TODO 已登记在 `TODO-backend.md`

**`passing` 的严格定义：**
- 所有 mock 已替换为真实接口
- 所有 TODO 已清理（代码处 + TODO-backend.md）
- 完整验证通过（build + 人工审核）
- feature_list.json 中该功能无残留 TODO 注记

### 进度文件更新时机：状态变化时立即更新

**旧规则：** 会话结束时更新进度文件

**新规则：** 状态变化时立即更新，不等到会话结束

```
功能状态变化时 → 立即更新 feature_list.json
会话中有实质进展时 → 立即追加 claude-progress.md
会话结束时 → 核验两者是否准确，补漏，不是首次更新
```

类比：git commit 在"完成一个逻辑单元"时提交，不是在"今天下班"时提交。

---

## 落地情况

### 已更新的文件

| 文件 | 更新内容 |
|------|----------|
| `governance/rule-governance.md` P0 | 「不冒充真实」明确 `pending-backend` 不得标 passing |
| `workflow.md` §三 | 功能执行协议中：状态变化时立即更新，不等到会话结束 |
| `workflow.md` §七 | 收尾 checklist 改为"核验准确性"而非"首次更新" |

### 适用范围

任何有外部依赖（后端、第三方 API、设计稿待定）的功能开发。
即使是纯前端项目，`pending-backend` 的逻辑也可以改写为 `pending-design` / `pending-review` 等类似中间态。
