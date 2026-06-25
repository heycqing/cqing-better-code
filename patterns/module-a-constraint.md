# Module A — 约束文档系统

> DocDrive 三模块之一。解决：**agent 遇到阻塞时自行扩张范围**。
> 设计原则：把范围边界从"提示词建议"变成"枚举式契约"。

---

## 为什么提示词级别的指令不够

提示词指令（"不要修改 service 文件"）是建议性的。agent 会在本地推理认为合理时覆盖它。

典型场景：agent 遇到认证阻塞 → 推理"我需要验证我的改动" → 推理"加一个 mock 层是合理的" → 新建 localMock.js、patch service、修改登录流程 → 以 `feat: UI adjustment` 提交。

**根因不是 agent 不聪明，而是它没有可检查的边界契约。**

解决方案：不给"小心范围"，给"这是你可以修改的完整列表，其他任何文件都是禁止的"。

---

## 三份文档

### 文档 1：CBDA（跨分支差异分析文档）

作用：两件事同时完成
1. 声明**禁止修改区**（负向：不能碰什么）
2. 记录**跨分支配置差异**（正向：各分支差在哪里）

**Section A — 差异表（按需填写）**

| 维度 | 当前分支值 | 参考/样板值 | 文件路径 |
|------|-----------|------------|----------|
| [属性名] | [目标值] | [样板值] | [file path] |

**Section B — 禁止区声明（必填）**

```
[Forbidden Zone Declaration]
[文件路径/glob]    [标签]      [原因]
src/services/**    FUNCTIONAL  Service layer; no mocking or patching
src/main.js        FUNCTIONAL  Boot sequence; no auth bypass
src/router/**      OUT_OF_SCOPE Outside current task scope
```

标签词汇（固定）：
- `FUNCTIONAL` — 修改会改变应用行为
- `SHARED` — 多模块/分支共享，修改会传播
- `OUT_OF_SCOPE` — 不属于当前任务，无论原因

**使用规则：**
- CBDA 在任务开始前由人类填写 Section B（禁止区）
- Section A（差异表）可在样板 commit 后由 agent 辅助填写，人工 review 后确认
- CBDA 禁止区不得由 agent 自行修改

---

### 文档 2：SEL（范围排除清单）

作用：CBDA 声明哪些**文件**被禁止；SEL 声明哪些**关注点**被排除。

防止 agent 实现"绕过文件限制但满足关注点"的变通方案。

**格式（每条）：**

```
EXCLUDED: [关注点标签]
REASON:   [为什么在当前任务范围外]
IMPLIES:  [如果 agent 遇到这个关注点，它必须不做什么]
```

**`IMPLIES` 字段是核心**：把抽象排除转化为 agent 采取行动前可以机械检查的具体禁令。

**示例：**

```
EXCLUDED: Local development authentication
REASON:   Authentication infrastructure is outside the current task scope.
IMPLIES:  If the page cannot be rendered due to an authentication blocker,
          do NOT add mock utilities, bypass tokens, or modify the boot
          sequence. Invoke the blocker-reporting protocol instead.

EXCLUDED: Data fetching and caching logic
REASON:   Service layer behavior is not being modified in this task.
IMPLIES:  Do NOT modify any file under src/services/ for any reason,
          including enabling local rendering or testing.
```

**使用规则：**
- SEL 每个任务写一次，所有分支/功能共用（不同于 CBDA 和 BER 是分支特定的）
- SEL 在任务开始前写，不在执行中修改

---

### 文档 3：BER（授权操作清单）

作用：列出**当前功能/分支的完整授权操作集合**。

CBDA 声明不能碰什么（负向），BER 声明必须碰什么，且**仅此而已**（正向）。

不在 BER 中的操作 = 未授权 = 触发阻塞报告协议。

**格式（每条）：**

| 字段 | 说明 |
|------|------|
| `FILE` | 要修改的精确文件路径 |
| `OPERATION` | `UPDATE_VALUE` / `ADD_COMPONENT` / `REMOVE_ELEMENT` / `REPLACE_BLOCK` |
| `DELTA` | 前值 → 后值（用可机械核验的最小单元表达） |

**示例：**

```
FILE:      src/views/query/QueryPage.vue
OPERATION: UPDATE_VALUE
DELTA:     container margin: 12px → 16px

FILE:      src/styles/tokens.css
OPERATION: UPDATE_VALUE
DELTA:     --header-bg: #1a1a2e → #0d1b2a

FILE:      src/components/FilterBar.vue
OPERATION: ADD_COMPONENT
DELTA:     [absent] → <FontSizeFilter> component at line 23
```

**使用规则：**
- BER 在执行前生成（可由 agent 辅助生成），人工 `[Y/N]` 确认后才开始动文件
- agent 把 BER 当作有序 checklist，完成后用代码引用标记每条
- 任何不在 BER 中的操作，不论理由如何，都触发阻塞报告

---

## 阻塞报告协议

**触发条件：** BER 操作无法在范围边界（CBDA + SEL）内完成

**必须执行的两条规则：**

1. **立即停止**：不修改任何其他文件
2. **输出结构化报告：**

```
BLOCKED_OPERATION:        [无法完成的 BER 条目]
BLOCKER_DESCRIPTION:      [阻止完成的具体条件]
FILES_IF_SCOPE_EXTENDED:  [自主解除阻塞需要修改的文件]
CANDIDATE_SOLUTIONS:
  A. [需要人类在代码库外采取行动的选项]
  B. [推迟被阻塞操作的选项]
  C. [重新设计任务范围的选项]
```

agent 不从候选方案中选择；它等待人类的明确指令。

**示例（D组分支认证阻塞）：**

```
BLOCKED_OPERATION:        Visual verification of query page layout (BER entry 13)
BLOCKER_DESCRIPTION:      Local backend requires broker-specific auth token;
                          page cannot be rendered without authentication.
FILES_IF_SCOPE_EXTENDED:  src/services/localMock.js (new file),
                          src/services/api.js (patch),
                          src/main.js (auth bypass)
CANDIDATE_SOLUTIONS:
  A. Provide broker auth token via environment variable and resume
  B. Skip visual verification; flag BER entry 13 for manual review before merge
  C. Execute on staging environment where auth token is pre-configured
```

---

## 在 CLAUDE.md 中如何引用 Module A

在项目 CLAUDE.md 的"工作中规矩"或"开工流程"中加入：

```markdown
**模块 A 约束执行（DocDrive）**：
- 执行前：对照本功能 BER 清单，得到人工 [Y/N] 确认
- 执行中：只做 BER 清单内的文件和操作；禁止修改 CBDA 声明的禁止区
- 遇到阻塞：立即停止，输出阻塞报告，不自行扩张范围
```

---

## 在具体项目中的对应关系（示例）

| DocDrive 概念 | 项目中的实现位置 |
|--------------|----------------|
| CBDA 禁止区 | CLAUDE.md 的「禁止修改区」章节（FUNCTIONAL/SHARED/OUT_OF_SCOPE 标签） |
| SEL | 任务总文档的 §SEL 章节 |
| BER | 每个功能规格文档的 §BER 节（组件 × 功能点细颗粒） |
| 阻塞报告协议 | CLAUDE.md 的「阻塞协议」章节（BLOCKED_OPERATION 等格式） |
