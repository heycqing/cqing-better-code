# Harness Engineering — 理论基础

> 本文件是 cqing-better-code 方法论的理论根基。
> 详细材料见 `../ref/harness-engineering-core.md`（原始课程内容提炼副本）。

---

## 是什么

Harness Engineering 是围绕 AI 编程 agent 构建一套完整工作环境，使其产生可靠结果的工程学科。

**不是关于写更好的提示词。是关于设计 agent 运行其中的系统。**

核心洞察：
- 强大的模型 ≠ 可靠的执行
- 改变 Harness（不改变模型）可以产生质变，不只是边际提升
- 模型决定写什么代码；Harness 管控何时、何地以及如何写

---

## 五个子系统（一句话版）

| 子系统 | 一句话 |
|--------|--------|
| **指令** | 告诉 agent 做什么、按什么顺序、遇到问题找哪里 |
| **状态** | 把进度持久化到磁盘，使下一次会话从上次结束处继续 |
| **验证** | 只有通过的测试/build 才算数；agent 不能仅凭"感觉正确"宣布完成 |
| **范围** | 一次只做一个功能，不越界，不同时维护三个半成品 |
| **会话生命周期** | 开始时初始化环境，结束时清理并留下干净重启路径 |

完整图示和对比见 `../ref/harness-engineering-core.md`。

---

## 与 DocDrive 的关系

DocDrive（见 `docdrive.md`）是 Harness Engineering 的一个具体实践：

- Harness Engineering 是**理论框架**（5个子系统，来自 Anthropic/OpenAI）
- DocDrive 是**实践实现**（3个模块，针对"人类主调度 + AI 执行"场景下的三个特定短板）

DocDrive 在 Harness Engineering 的基础上，增加了：
1. 精确的**禁止修改区**声明机制（Module A）
2. **结构化阻塞报告**协议（Module A）
3. **批量迁移**的样板优先协议（Module B）
4. **视觉验证**的预提交门控机制（Module C）

---

## 理解顺序建议

1. 先读本文件（5 分钟，建立概念框架）
2. 再读 `docdrive.md`（了解 DocDrive 三模块如何实现这个框架）
3. 再读 `../patterns/`（理解每个模块的操作细节）
4. 需要原始材料时查 `../ref/harness-engineering-core.md`
