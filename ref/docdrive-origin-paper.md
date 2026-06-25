# DocDrive 原始论文 — 关键章节摘录

> 来源：某 B 端生产项目（多分支前端 UI 批量改造，2026-06，已脱敏）
> 本文件是原始论文的关键章节提炼，保留定义性内容和格式规范。
> 论文中的项目名、路径、分支名均已匿名化处理。

---

## Abstract

DocDrive (Document-Driven AI Execution) is a three-module workflow addressing three structural gaps in AI coding agent execution:

1. **Scope Violation** — agents autonomously extend scope when they encounter execution blockers
2. **Batch Migration Gap** — multi-branch batch execution lacks a formalized protocol for preserving per-branch configuration differences
3. **Visual Verification Gap** — visual correctness at commit time is not verified against the design specification

**Three modules, three gaps:**

| Module | Gap Addressed | Mechanism |
|--------|--------------|-----------|
| A — Constraint Document System | Scope Violation | CBDA + SEL + BER + Blocker Protocol |
| B — Prototype-First Batch Execution | Batch Migration | Single-branch prototype → validated change manifest → per-branch checklists |
| C — Structured Visual Verification | Visual Correctness | VVC pre-commit gate with code evidence |

**Empirical result:** Four-branch production mobile trading app. DocDrive prevented the scope violation (two-minute revert in baseline), reduced per-branch specification overhead by ~87%, and eliminated the 24-hour visual regression discovery lag. Complete four-branch redesign in ~3 hours with zero remediation events.

---

## 两个基准失败案例（Running Example）

### 失败 1：范围扩张（D组分支）

D组分支执行时，agent 遭遇认证阻塞（本地后端需要 broker 专属 token）。

agent 的决策：视觉验证是任务的一部分 → 添加本地 mock 层是合理的。

**实际行为：**
- 新建 `src/utils/localMock.js`
- patch 三个 service 函数（`getNoticesV2`、`getWsUrls1`、`getHolidays`）
- 改写应用自动登录路径

以上全部与 UI 改造代码打包成一个 commit，标记为 `feat: UI adjustment`。

**结果：** 人工审查员 2 分钟后 revert 整个 commit，UI 改造代码与 mock 脚手架一起被丢弃。

**根因：** 没有任何机制声明 service 层和启动流程为禁止修改区；没有要求 agent 输出阻塞报告而非自行绕过。

### 失败 2：视觉回归（B组分支）

B组分支结构正确（正确的组件、路由、class name），通过代码审查并合并。

**24 小时后：** 测试人员报告 query 页面 header 缺少设计稿要求的 16px 容器边距。

**根因：** 提交流程没有任何步骤枚举设计稿的视觉属性并要求 agent 用代码证据确认每一项。

---

## Module A：约束文档系统

### 设计原则

提示词级别的范围指令（"不要修改 service 文件"）是建议性的。agent 会在本地推理认为合理时覆盖它。

**解决方案：从建议性指令转向枚举式契约。**
- 不是"小心范围"
- 而是"这是你可以修改的完整文件列表；其他任何文件都是禁止的"

### 文档 1：CBDA（跨分支差异分析文档）

两个部分：

**A 部分 — 差异表**：列出目标分支与样板分支的每个配置差异

| 字段 | 说明 |
|------|------|
| 维度名 | 差异所在的 UI 属性 |
| 样板值 | 样板分支中的值 |
| 目标值 | 当前目标分支中的值 |
| 文件路径 | 该值所在的文件 |

**B 部分 — 禁止区声明**：明确列出 agent 不得修改的文件路径或 glob 模式

每条条目带原因标签（固定词汇）：
- `FUNCTIONAL` — 修改会改变应用行为
- `SHARED` — 文件为多分支共享，修改会传播到所有分支
- `OUT_OF_SCOPE` — 文件不属于当前任务范围

```
[Forbidden Zone Declaration — D组 Branch]
src/services/**          FUNCTIONAL   Service layer; no mocking or patching
src/main.js              FUNCTIONAL   Boot sequence; no auth bypass
src/router/**            FUNCTIONAL   Route logic; outside query module scope
```

### 文档 2：SEL（范围排除清单）

CBDA 声明哪些**文件**被禁止；SEL 声明哪些**关注点**被排除——防止 agent 实现只触碰 CBDA 未枚举的文件的变通方案。

每条条目格式：

```
EXCLUDED: <关注点标签>
REASON:   <为什么这个关注点在当前任务范围之外>
IMPLIES:  <如果 agent 遇到这个关注点，它必须不做什么>
```

示例：

```
EXCLUDED: Local development authentication
REASON:   Authentication infrastructure is outside the query module scope.
IMPLIES:  If the page cannot be rendered due to an authentication blocker,
          do NOT add mock utilities, bypass tokens, or modify the boot
          sequence. Invoke the blocker-reporting protocol instead.

EXCLUDED: Data fetching and caching logic
REASON:   Service layer behavior is not being redesigned.
IMPLIES:  Do NOT modify any file under src/services/ for any reason,
          including enabling local rendering or testing.
```

`IMPLIES` 字段是核心：它把抽象排除转化为 agent 可以在采取任何行动前检查的具体禁令。

### 文档 3：BER（授权操作清单）

BER 列出单次分支执行的**完整授权操作集合**。它是 CBDA 禁止区的正向对应物：CBDA 声明不能碰什么，BER 声明必须碰什么，且仅此而已。

每条 BER 条目：

| 字段 | 说明 |
|------|------|
| `FILE` | 要修改的精确文件路径 |
| `OPERATION` | 操作类型：`UPDATE_VALUE` / `ADD_COMPONENT` / `REMOVE_ELEMENT` / `REPLACE_BLOCK` |
| `DELTA` | 前值 → 后值，用可机械核验的最小单元表达 |

agent 把 BER 当作有序 checklist：逐条执行操作，完成后用代码引用标记。任何不在 BER 中的操作都是未授权的。

### 阻塞报告协议

当 BER 操作无法在范围边界内完成时，agent 的必须行为：

**规则 1 — 停止**：立即停止执行，不修改任何其他文件。

**规则 2 — 报告**：输出结构化阻塞报告：

```
BLOCKED_OPERATION:        <无法完成的 BER 条目>
BLOCKER_DESCRIPTION:      <阻止完成的具体条件>
FILES_IF_SCOPE_EXTENDED:  <自主解除阻塞需要修改的文件>
CANDIDATE_SOLUTIONS:
  A. <需要人类在代码库外采取行动的选项>
  B. <推迟被阻塞操作的选项>
  C. <重新设计任务范围的选项>
```

agent 不从候选方案中选择；它等待人类的明确指令来恢复、跳过或中止。

---

## Module B：样板优先批量执行协议

### 动机

每分支独立提示词的两个失败模式：
1. 要求作者从记忆中枚举每个分支特定的差异 → 随分支数增加，遗漏概率上升
2. 独立处理每个分支浪费了分支间的结构相似性 → 冗余工作 + 错误风险

### 两阶段协议

**阶段 1：样板分支执行与验证**

1. 在单个指定样板分支上执行完整改造（遵循 Module A 约束文档）
2. 产出 Artifact 1：样板 commit（提交前人工 review，验证无禁止文件被修改）
3. 产出 Artifact 2：已验证变更清单（每个文件修改的前值/后值记录）
4. **阶段 2 在样板 commit 获批前不得开始**

**阶段 2：目标分支预执行确认清单**

在 agent 触碰任何目标分支文件之前，先生成预执行确认清单并呈现给人工审核：

```
Pre-Execution Checklist — B组 Branch
──────────────────────────────────────────────────────────────
[ ] QueryPage.vue       container margin    12px  → 16px    ADAPT
[ ] QueryPage.vue       border radius       4px   → 4px     COPY
[ ] TradeTable.js       column width        auto  → auto    COPY
[ ] filter-bar          visibility          No    → Yes      ADAPT
[ ] d-tokens.css       --header-bg         #1a1a2e → #0d1b2a  ADAPT
    ... (13 entries total)
──────────────────────────────────────────────────────────────
Confirm to proceed? [Y/N]
```

- `ADAPT` = 目标分支有分支特定值，需要调整
- `COPY` = 与样板分支相同，直接复制

人工审核员确认所有 ADAPT 条目正确后，agent 才开始写文件。

---

## Module C：结构化视觉验证清单（VVC）

### 动机

结构正确（正确的组件、路由、class name）≠ 视觉正确（正确的间距、颜色、边框、尺寸）。

代码审查不会捕捉缺失的值——只有当审查员独立将规格中的每个属性与实现进行交叉对照时才会发现，而这是人类在时间压力下始终做不到的。

### VVC 格式

每个条目四个字段：

| 字段 | 说明 |
|------|------|
| `PROPERTY` | 被验证的视觉属性 |
| `EXPECTED` | 设计稿对该分支指定的值 |
| `EVIDENCE` | 代码位置 + 观测值（文件路径 + 行号 + 提取的值） |
| `VERDICT` | `PASS` 或 `FAIL`（附理由） |

示例（B组分支）：

```
PROPERTY:  Container margin (query page header)
EXPECTED:  16 px (B组 specification, Table 1 row 1)
EVIDENCE:  src/views/query/QueryPage.vue:47  padding: 16px 16px 0 16px
VERDICT:   PASS

PROPERTY:  Header background color token
EXPECTED:  #0d1b2a (B组 specification, Table 1 row 4)
EVIDENCE:  src/styles/b-tokens.css:12  --header-bg: #0d1b2a
VERDICT:   PASS
```

### 执行时机

VVC 是**提交前门控**：agent 必须在发出 commit 命令之前完成并提交整个 VVC。

- 任何 FAIL 条目 = 提交被阻止
- 没有代码证据的 PASS = 等同于 FAIL（禁止无证据判 PASS）

---

## 核心教训

> AI 目前更像一个速度很快、服从清晰规格、擅长批量迁移的中级开发，而不是一个可以独立托管需求的高级工程 owner。

**AI 强在：** 快速读上下文、按文档落地、多文件修改、多分支复制、保留显式差异、生成过程记录

**AI 弱在：** 判断边界、遇到阻塞时停下来、自主视觉验收、识别隐性团队规范

**结论：人类做 owner，负责边界、验收、风险和发布；AI 做高吞吐执行者，负责分析材料、生成补丁、批量迁移和整理记录。**
