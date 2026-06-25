# cqing-better-code

> 个人 AI 辅助编程方法论库。
> 从多个真实项目中提炼的规则、流程、模式与模板，供所有后续项目引用。

---

## 这个库是什么

**cqing-better-code** 是一个跨项目的方法论沉淀库，目标是：

- 让每个新项目不从零开始定义 AI 协作规则
- 让过去的经验教训（包括失败案例）可以被未来的项目继承
- 让 AI agent 在读到这个库的引用后，能以更高的可靠性执行工程任务

---

## 三层架构

```
┌─────────────────────────────────────────────────────┐
│              理论基础层（foundations/）               │
│                                                     │
│  Harness Engineering ──实践化──→ DocDrive            │
│  5子系统框架                      三模块工作流        │
└─────────────────────────────────────────────────────┘
                         ↓ 机制化
┌─────────────────────────────────────────────────────┐
│              可复用模式层（patterns/）                │
│                                                     │
│  Module A          Module B          Module C       │
│  约束文档系统      样板优先批量迁移   视觉验证清单   │
│  CBDA/SEL/BER      样板→确认清单→    VVC预提交门控   │
│  阻塞协议          目标执行                          │
└─────────────────────────────────────────────────────┘
                         ↓ 治理化
┌─────────────────────────────────────────────────────┐
│            治理与流程层（governance/ + 根文件）       │
│                                                     │
│  rule-governance.md    workflow.md                  │
│  P0/P1/P2分层          会话协议 + 变更原子性         │
└─────────────────────────────────────────────────────┘
```

---

## 目录结构

```
cqing-better-code/
│
├── README.md                        本文件 — 全局导航与使用手册
│
├── foundations/                     理论基础层
│   ├── harness-engineering.md       Harness Engineering 5子系统理论摘要
│   └── docdrive.md                  DocDrive 方法论定义、来源、三模块、扩展历史
│
├── patterns/                        可复用机制层（DocDrive 三模块操作手册）
│   ├── module-a-constraint.md       Module A：CBDA + SEL + BER + 阻塞协议
│   ├── module-b-batch-migration.md  Module B：样板优先批量执行协议
│   └── module-c-visual-verify.md    Module C：VVC 结构化视觉验证清单
│
├── governance/                      规则治理层
│   └── rule-governance.md           P0/P1/P2 规则分层框架（通用版）
│
├── workflow.md                      AI 协作分工 + 会话启动/收尾协议 + 变更原子性
│
├── improvements/                    改进记录（每次方法论升级在此存档）
│   ├── doc-sync-v1.md               改进 #1：文档代码同步原子性
│   ├── review-gate-enforcement-v1.md 改进 #2：人工审核门执行纪律
│   ├── mock-file-discipline-v1.md   改进 #3：Mock 文件四件套纪律
│   └── feature-status-definition-v1.md 改进 #4：功能状态定义与进度文件即时更新
│   └── doc-sync-v1.md               改进 #1：文档代码同步原子性（2026-06-25）
│
├── templates/                       项目启动模板
│   ├── CLAUDE-template.md           新项目 CLAUDE.md 模板（含 P0/P1/P2 结构）
│   └── backend-api-template.md      接口契约登记文件模板
│
└── ref/                             外部参考材料副本（独立可读，不依赖外部路径）
    ├── harness-engineering-core.md  Harness Engineering 课程核心内容提炼
    └── docdrive-origin-paper.md     DocDrive 原始论文关键章节（已脱敏）
```

---

## 各文件功能详解

### `foundations/harness-engineering.md`
- **是什么：** 本库方法论的理论根基——理解 AI 工程 Harness 的核心框架
- **读它获得：** 5个子系统的概念、有/无 Harness 的对比、与 DocDrive 的关系
- **读完后去：** `foundations/docdrive.md`

### `foundations/docdrive.md`
- **是什么：** DocDrive 方法论的正式定义文件
- **读它获得：** DocDrive 是什么、来自哪里、三模块定义、后续项目对它的扩展
- **读完后去：** `patterns/` 下对应模块

### `patterns/module-a-constraint.md`
- **解决的问题：** agent 遇到阻塞时自行扩张范围
- **包含：** CBDA / SEL / BER 三份文档格式 + 阻塞报告协议 + CLAUDE.md 引用方式
- **何时使用：** 任何需要限制 agent 修改范围的任务

### `patterns/module-b-batch-migration.md`
- **解决的问题：** 多目标迁移时的遗漏与冗余
- **包含：** 两阶段协议（样板 → 目标）+ 预执行确认清单格式
- **何时使用：** 多分支 / 多页面 / 多租户的同构改造任务

### `patterns/module-c-visual-verify.md`
- **解决的问题：** 结构正确 ≠ 视觉正确，验收缺口被跳过
- **包含：** VVC 格式（file:line + 实测值 + PASS/FAIL）+ 无证据不判 PASS 约束
- **何时使用：** 任何需要验证"实现与规格一致"的任务，不限于 UI

### `governance/rule-governance.md`
- **是什么：** P0/P1/P2 规则分层框架，通用版，不含项目特定内容
- **何时使用：** 新项目设计 CLAUDE.md 规则结构时作为参照

### `workflow.md`
- **包含：** 三角色分工（Claude / Codex / 用户）+ 会话启动 / 收尾 checklist + 变更原子性协议
- **何时使用：** 项目启动时对照；会话收尾时执行 checklist

### `improvements/`
每个文件格式：问题描述（现象 + 根因）→ 解决方案 → 落地情况。第一个文件（`doc-sync-v1.md`）是范例。

| 文件 | 解决的问题 |
|------|-----------|
| `doc-sync-v1.md` | API 文档与代码不同步；文档更新被当作可以后补的次要工作 |
| `review-gate-enforcement-v1.md` | 审核门定义了但从未执行；声明不等于执行 |
| `mock-file-discipline-v1.md` | Mock 数据内联进共用 service；假接口无契约描述、无系统清理 |
| `feature-status-definition-v1.md` | 功能状态机缺少中间态；进度文件更新滞后导致失真 |

### `templates/CLAUDE-template.md`
- **如何使用：** 复制到项目根目录 → 填写 `[占位项]` → 按需增减 P2 规则

### `templates/backend-api-template.md`
- **如何使用：** 复制到项目 docs/ 目录 → 每接入一个真实接口在此登记

### `ref/harness-engineering-core.md`
- **读它获得：** 5子系统图、会话生命周期、12 讲核心观点速查
- **为何在 ref/：** 原始课程在外部，此文件是可独立阅读的提炼副本

### `ref/docdrive-origin-paper.md`
- **读它获得：** DocDrive 完整定义、两个基准失败案例、三模块正式规格与关键格式
- **为何在 ref/：** 原始论文在非公开代码库，此文件是已脱敏的提炼副本

---

## 怎么用（自己）

**启动新项目，做三件事：**

```
1. 复制 templates/CLAUDE-template.md 到新项目根目录，填写 [占位项]
2. 在项目 CLAUDE.md 顶部写一行引用：
   > 方法论参考：cqing-better-code/（见 governance/ + workflow.md）
3. 复制 templates/backend-api-template.md 到项目 docs/ 目录
```

**项目运行中，按需查阅：**

| 遇到的情况 | 去哪里查 |
|-----------|---------|
| 需要约束 agent 的改动范围 | `patterns/module-a-constraint.md` |
| 多目标批量改造 | `patterns/module-b-batch-migration.md` |
| 提交前验证功能与规格是否一致 | `patterns/module-c-visual-verify.md` |
| 规则冲突，不知道哪条优先 | `governance/rule-governance.md` |
| 会话收尾不知道做什么 | `workflow.md` §七收尾 checklist |

**不要把库内容复制进项目**——引用可以随库更新；复制会产生版本漂移。

---

## 怎么分享给他人

**给团队（共享同一套约束）：**

把库推到团队可访问的 GitHub 仓库，在新项目 CLAUDE.md 里统一写：

```markdown
> 方法论参考：https://github.com/[账号]/cqing-better-code
```

这样团队所有项目引用同一个来源，库更新后所有项目下一次会话自动生效。

**给个人（让对方建自己的版本）：**

让对方 fork 这个库。fork 之后：
- `foundations/` 和 `patterns/` 保持不变（理论和三模块结构通用）
- `governance/rule-governance.md` 的 P2 层按对方项目偏好修改
- `improvements/` 由对方自己的项目教训填充

两者共享的是方法论结构，分叉的是项目偏好规则。**不要让对方直接用你的库**——你的 P2 规则和你的项目上下文绑定，对别人没有意义。

---

## 怎么持续优化方法论

**唯一可靠的方式：从实践提取，不从思考提取。**

每次项目结束后问一个问题：**"这次出了什么问题，或者发现了什么更好的做法？"**

- 一句话能说清楚 → 更新 `governance/rule-governance.md` 对应规则条目
- 需要根因 + 解决方案 → 新建 `improvements/xxx-v1.md`，按现有格式写，更新版本日志

**改进文件的格式（参照 `improvements/doc-sync-v1.md`）：**

```
问题描述（现象 + 根因）
解决方案（核心重定义 + 具体操作）
落地情况（已更新的文件清单 + 适用范围）
```

**警惕一个陷阱：** 不要在没有真实案例前预判需要什么规则然后加进去。空规则让库越来越难读、越来越难信任。**库的每一行都应该有对应的真实事件支撑。**

**维护节奏：**

```
项目开始 → 用库里的模板启动
项目中   → 遇到问题先查库，没有就自己解决
项目结束 → 把解决方案写进 improvements/，更新版本日志
```

---

## 版本日志

| 版本 | 日期 | 内容 |
|------|------|------|
| v1 | 2026-06-25 | 初始提炼（dev-playbook）：P0/P1/P2 规则框架、AI 协作工作流、文档同步改进 |
| v2 | 2026-06-25 | 重组为三层架构；新增 foundations（Harness Engineering + DocDrive）、patterns（DocDrive 三模块操作手册）、ref（外部材料副本）；重命名为 cqing-better-code；脱敏处理 |
| v3 | 2026-06-25 | 补入 4 条来自实际项目复盘的漏项：审核门执行纪律（#2）、Mock 文件四件套（#3）、功能状态定义 pending-backend（#4）；新增 §八验证环境降级协议 |
