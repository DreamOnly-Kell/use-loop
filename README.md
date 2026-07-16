# use-loop

Agent skill that runs multi-step work under **loop engineering** discipline — not freeform chat, and not “think longer until it feels done.”

**use-loop** is the skill / entry name (`/use-loop`).  
**Loop engineering** is the method: freeze a goal contract, keep explicit state, and drive a closed cycle with **external evidence**, **budget**, and **hard stop rules**. The human designs or confirms the contract and intervenes only at gates the agent escalates to.

中文说明见下方 [中文](#中文)。

---

## Why

Typical agent runs fail in quiet ways: long monologues with no observation, “done” claimed without tests, infinite retries on the same tool call, or pinging the user every micro-step.

This skill turns that into an auditable runtime:

1. **Goal Contract** before any action  
2. **State** as the single source of truth  
3. **Loop Tick** each iteration: select → execute → observe → apply → verify → policy  
4. Stop only on `success` | `needs_user` | `failed`

No contract → no loop.

---

## Install

Copy the skill into your agent’s skills directory (layout varies by product):

```bash
# example: Claude Code / compatible agents
cp SKILL.md /path/to/skills/use-loop/SKILL.md
# optional Chinese counterpart
cp SKILL.zh.md /path/to/skills/use-loop/SKILL.zh.md
```

Or clone this repo and point your agent at the skill path.

| File | Role |
|------|------|
| [`SKILL.md`](./SKILL.md) | English runtime (**canonical**) |
| [`SKILL.zh.md`](./SKILL.zh.md) | Chinese counterpart (same semantics; identifiers stay English) |

---

## Usage

**Triggers**

- User invokes `/use-loop`
- User asks to run under loop engineering (`用 loop 跑`, `按 loop engineering 做`, etc.)
- Task needs multi-step progress with **checkable** outcomes (implement, fix, research + deliver, tool loops)

**Do not** use this skill as an excuse to burn tokens without a contract.

### Goal Contract (shape)

```markdown
## Goal Contract
- **Goal:** <one sentence>
- **Success when:** <observable checklist — all must pass>
- **Failure OK when:** <honest failure reasons that still count as correct termination>
- **Never:** <hard prohibitions>
- **Budget:** max_steps=<N>, other limits=<...>
- **Human gates:** <ambiguity / auth / irreversible / budget>
```

### Loop Tick (shape)

```markdown
### Loop Tick <n>
- **Select:** <action> because <state-based reason>
- **Execute:** <what you did>
- **Observe:** <raw/external result — exit code, key output, errors>
- **Apply:** <how State changed — facts only>
- **Verify:** <which Success when items pass/fail now>
- **Policy:** continue | switch_strategy | escalate | success | fail
```

Full protocol, guards, terminal outputs, and anti-patterns: see [`SKILL.md`](./SKILL.md).

---

## Relationship to other skills

If this conflicts with a more specific skill (e.g. TDD, systematic debugging), keep those methods **inside** the loop as actions / verify steps — do not abandon the loop structure.

---

## License

[MIT](./LICENSE)

---

## 中文

**use-loop** 是一个 Agent skill：按 **loop engineering** 纪律跑多步、可验收的任务。

- **产品 / 调用名：** `use-loop`（`/use-loop`）  
- **方法名：** loop engineering（目标契约 → 行动 → 观察 → 验收 → 策略）

核心位移：不要等人说「继续下一步」。用**外部证据**、**显式状态**和**硬停条件**自己驱动循环；人只在门禁处介入。

| 文件 | 作用 |
|------|------|
| `SKILL.md` | 英文权威运行版 |
| `SKILL.zh.md` | 中文对照版（语义一致） |

安装：将上述文件拷入你所用 Agent 的 skills 目录。完整协议见 skill 正文。
