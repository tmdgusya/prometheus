[한국어](README.md) | English

# prometheus 🔥

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![code from: fivetaku/fablize](https://img.shields.io/badge/code%20from-fivetaku%2Ffablize-orange.svg)](https://github.com/fivetaku/fablize)

> **Three skills form one chain** — align the intent (`align`), forge the acceptance criteria (`forge`), execute and verify all the way (`prometheus`).
>
> Prometheus stole **fire** from the gods and brought it to humanity. This repo brings the procedural fire held by strong agent models to **GLM-5.2** — and aligns when to light it, and shapes how to forge it.

## The chain — roles of the three skills

```
align  →  forge   →  prometheus
intent    criteria    execution + verification
```

| Skill | Role | Output | When |
|---|---|---|---|
| **align** ⟷ | Aligns intent. Explores the codebase first and drives ambiguity to zero through iteration with the user. | Handoff document (intent + confirmed scope + done conditions) | Before multi-step implementation, when a request is ambiguous |
| **forge** ⚒️ | Converts the handoff's "done" conditions into objective acceptance criteria (command + expected result) that `/goal` can judge. | Acceptance criteria list | After align, before prometheus decomposition |
| **prometheus** 🔥 | Takes the acceptance criteria, turns them into `/goal` sentences, and executes + verifies all the way. | Execution results + evidence | After forge, the actual work |

Intended flow for multi-step work: secure intent with `align` → forge the done conditions with `forge` → execute all the way with `prometheus`. A trivially clear single-step task can skip align and go straight to forge/prometheus.

## Why — why steal the fire

GLM-5.2 is strong but often stops short — it produces an artifact yet never runs it, says "done" with no evidence, fixes one thing and drops another. It also starts coding against a guessed intent and goes the wrong way for an hour. This is not a capability gap, it's a **habit**. And a habit can be replaced with procedure.

This repo hands you that procedure in three stages:

- **align**: Before implementation, align intent. Surface ambiguities and iterate with the user until they're resolved.
- **forge**: Reforge the aligned "done" into verifiable acceptance criteria (command + expected result).
- **prometheus**: When it produces an artifact, **run and observe it directly**; for multi-step work, decompose so the runtime judges completion every turn; trace bugs via **reproduce → competing hypotheses → causal chain**.

It cannot raise the model's ceiling. But it lights the path all the way up to that ceiling.

## What prometheus adds — what is the fire

The effect of the procedures (sparks) prometheus brings:

| Procedure (spark) | Effect | Why |
|---|:--:|---|
| Verification grounding (direct run & observe) | 🔥 | See for yourself whether the artifact actually works |
| Sequential `/goal` multi-story (ZCode runtime) | 🔥 | The runtime judges completion at the end of every turn — it rejects "done" without evidence |
| Systematic investigation (reproduce → hypothesis → causal chain) | 🔥 | Don't cling to the first hypothesis; see the whole chain |
| Capability | ❌ | Depth of discovery, creative detail — the model's share |

When it reaches capability, prometheus escalates rather than fakes it (SKILL.md §5).

## Why we dropped goals.py

prometheus originally shipped its own multi-story engine, `scripts/goals.py` — four commands (`create`/`next`/`checkpoint`/`status`) that forced non-empty evidence on `complete` and a verify gate on the final story. It came from fablize.

**Once we learned ZCode provides a `/goal` command, we dropped goals.py.** The reason isn't mere feature overlap — it's a difference in enforcement strength.

- **goals.py's gate only fires if the agent voluntarily calls goals.py.** If the agent never runs `goals.py checkpoint`, the gate never gets a chance to fire. The most deterministic device has its **entrance** on the least deterministic layer (agent spontaneity).
- **ZCode `/goal` has no entrance.** At the end of every turn the runtime evaluates whether the goal is reached, and if the evidence doesn't support it, it continues to the next turn automatically. The agent cannot declare "done" — the system judges.
- **This isn't an abstract comparison — it was observed in this very project.** While demoing prometheus, the agent once finished a task that should have gone through the decomposition→verify-gate loop **without ever calling goals.py**. A hard gate is useless if you don't walk through its door. Had ZCode `/goal` been set instead, the system would have intervened every turn.

Conclusion: **the verification gate is delegated to ZCode `/goal`, not goals.py.** Note that the agent cannot set `/goal` directly — it designs verifiable goal sentences and proposes them to the user, who sets them via `/goal` (or `/goal replace`), after which the runtime enforces them each turn. What remains in prometheus is not a gate engine but the procedural wisdom to design good goal sentences for that gate (§1 context-gathering, independent-unit decomposition), plus the render-artifact verification (§4) and investigation discipline (§3) that the gate doesn't cover. That is prometheus's real value in the ZCode environment.

## Install — receiving the fire

Install all three skills to the standard ZCode skill path:

```bash
git clone https://github.com/tmdgusya/prometheus.git
cp -R prometheus/skills/align prometheus/skills/forge prometheus/skills/prometheus ~/.agents/skills/
```

After restart, each skill fires on its triggers:

- **align** ⟷ — `/align` or "정렬" (align), "의도 맞춰" (match intent), "문맥 파악해" (figure out context), "명확히 해줘" (clarify)
- **forge** ⚒️ — `/forge` or "forge", "인수조건 만들어" (make acceptance criteria), "검증 조건 세워" (set verification conditions)
- **prometheus** 🔥 — `/prometheus` or "끝까지 해줘" (take it all the way), "목표로 쪼개줘" (split into goals), "verify as you go"

The completion judgment for multi-story is made by the ZCode `/goal` runtime — but the agent cannot set the goal directly. The agent designs verifiable goal sentences and proposes them to the user; once the user sets them via `/goal`, the runtime judges each turn.

## How it burns — behavior

- **2+ sequential goals** → secure context → decompose into independent units → **the agent converts each story into a `/goal` sentence and proposes it**, the user sets it via `/goal` (first story) · `/goal replace` (subsequent) → the runtime judges completion every turn.
- **Debugging / unknown cause** → reproduce → 3+ competing hypotheses → causal chain → report even the hypotheses you rejected.
- **Render artifacts** (HTML/SVG/games/charts) → run and observe directly. Static parsing only checks well-formed.
- **Capability ceiling** → recommend a stronger reasoning mode → hand off to a stronger model → escalate to a human.

## Status — not yet validated

We have **not yet measured whether these procedures actually help on GLM-5.2.** prometheus itself plans to run that measurement — A/B the same task with prometheus on / off, and compare completion rate, the share of evidence-backed completions, and rework count. Until measured, we do not claim "validated." (The initial form of the code came from fivetaku/fablize, but its effectiveness figures are for a different model family and cannot be ported here.)

## Honest limits — where the fire doesn't reach

- **It cannot raise capability.** The polish of open-ended creation and spontaneous discovery belong to model choice.
- **GLM-5.2 effectiveness is unmeasured.** See Status above. prometheus only proposes procedure; it does not guarantee its effect.

## Credits — origin of the fire

The **code of prometheus** started from **[`fivetaku/fablize`](https://github.com/fivetaku/fablize)** (MIT) — the investigation/verification packs and the routing design are its form. (The original `goals.py` multi-story engine was brought over too, but was dropped due to overlap with and the enforcement-strength difference of ZCode `/goal` — see "Why we dropped goals.py" above.) The original was built for a different model family, and its effectiveness figures are not ported here.

**`align` and `forge` are newly created in this repo.** They do not originate from fablize — align proceduralizes "pre-implementation intent alignment" and forge proceduralizes "converting a handoff into verifiable acceptance criteria," each designed to pair with prometheus's execution/verification procedure.

What this repo added over fablize: adaptation to the ZCode skill format, **new `align`/`forge` skills**, the decomposition-forcing procedure (context securing + subagent exploration + independent-unit decomposition), **the design that delegates the verification gate to ZCode `/goal`**, and **a plan for an independent benchmark on GLM-5.2**.

## License

MIT — [LICENSE](./LICENSE). The MIT copyright of the original fivetaku/fablize is preserved.
