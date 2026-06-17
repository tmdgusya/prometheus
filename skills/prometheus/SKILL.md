---
name: prometheus
description: A harness that forces verification, completion, and investigation as procedure so that GLM-5.2 (or ZCode's model) takes a task all the way to the end. It steals the procedural fire held by Fable (Anthropic) and brings it to GLM-5.2. Use it for multi-step work (2+ sequential goals), long autonomous tasks, debugging or root-cause investigation, and when producing render/executable artifacts (HTML, SVG, games, charts), or when the user says "prometheus", "끝까지 해줘" (take it all the way), "verify as you go", "목표로 쪼개줘" (split into goals), or "split into goals". It does not allow "done" without evidence — before claiming a task is finished, confirm you ran/observed it yourself. For multi-story, the agent converts each story into a verifiable /goal sentence and proposes it to the user; once the user sets it via /goal (or /goal replace), the ZCode runtime judges completion every turn.
---

# prometheus 🔥

> Just as Prometheus stole fire from the gods and brought it to humanity, this harness brings the working habits of strong agent models to **GLM-5.2** — run and observe directly, prove completion with evidence, investigate systematically. (The initial form of the code came from fivetaku/fablize, but its effectiveness figures are for a different model family and are not ported here. Effectiveness on GLM-5.2 is unmeasured.)
>
> Principle: a harness cannot raise a model's ceiling. It only makes the model reach its own ceiling — by forcing verification, completion, and investigation as **procedure**. If what blocks you is a capability ceiling (open-ended creativity, self-directed discovery), escalate (§5).

## 0. Bundle files (the sparks)

This skill keeps auxiliary files under `~/.agents/skills/prometheus/`. When you need the path, use the absolute paths below:

- `~/.agents/skills/prometheus/packs/investigation-protocol.txt` — debugging investigation discipline
- `~/.agents/skills/prometheus/packs/verification-grounding-pack.txt` — render-artifact verification discipline

> **Note — `goals.py` has been removed.** The multi-story engine (`goals.py`) that prometheus brought from fablize overlapped with ZCode's `/goal` command, and more importantly, goals.py's verification gate had a weakness: "it only fires if the agent voluntarily calls `goals.py`" — no matter how hard the gate is, a soft entrance makes it meaningless. ZCode `/goal` is evaluated by the **runtime** at the end of every turn, so that entrance does not exist at all. Therefore the verification gate is delegated to ZCode `/goal`, not `goals.py`. However, the agent cannot set `/goal` directly — the agent designs verifiable goal sentences and proposes them to the user, who sets them via `/goal` (or `/goal replace`). (For the background on this decision, see the "Why we dropped goals.py" section in the README.) Multi-story execution follows the procedure in §2.

## 1. Before decomposition — secure context (required, do not skip)

**The quality of decomposition depends on context.** Decomposing without context makes the model produce stories from guesses, and if those guesses are wrong the whole task goes the wrong way. So you must secure context **before** decomposing. Do not skip this step and jump straight to `/goal`.

### 1-A. Context sufficiency self-diagnosis

For the task you are about to decompose, check the following four points yourself. **If even one is "I don't know," context is insufficient:**

1. Do you specifically know the **codebase area** (files / modules / functions) this task will touch? — actual paths, not "somewhere."
2. Do you know the **current structure and rules** of that area (architecture, naming conventions, dependencies)?
3. Do you know the **success condition** of the task in observable form? — not "it works" but "this command / test produces this result."
4. (If it is a research task) Have you surveyed **existing known approaches / solutions**?

### 1-B. If context is insufficient — force an exploration-only subagent

If anything in 1-A is judged lacking, **do not guess yourself** — launch an exploration-only subagent (Agent tool, subagent_type=`Explore`) to gather context. The main agent does not explore and decompose at the same time — exploration goes to the subagent; decomposition is done by the main agent only after context is secured.

```text
Agent tool call (subagent_type: "Explore")
prompt:
  "Gather the context needed to decompose this task. Exploration scope: [codebase area or research topic].
   Specifically, find out:
   1) the actual paths and roles of the relevant files / modules
   2) the structure / rules / dependencies of that area
   3) how similar work is implemented in this codebase, if any
   4) entry points (tests, CLI, endpoints) that can make the success condition observable
   Return only the conclusion; do not dump file contents."
```

Only after receiving the exploration results do you re-check 1-A. If still insufficient, narrow the exploration scope and run it once more. **Never propose a `/goal` sentence to the user until context is sufficient.**

### 1-C. Once context is sufficient — decompose into independent units

Now design the stories. Each story must satisfy **all** of the following:

- **Independently verifiable**: when this story alone is completed, you must be able to prove that fact with a command / observation. ("I cleaned up the code" is unverifiable — "`ruff check .` reports 0 errors" is verifiable.)
- **Independently performable**: a story that cannot even start without the result of another story is too large or ordered wrong. (Having a dependency is OK — that is what sequential stories mean. But if "without reading the previous story's output you don't even know the starting point," the decomposition is incomplete.)
- **One-dimensional goal**: one story, one thing. "Add API + fix UI + tests" is three stories.
- **A verifiable goal sentence**: each story must be expressible in a form that fits ZCode `/goal` — "this command produces this result" (see §2-C).
- **The last one is a verification story**: the final story must be end-to-end verification.

### 1-D. Decomposition example (good / bad)

> User: "Fix the login error"

**❌ Bad decomposition** (guessing without context):
- G001: analyze the error
- G002: fix the error
- G003: verify

Problem: "analyze the error" has no observable artifact. "fix the error" tells you neither what nor how. Each story is not independently verifiable. Putting such vague goals into `/goal` makes the verifier fool itself on the vagueness, and self-deception sets in.

**✅ Good decomposition** (after 1-B exploration identified `auth/login.ts:42`, the error log, and the existing test pattern):
- G001: write an error reproduction script → running `node repro.js` prints the same error log
- G002: fix the cause at `auth/login.ts:42` → `npm test auth/login` passes
- G003: final verification → re-running `repro.js` shows no error + full `npm test` passes

The difference: in a good decomposition each story has a **specific path + observable evidence**. This is only possible because 1-B exploration happened.

## 2. Decomposition execution — sequential `/goal` multi-story (2+ sequential goals)

After securing context and designing the decomposition in §1, **convert each story into a ZCode `/goal` sentence and propose it to the user**, and once the user sets a goal with it, work on only that story. Each time a story ends, propose the next story sentence.

### 2-A. Why `/goal` (and not `goals.py`)

ZCode `/goal` is what makes the "verification gate" that prometheus once tried to implement directly via `goals.py` **enforced by the runtime**:

- **The agent does not declare completion.** At the end of every turn the runtime verifier evaluates whether the goal is reached, and if not, it automatically proceeds to the next turn.
- **It rejects proxy signals.** Passing tests / a successful build / writing code alone is not recognized as "done" — every condition the goal requires must be covered by evidence.
- **Uncertainty is treated as incomplete.** "Seems roughly done" is not done — verify further or keep working.

This is the decisive difference from `goals.py`. `goals.py` forced the checkpoint gate with `sys.exit()`, but **whether to call `goals.py` at all was the agent's voluntary choice** — a soft entrance neutralizes the gate. `/goal` has no such entrance; the runtime intervenes every turn.

### 2-B. Division of roles — the agent designs, the user sets

> **Important: the agent cannot set `/goal` directly.** `/goal` is a ZCode chat input command and is not reachable from the agent's bash shell or tools. All the agent can do is **build a verifiable goal sentence and propose it to the user**. The user does the setting. Do not fool yourself about this limit — any text written on the assumption that you can create a goal via a prompt fails immediately.

Roles:
- **Agent (prometheus)**: decomposes the stories via §1, converts each story into a `/goal` sentence in the 2-C format, and **presents it to the user**. Once the user has set it, work on only that story.
- **User**: copies the sentence the agent presented and sets it in the chat input as `/goal` (first story) or `/goal replace` (subsequent stories).
- **ZCode runtime**: runs the verifier on the set goal at the end of every turn. It judges regardless of the agent's completion declaration.

### 2-C. How to build a good `/goal` sentence

Turn each story into a verifiable sentence exactly in the form designed in 1-C. **Avoid vague wording; write the verification command and expected output directly inside the sentence.** The agent presents this sentence to the user verbatim.

Good goal sentence (verifiable):
```
/goal Fix the EAUTH cause at auth/login.ts:42. Verification: `npm test auth/login` passes with 9 passing, and `node repro.js` exits with exit code 0 and prints no error log.
```

Bad goal sentence (unverifiable, risk of self-deception):
```
/goal Resolve the login error elegantly.
```

The difference: a good goal has objective conditions (command + expected output) the verifier can judge. A bad goal, with vague words like "elegantly," fools even the verifier, and that becomes self-deception.

### 2-D. The sequential multi-story loop

```text
1. Convert each story from the list designed in §1 into a /goal sentence in the 2-C format, and present the entire list to the user at once.
2. Tell the user to set the first story sentence via /goal. (The user types it into the chat input.)
3. Once the user signals it is set (or the system indicates goal mode), work on only that story. Do not touch the other stories.
4. When the runtime verifier passes it (= a system reminder announces the goal is reached), present the next story sentence and tell the user to "replace via /goal replace."
5. Repeat until the last story. The last story must always be an end-to-end verification story.
```

Rules:
- Work on only **one** active goal at a time. This is not parallel.
- **The agent never types `/goal` itself.** Always present the sentence to the user and leave the setting to them.
- Do not declare a false "done" because you're stuck — the runtime will reject it anyway. Instead, gather more evidence or move into the §3 investigation procedure.
- A single-step task (decomposition yields 1 story) skips this loop — one goal is enough. However, the §1 context securing can still be useful even in a single step, e.g. for identifying a bug's cause.

### 2-E. Status tracking

You may record each story's progress in parallel with the TodoWrite tool (to show the user a visual progress indicator). However, **the completion judgment is made by the `/goal` runtime verifier, not by TodoWrite** — TodoWrite is a display, not a gate. Do not confuse them.

## 3. Deep investigation (debugging / unknown cause / review)

Read and follow `~/.agents/skills/prometheus/packs/investigation-protocol.txt`: first reproduce → form 3+ competing hypotheses → gather evidence per hypothesis → trace the full causal chain (removing the symptom is not removing the defect) → verify before and after → report the hypotheses you rejected. For reviews, report all low-confidence findings and filter them in a separate step.

## 4. Verification grounding (render/executable artifacts — always)

For artifacts whose correctness only surfaces when you run them (HTML, SVG, games, UI, charts), follow `~/.agents/skills/prometheus/packs/verification-grounding-pack.txt`: run in the real renderer → observe the actual output → fix what the observation reveals → re-run. Static parsing confirms well-formed, not correct.

Perform this loop yourself — it is not "tell the user to open it and stop." You run it and observe it yourself, via a headless browser, script execution, reading screenshots, etc.

## 4-1. Working style (always)

Start with the result. Stay within the requested scope (no incidental refactoring / abstraction). Ground every completion claim in this session's tool results. Confirm before destructive or hard-to-reverse actions.

## 5. At the capability ceiling (escalation)

Signals that you've hit the model ceiling: stuck on the same problem 2+ times; open-ended creation where the detail itself is the value; a deep review that needs discovery beyond the spec. These are capability, not procedure, and the harness cannot fill them. In order: (1) **GLM-5.2 has two thinking-effort levels (High, Max) and scales with reasoning difficulty.** Proceed at High by default, and when the above ceiling signals appear, raise it to **Max** to activate deeper reasoning — Z.ai recommends Max for coding work. If the client does not expose Max or the current session is pinned to High, recommend to the user switching to a stronger reasoning mode (Max); (2) if still insufficient, hand off in a new session to a stronger model, with an evidence package (symptoms, attempts, failure points, reproduction); (3) otherwise, honestly report the limits and mark where a human should step in.
