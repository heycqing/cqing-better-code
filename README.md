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
│  Harness Engineering ← DocDrive                     │
│  5子系统框架          ↑                              │
│                    实践化                            │
│                 三模块工作流                         │
└─────────────────────────────────────────────────────┘
                         ↓ 机制化
┌─────────────────────────────────────────────────────┐
│              可复用模式层（patterns/）                │
│                                                     │
│  Module A          Module B          Module C       │
│  约束文档系统      样板优先批量迁移   视觉验证清单   │
│  (CBDA/SEL/BER/    (样板→确认清单→   (VVC预提交门控) │
│   阻塞协议)         目标执行)                        │
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

## 目录结构与文件说明

```
cqing-better-code/
│
├── README.md                    本文件 — 全局导航与使用手册
│
├── foundations/                 理论基础层
│   ├── harness-engineering.md   Harness Engineering 5子系统理论摘要
│   └── docdrive.md              DocDrive 方法论定义、来源、三模块、扩展历史
│
├── patterns/                    可复用机制层（DocDrive 三模块的操作手册）
│   ├── module-a-constraint.md   Module A：CBDA + SEL + BER + 阻塞协议
│   ├── module-b-batch-migration.md  Module B：样板优先批量执行协议
│   └── module-c-visual-verify.md   Module C：VVC 结构化视觉验证清单
│
├── governance/                  规则治理层
│   └── rule-governance.md       P0/P1/P2 规则分层框架（通用版）
│
├── workflow.md                  AI 协作分工 + 会话启动/收尾协议 + 变更原子性
│
├── improvements/                改进记录（每次跨项目方法论升级在此存档）
│   └── doc-sync-v1.md           改进 #1：文档代码同步原子性（2026-06-25）
│
├── templates/                   项目启动模板
│   ├── CLAUDE-template.md       新项目 CLAUDE.md 模板（含 P0/P1/P2 结构）
│   └── backend-api-template.md  接口契约登记文件模板
│
└── ref/                         外部参考材料副本（独立可读，不依赖本机路径）
    ├── harness-engineering-core.md   Harness Engineering 课程核心内容提炼
    └── docdrive-origin-paper.md      DocDrive 原始论文关键章节（已脱敏）
```

---

## 各文件功能详解

### `foundations/harness-engineering.md`
- **是什么：** 本库方法论的理论根基——理解 AI 工程 Harness 的核心框架
- **读它获得：** 5个子系统的概念、有/无 Harness 的对比、与 DocDrive 的关系
- **读完后去：** `foundations/docdrive.md`

### `foundations/docdrive.md`
- **是什么：** DocDrive 方法论的正式定义文件
- **读它获得：** DocDrive 是什么、来自哪里（某真实生产项目）、三模块定义、后续项目对它的扩展
- **读完后去：** `patterns/` 下对应模块

### `patterns/module-a-constraint.md`
- **是什么：** DocDrive Module A 的完整操作手册
- **解决的问题：** agent 遇到阻塞时自行扩张范围
- **包含：** CBDA 格式 + SEL 格式 + BER 格式 + 阻塞报告协议 + CLAUDE.md 引用方式
- **何时使用：** 任何"需要限制 agent 修改范围"的任务

### `patterns/module-b-batch-migration.md`
- **是什么：** DocDrive Module B 的完整操作手册
- **解决的问题：** 多目标迁移时的遗漏与冗余
- **包含：** 两阶段协议（样板 → 目标）+ 预执行确认清单格式 + 应用范围
- **何时使用：** 多分支/多页面/多租户的同构改造任务

### `patterns/module-c-visual-verify.md`
- **是什么：** DocDrive Module C 的完整操作手册
- **解决的问题：** 视觉正确性未被验证（结构正确 ≠ 视觉正确）
- **包含：** VVC 格式 + 核心约束（无证据不判 PASS）+ 非 UI 场景应用
- **何时使用：** 任何需要验证"实现与规格一致"的任务（不限于 UI）

### `governance/rule-governance.md`
- **是什么：** P0/P1/P2 规则分层框架（通用版，不含项目特定内容）
- **读它获得：** 如何对规则分层、每层的违反后果、规则生命周期管理
- **何时使用：** 新项目设计 CLAUDE.md 规则结构时

### `workflow.md`
- **是什么：** AI 协作分工模型 + 标准会话协议
- **包含：** 三角色分工（Claude/Codex/用户）+ 会话启动/收尾 checklist + 变更原子性协议
- **何时使用：** 每次项目启动时对照；每次会话收尾时执行 checklist

### `improvements/doc-sync-v1.md`
- **是什么：** 第一个跨项目改进记录（文档代码同步原子性）
- **读它获得：** 根因分析、解决方案、落地情况、适用范围
- **作为范例：** 后续改进记录应遵循同样格式（问题 + 根因 + 解决方案 + 落地情况）

### `templates/CLAUDE-template.md`
- **是什么：** 新项目 CLAUDE.md 的起始模板
- **如何使用：** 复制到项目根目录 → 填写 `[占位项]` → 按需增减 P2 规则

### `templates/backend-api-template.md`
- **是什么：** 接口契约登记文件（backend-api.md）的模板
- **如何使用：** 复制到项目 docs/ 目录 → 每接入一个真实接口在此登记

### `ref/harness-engineering-core.md`
- **是什么：** Harness Engineering 课程核心内容的提炼副本
- **为何在 ref/：** 原始材料在本机路径，GitHub 无法访问；此文件是可独立阅读的副本
- **读它获得：** 5子系统图、会话生命周期、12 讲速查表

### `ref/docdrive-origin-paper.md`
- **是什么：** DocDrive 原始论文的关键章节提炼（已脱敏，来自某真实 B 端生产项目）
- **为何在 ref/：** 原始论文在非公开代码库，此文件是可独立阅读的副本
- **读它获得：** DocDrive 完整定义、失败案例、三模块正式规格、关键格式

---

## 如何使用这个库

### 场景 1：启动新项目

```
1. 复制 templates/CLAUDE-template.md 到新项目根目录
2. 阅读 foundations/docdrive.md，判断需要哪些模块
3. 按需从 patterns/ 选取对应模块操作手册
4. 在项目 CLAUDE.md 的规则部分，引用 governance/rule-governance.md 的 P0/P1/P2 结构
5. 复制 templates/backend-api-template.md 到项目 docs/ 目录
```

### 场景 2：有多分支/多目标同构改造任务

```
1. 读 patterns/module-a-constraint.md，为任务制定 CBDA + SEL + BER
2. 读 patterns/module-b-batch-migration.md，执行两阶段样板优先协议
3. 完成每个目标后，用 Module C VVC 做提交前验证
```

### 场景 3：遇到 agent 越界/半成品/文档不同步等问题

```
1. 看 governance/rule-governance.md 判断属于哪层（P0/P1/P2）
2. 看 improvements/ 是否已有对应改进记录
3. 如果是新问题，在 improvements/ 下新建 xxx-v1.md 记录改进
4. 同时更新 rule-governance.md 和 workflow.md
5. 更新本 README 的版本日志
```

### 场景 4：在 CLAUDE.md 中引用这个库

在项目 CLAUDE.md 中写：

```markdown
## 方法论参考
见 `cqing-better-code/`（GitHub 地址待定）
- 规则分层：governance/rule-governance.md
- 工作流协议：workflow.md
- DocDrive 三模块：patterns/module-a/b/c-*.md
```

不要把本库内容复制进项目——引用可以随库更新；复制会产生版本漂移。

---

## 版本日志

| 版本 | 日期 | 内容 |
|------|------|------|
| v1 | 2026-06-25 | 初始提炼（dev-playbook）：P0/P1/P2 规则框架、AI 协作工作流、文档同步改进 |
| v2 | 2026-06-25 | 重组为三层架构：foundations（Harness Engineering + DocDrive 定义）+ patterns（DocDrive 三模块操作手册）+ ref（外部材料副本）；重命名为 cqing-better-code |
