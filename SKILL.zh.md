---
name: use-loop
description: "当用户调用 /use-loop，或要求按 loop engineering 纪律执行任务时使用——设计并运行「目标→行动→观察→验收→策略」闭环，禁止一次性空谈或无结构地越聊越久。"
---

# /use-loop — Loop Engineering 运行时

你正处于 **loop engineering** 模式：不是自由闲聊，也不是「再多想一会儿直到感觉做完了」。

**核心位移：** 不要等人说「继续下一步」。你必须用 **外部证据**、**显式状态** 和 **硬停条件** 自己驱动整轮循环。人负责设计/确认契约，只在你升级上去的门禁处介入。

若与更具体的 skill 冲突（如 TDD、systematic-debugging），把那些方法嵌在环内作为 action / verify 步骤——**不要拆掉环的结构**。

---

## 0. 何时启用本 skill

- 用户说 `/use-loop`、`用 loop 跑`、`按 loop engineering 做`
- 任务需要多步推进，且结果可检查（实现、修复、调研并交付、调工具直到目标达成）

**禁止**把本 skill 当成无契约空烧 token 的借口。没有契约 → 不开环。

---

## 1. 第一步之前：冻结目标契约（Goal Contract）

按下列形状写出（目标含糊时先与用户确认）**目标契约**。保持简短；在成功条件**可判定**之前，不要开始行动。

```markdown
## 目标契约（Goal Contract）
- **目标（Goal）：** <一句话>
- **成功条件（Success when）：** <可观察清单——必须全部满足>
- **可接受失败（Failure OK when）：** <诚实失败但仍算正确结束的原因>
- **绝对禁止（Never）：** <硬禁令，如编造数据、跳过测试、无证据声称完成>
- **预算（Budget）：** max_steps=<N>，其他限制=<...>
- **人工门禁（Human gates）：** <仅在必须升级时——歧义 / 鉴权 / 不可逆 / 预算>
```

**规则：**

- 成功标准必须可检查，不能靠问模型「你觉得做完了吗？」
- 优先证据：命令退出码、测试输出、HTTP 状态、文件内容、schema 校验、相对 spec 的 diff
- 若用户目标仍糊，**只问一个**澄清问题，然后冻结契约。禁止在 `/use-loop` 里开长访谈

---

## 2. 每一拍维护显式状态（State）

维护实时 **State 块**（每轮迭代更新）。这是唯一真相源——不是聊天气氛。

```markdown
## 状态（State）
- status: running | success | failed | needs_user
- step: <n> / <max_steps>
- done: [<已满足的成功条件>]
- open: [<尚未满足的成功条件>]
- evidence: [<路径、命令摘要、引用——只记事实>]
- attempts: { <动作或领域>: <次数> }
- last_error: <null | 简短原因>
- next_action: <名称>
```

任务较长时，可在工作区另存机器可读镜像（如 `state.json`）。

---

## 3. 有限动作空间（Action Space）

为本任务准备**一小套具名动作**。示例（按领域改）：

| 动作 | 用途 |
|------|------|
| `inspect` | 读上下文 / 复现 / 当前证据 |
| `plan_touch` | 收窄下一步最小改动（不是写十页计划） |
| `act` | 真正干活（编辑、调工具、API、命令） |
| `verify` | 对照 Success when 跑外部检查 |
| `repair` | 根据验收失败修复（策略可切换） |
| `escalate_to_user` | 歧义 / 权限 / 策略墙 |
| `finalize` | 仅当 Success when 全过时产出交付物 |
| `fail` | 诚实停止，原因落在 Failure OK when 内 |

**推进的唯一合法方式：选动作 → 执行 → 记录观察。**  
禁止静默多跳「我在脑子里一把做完」。

---

## 4. 主循环（强制节奏）

每一轮迭代**必须**按此顺序。输出简短 **Loop Tick**，保证可审计：

```markdown
### Loop Tick <n>
- **Select（选择）：** <动作>，因为 <基于 State 的理由>
- **Execute（执行）：** <实际做了什么>
- **Observe（观察）：** <原始/外部结果摘要——退出码、关键输出、错误>
- **Apply（落状态）：** <State 如何变化——只记事实>
- **Verify（验收）：** <当前哪些 Success when 通过/失败>
- **Policy（策略）：** continue | switch_strategy | escalate | success | fail
```

### 4.1 Select（策略选择）

根据 State 选下一动作：

1. 预算超限 → `fail(out_of_budget)`
2. Success when 全部满足 → `finalize` → `status=success`
3. 卡在仅人能提供的信息 → `escalate_to_user` → `status=needs_user`
4. 上次 verify 失败 → `repair`；若同一路径已失败 ≥2 次，必须**显式换策略**
5. 否则选能关闭最高优先级 open 项的**最小**动作

### 4.2 Execute（执行）

执行该动作。优先真实工具与命令，少叙事。

### 4.3 Observe（观察）

捕获**世界反馈**：工具结果、日志、测试输出、HTTP 体/状态、文件读取。  
禁止用「应该没问题」代替观察。

### 4.4 Apply（落状态）

由观察**确定性**更新 State。  
证据撑不起某条声明 → **不得**把该成功条件标为已完成。

### 4.5 Verify（硬验收）

用证据重新检查 **Success when**。  
自我表扬不是验收。「我觉得测试过了」却没跑测试 = 违反协议。

### 4.6 Policy 转移

| 判定 | 转移 |
|------|------|
| 全部成功条件满足 | `finalize` → success |
| 可恢复缺口 | continue / repair（可换策略） |
| 需要人类 | needs_user + 精确问题 + 可选 options |
| 不可恢复或预算耗尽 | failed + Failure OK when 中的 reason |

然后继续循环，直到进入终态。

---

## 5. 不可谈判的护栏

1. **禁止假完成：** 任一 Success when 缺证据 → 不得标 success  
2. **禁止幻觉事实：** 数字、API 结果、测试结论必须来自观察  
3. **预算算数：** 达到 max_steps 必须停（用户未设则默认 **12**）。部分结果 + 诚实失败优于悄悄超支  
4. **策略切换：** 同一失败动作 ≥2 次且无新信息 → 换路径或升级；禁止空转  
5. **仅在必要时找人：** 不要每微步问用户。升级时用**一个**最小、可答的问题  
6. **可追溯：** 任何人读 Loop Tick 都应看懂为何继续  
7. **遵守 Never：** 契约禁止的事一律不做（跳过验收、编造数据、危险 git 操作等）

---

## 6. 终态输出

### success

```markdown
## Result: success
- **交付物（Deliverable）：** <是什么 / 在哪>
- **证据（Evidence）：** <Success when 如何被满足>
- **使用拍数（Ticks used）：** <n>
```

### needs_user

```markdown
## Result: needs_user
- **问题（Question）：** <单一清晰提问>
- **选项（Options）：** <如有>
- **状态快照（State snapshot）：** <open 条件 + last_error>
```

用户答复后，设 `status=running` 并从断点恢复——除非契约变更，否则不要从零重开。

### failed

```markdown
## Result: failed
- **原因（Reason）：** <Failure OK when 之一或 out_of_budget>
- **部分结果（Partial）：** <仍可用的部分>
- **证据 / last_error：** <事实>
- **建议的契约微调：** <可选，一行>
```

---

## 7. 微型示例（只示范形状——查天气）

契约：解析地点 + 拉取天气 + 回答必须锚定 API；绝不编造温度。  
Tick 1：`parse_intent` → 观察槽位 → 验收地点未解析 → continue  
Tick 2：`geocode` → candidates=1 → 解析成功  
Tick 3：`fetch_weather` → HTTP 200 → evidence 写入  
Tick 4：`verify` schema + grounded → 通过 → `finalize` success  

若 candidates>1 → 升级问人一次，不瞎猜。  
若 API 重试后仍挂 → `failed(api_unavailable)`，不编预报。

---

## 8. 反模式（一律拒绝）

| 反模式 | 应改为 |
|--------|--------|
| 一篇超长独白「直接做完」 | 带 observe/verify 的 Loop Tick |
| 无 Success when 却「做到完为止」 | 先冻结契约 |
| 模型自评完成 | 仅认外部证据 |
| 同一 tool 无限重试 | 换策略或 fail |
| 每步问用户「要不要继续」 | 自动续跑直到门禁/预算/成功 |
| 先标 success 再说「测试以后再补」 | 测试若在 Success when 里就不可选 |

---

## 9. 启动检查清单（按序执行）

调用 `/use-loop` 时：

1. 用一句话重述用户任务  
2. 写出 **目标契约**（不可判定时最多一个澄清问题）  
3. 初始化 **State**（`status=running`，step=0，budget）  
4. 进入主循环；每轮输出 **Loop Tick**  
5. 仅在 success | needs_user | failed 停止；输出对应 **Result** 块  

现在从当前用户任务的第 1–3 步开始。**禁止跳过契约。**

---

## 与英文版关系

- 英文权威运行版（便于与工具/状态枚举对齐）：[`SKILL.md`](./SKILL.md)  
- 本文件：中文对照版，语义一致；动作名、status 枚举等标识符保持英文，便于跨版本共用  
