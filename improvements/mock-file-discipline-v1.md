# 改进 #3：Mock 文件纪律

> 状态：已落地（2026-06-25）
> 来源项目：某 B 端权限管理项目（已脱敏）
> 影响规则层级：P1
> 已同步至：`governance/rule-governance.md`

---

## 问题描述

### 现象

某项目中，后端接口未就绪时需要在前端用假数据占位。实际出现了以下问题：

| 问题 | 具体表现 |
|------|---------|
| Mock 内联进共用 service | 假数据直接写在 service 函数里，和真实逻辑混在同一文件 |
| 无契约描述 | 假接口的 URL/入参/出参结构没有记录，后端就绪后不知道对齐什么 |
| TODO 只登记了一处 | 只在代码处留了注释，没有在 TODO-backend.md 同步 |
| 清理无序 | 后端就绪后，mock 代码和 TODO 没有系统性清理 |

### 根因

**没有把"假接口是临时状态"这件事制度化。** 假接口被当作普通实现来处理，没有对应的隔离、描述和删除机制。

结果是：随着项目推进，哪些接口是假的、假到什么程度、对应后端什么 TODO，没有人能快速回答。

---

## 解决方案

### 假接口四件套（缺一不可）

每接入一个假接口，必须同时产出以下四样东西：

**① 独立 mock 数据文件（`xxx.mock.ts`）**

```typescript
// xxx.mock.ts
export const mockUserList = [
  { id: 1, name: '张三', role: 'admin' },
  // ...
]
```

真实 service 只 import 它返回，不把数据内联进 service 文件。

**② 契约描述文件（`xxx.mock.md`，与 mock 文件同级）**

```markdown
# 假接口描述

**对应 TODO ID**：BE-XXX-001
**URL**：POST /api/users/list
**入参**：{ keyword?: string, page: number, size: number }
**出参**：{ list: User[], total: number }
**当前行为**：返回 mock 数据，忽略入参
**后端就绪后**：删本文件 + xxx.mock.ts，改 service 为真实请求
```

**③ 双处 TODO**

```typescript
// service 代码处
// TODO(BE-XXX-001): 用真实接口替换 mock — 见 docs/TODO-backend.md
import { mockUserList } from './xxx.mock'
```

同时在 `TODO-backend.md` 登记同一个 ID。

**④ 后端就绪后的清理清单**

后端交付时，必须同时：
- 删 `xxx.mock.ts`
- 删 `xxx.mock.md`
- 删 service 中的 TODO 注释
- 删 `TODO-backend.md` 中的对应条目
- 改 service 为真实请求

**清理不完整 = 这个接口未完成。**

### 核心禁止

- ❌ 不得把 mock 数据内联进共用 service 文件
- ❌ 不得用 mock 冒充真实通过验证
- ❌ 不得修改登录/鉴权/请求拦截等全局逻辑来"让 mock 跑起来"
- ❌ mock 期间不得标 feature 为 passing（UI 完成但有 mock = `pending-backend`）

---

## 落地情况

### 已更新的文件

| 文件 | 更新内容 |
|------|----------|
| `governance/rule-governance.md` P1 | 更新「假接口纪律」为完整四件套要求 |

### 适用范围

任何有后端依赖、需要前端先行开发的项目。
前后端同步开发时尤其重要——假接口的生命周期往往跨越多个会话，最容易产生遗漏。
