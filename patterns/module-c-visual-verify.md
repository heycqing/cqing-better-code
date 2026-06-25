# Module C — 结构化视觉验证清单（VVC）

> DocDrive 三模块之一。解决：**视觉正确性未被验证（结构正确 ≠ 视觉正确）**。
> 设计原则：把设计稿转化为 agent 可执行的代码证据清单，作为提交前强制门控。

---

## 结构正确 ≠ 视觉正确

**结构正确**（代码审查可以验证）：
- 正确的组件存在
- 路由正确解析
- class name 匹配规格
- 配置值语法正确

**视觉正确**（代码审查无法可靠验证）：
- 间距值与设计稿一致
- 颜色 token 匹配
- 边框半径正确
- 组件尺寸和对齐正确

**失败模式**：B组分支 container margin（16px）出现在设计稿中，但不在代码里。代码审查通过；24 小时后测试人员才发现。

根因：没有步骤枚举设计稿的视觉属性，并要求 agent 用代码证据逐一确认。

---

## VVC 格式

每个视觉属性一条记录，四个必填字段：

| 字段 | 说明 |
|------|------|
| `PROPERTY` | 被验证的视觉属性（来自设计稿） |
| `EXPECTED` | 设计稿对当前功能/分支指定的值 |
| `EVIDENCE` | **文件路径 + 行号 + 提取的实测值**（缺一不可） |
| `VERDICT` | `PASS` 或 `FAIL`（附理由） |

**示例：**

```
PROPERTY:  Container margin (query page header)
EXPECTED:  16px
EVIDENCE:  src/views/query/QueryPage.vue:47  padding: 16px 16px 0 16px
VERDICT:   PASS

PROPERTY:  Header background color token
EXPECTED:  #0d1b2a
EVIDENCE:  src/styles/b-tokens.css:12  --header-bg: #0d1b2a
VERDICT:   PASS

PROPERTY:  Font-size filter bar visibility
EXPECTED:  visible (B组 branch)
EVIDENCE:  src/components/FilterBar.vue:8  v-if="showFontSize" → showFontSize=true (B组 config)
VERDICT:   PASS

PROPERTY:  Table column width mode
EXPECTED:  Fixed 80px (B组 branch)
EVIDENCE:  src/components/TradeTable.js:34  width: 80
VERDICT:   PASS
```

---

## 核心约束

**1. 无证据不得判 PASS**

```
✗ 错误（不可接受）：
EVIDENCE:  （组件已按设计稿实现）
VERDICT:   PASS

✓ 正确：
EVIDENCE:  src/views/query/QueryPage.vue:47  padding: 16px 16px 0 16px
VERDICT:   PASS
```

**2. 任何 FAIL = 提交被阻止**

提交前 VVC 必须全部 PASS。有 FAIL 条目时，必须先修复再重新验证，不得绕过。

**3. 执行时机：提交前，不是提交后**

VVC 是**强制前置门控**，不是事后记录。

---

## VVC 属性从哪里来

VVC 的 PROPERTY 列表由人类（或 agent 辅助）从设计稿 / 规格文档中提取：

1. 读设计稿，列出所有可测量的视觉属性
2. 对于多目标场景，标注每个属性是"共用值"还是"目标特定值"
3. 写入 VVC 模板

**建议做法：** VVC 属性列表在任务开始前（与 CBDA/SEL/BER 同时）写好，不在执行中临时添加。

---

## 非 UI 场景的 VVC 应用

Module C 不限于视觉 UI 任务。核心模式（"把可验证属性转化为代码证据清单"）适用于任何"验证正确性"场景：

| 场景 | PROPERTY 来自 | EVIDENCE 是 |
|------|-------------|------------|
| UI 重设计 | 设计稿的视觉属性 | `file:line + CSS值` |
| 功能规格落地 | 规格文档的行为要求 | `file:line + 代码逻辑` |
| 接口契约验证 | backend-api.md 的字段/类型 | `file:line + 实际调用参数` |
| 权限隔离验证 | 隔离策略文档 | `file:line + 条件判断` |

在实际项目中，VVC 可应用于：
- 每个功能的视觉属性验证（以确定的基准页面为参照）
- build 验证作为替代截图对照的最低门控

---

## 在 CLAUDE.md 中如何引用 Module C

```markdown
**每功能 VVC 提交前审核（DocDrive Module C）**：
- 提交前必须完成 VVC 清单
- 每条属性必须有 file:line + 实测值 作为证据
- 无证据不得判 PASS；有 FAIL 不得提交
- VVC 清单提交给人工审核（审核门 4）
```
