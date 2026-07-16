---
name: use-loop
description: "Use when the user invokes /use-loop or asks to run a task under loop engineering discipline — design and execute a closed goal→act→observe→verify→policy cycle instead of one-shot or endless freeform chatting."
---

# /use-loop — Loop Engineering Runtime

You are running under **loop engineering**, not freeform chat and not "think longer until it feels done."

**Core shift:** Do not wait for the human to say "continue next step." Drive the cycle yourself using **external evidence**, **explicit state**, and **hard stop rules**. The human designs/approves the contract and intervenes only at gates you escalate to.

If this conflicts with a more specific skill (e.g. TDD, systematic-debugging), keep those methods **inside** the loop as actions/verify steps — do not abandon the loop structure.

---

## 0. When this skill applies

- User says `/use-loop`, "用 loop 跑", "按 loop engineering 做"
- Task needs multi-step progress with checkable outcomes (implement, fix, research+deliver, call tools until goal met)

**Do not** use this skill as an excuse to burn tokens without a contract. No contract → no loop.

---

## 1. Before step 1: freeze the Goal Contract

Write (or confirm with the user if ambiguous) a **Goal Contract** in this shape. Keep it short; do not start acting until success is *decidable*.

```markdown
## Goal Contract
- **Goal:** <one sentence>
- **Success when:** <observable checklist — all must pass>
- **Failure OK when:** <honest failure reasons that still count as correct termination>
- **Never:** <hard prohibitions, e.g. invent data, skip tests, claim done without evidence>
- **Budget:** max_steps=<N>, other limits=<...>
- **Human gates:** <only when you must escalate — ambiguity / auth / irreversible / budget>
```

**Rules:**

- Success criteria must be checkable without asking the model "do you feel done?"
- Prefer evidence: command exit codes, test output, HTTP status, file contents, schema validation, diff against spec.
- If the user goal is still mushy, **one** clarifying question, then freeze the contract. Do not start a long discovery interview inside `/use-loop`.

---

## 2. Maintain explicit State every turn

Keep a live **State block** (update every iteration). This is the source of truth — not chat vibes.

```markdown
## State
- status: running | success | failed | needs_user
- step: <n> / <max_steps>
- done: [<success criteria already met>]
- open: [<success criteria not yet met>]
- evidence: [<paths, command summaries, citations — facts only>]
- attempts: { <action_or_area>: <count> }
- last_error: <null | short cause>
- next_action: <name>
```

Optionally keep a machine-friendly mirror (`state.json`) in the workspace if the task is long.

---

## 3. Finite Action Space

Prefer a **small named action set** for this task. Examples (adapt per domain):

| Action | Purpose |
|--------|---------|
| `inspect` | Read context / repro / current evidence |
| `plan_touch` | Narrow the next minimal change (not a 10-page plan rewrite) |
| `act` | Do the work (edit, tool call, API, command) |
| `verify` | Run external checks against Success when |
| `repair` | Fix based on verify failure (strategy may change) |
| `escalate_to_user` | Ambiguity / permission / policy wall |
| `finalize` | Emit deliverable only when Success when all pass |
| `fail` | Honest stop with reason in Failure OK when |

**You may only progress by choosing an action, executing it, and recording observation.**  
No silent multi-hop "I'll just finish everything in my head."

---

## 4. The Loop (mandatory rhythm)

Each iteration **must** follow this order. Emit a short **Loop Tick** so the run is auditable:

```markdown
### Loop Tick <n>
- **Select:** <action> because <state-based reason>
- **Execute:** <what you did>
- **Observe:** <raw/external result summary — exit code, key output, errors>
- **Apply:** <how State changed — facts only>
- **Verify:** <which Success when items pass/fail now>
- **Policy:** continue | switch_strategy | escalate | success | fail
```

### 4.1 Select (Policy)

Choose the next action from State:

1. If budget exceeded → `fail(out_of_budget)`
2. If Success when all pass → `finalize` → `status=success`
3. If blocked on human-only info → `escalate_to_user` → `status=needs_user`
4. If last verify failed → `repair` with **explicit strategy change** if same approach already failed ≥2 times
5. Else pick the smallest action that closes the highest-priority open criterion

### 4.2 Execute

Do the action. Prefer real tools and commands over narration.

### 4.3 Observe

Capture **world feedback**: tool results, logs, test output, HTTP bodies/status, file reads.  
Do not replace observation with "it should be fine."

### 4.4 Apply

Update State deterministically from observation.  
If evidence does not support a claim, **do not** mark the criterion done.

### 4.5 Verify (hard)

Re-check **Success when** using evidence.  
Self-praise is not verify. "I believe tests pass" without running them is a protocol violation.

### 4.6 Policy transition

| Verdict | Transition |
|---------|------------|
| All success criteria met | `finalize` → success |
| Recoverable gap | continue / repair (maybe switch strategy) |
| Needs human | needs_user + precise question + options if any |
| Unrecoverable or budget | failed + reason from Failure OK when |

Then loop until terminal status.

---

## 5. Non-negotiable guards

1. **No fake done:** Never mark success if any Success when item lacks evidence.
2. **No hallucination of facts:** Numbers, API results, test outcomes must come from observation.
3. **Budget is real:** Stop at max_steps (default **12** if user did not set one). Partial results + honest failure beat silent overrun.
4. **Strategy switch:** Same failing action ≥2 times without new information → change approach or escalate; do not thrash.
5. **Human gates only when needed:** Do not ping the user each micro-step. Escalate with a minimal, answerable question.
6. **Traceability:** Anyone reading Loop Ticks should see why you continued.
7. **Never list:** whatever the contract forbids (skipping verify, inventing data, force-pushing, etc.).

---

## 6. Terminal outputs

### success

```markdown
## Result: success
- **Deliverable:** <what / where>
- **Evidence:** <how Success when was met>
- **Ticks used:** <n>
```

### needs_user

```markdown
## Result: needs_user
- **Question:** <single clear ask>
- **Options:** <if applicable>
- **State snapshot:** <open criteria + last_error>
```

After the user answers, set `status=running` and resume the loop — do not restart from zero unless the contract changed.

### failed

```markdown
## Result: failed
- **Reason:** <from Failure OK when or out_of_budget>
- **Partial:** <what is usable>
- **Evidence / last_error:** <facts>
- **Suggested next contract tweak:** <optional, one line>
```

---

## 7. Mini example (shape only — weather)

Contract: resolve location + fetch weather + answer grounded in API; never invent temp.  
Tick 1: `parse_intent` → observe slots → verify location not resolved → continue  
Tick 2: `geocode` → candidates=1 → resolve  
Tick 3: `fetch_weather` → HTTP 200 → evidence filled  
Tick 4: `verify` schema + grounded → pass → `finalize` success  

If candidates>1 → escalate once, not guess. If API down after retries → failed(api_unavailable), not a made-up forecast.

---

## 8. Anti-patterns (refuse these)

| Anti-pattern | Do instead |
|--------------|------------|
| One giant monologue that "finishes" the task | Loop ticks with observe/verify |
| "Continue until done" with no Success when | Freeze contract first |
| Model self-score as done | External evidence only |
| Infinite retry same tool call | Strategy switch or fail |
| Ask user "should I continue?" every step | Auto-continue until gate/budget/success |
| Mark success then "we can fix tests later" | Tests in Success when → not optional |

---

## 9. Startup checklist (do in order)

When `/use-loop` is invoked:

1. Restate the user task in one sentence.
2. Write **Goal Contract** (ask at most one clarification if undecidable).
3. Initialize **State** (`status=running`, step=0, budget).
4. Enter the loop; emit **Loop Tick** each iteration.
5. Stop only on success | needs_user | failed; emit the matching **Result** block.

Begin now with step 1–3 for the user's current task. Do not skip the contract.
