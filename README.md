English | [한국어](README.ko.md)

# prometheus 🔥

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![code from: fivetaku/fablize](https://img.shields.io/badge/code%20from-fivetaku%2Ffablize-orange.svg)](https://github.com/fivetaku/fablize)

> 프로메테우스가 신에게서 **불**을 훔쳐 인간에게 가져다주었다. 이 하네스는 그 불이다.
>
> 강한 에이전트 모델이 가진 절차의 불을, **GLM-5.2**에게 가져다준다. 각 스토리는 불씨고, 검증 게이트를 통과할 때마다 불이 붙는다. "다 했다"는 말은 그 불을 직접 본 자만 할 수 있다.

## Why — 왜 불을 훔치는가

GLM-5.2는 강하지만, 자주 멈춘다 — 산출물을 만들고도 직접 돌려보지 않는다, "다 했다"고 하고 증거는 없다, 한 가지를 고치다가 다른 것을 놓친다. 이건 능력이 아니라 **습관**이다. 습관은 절차로 바꿀 수 있다.

prometheus는 그 절차의 불을 쥐여준다:

- 산출물을 만들면 **직접 실행하고 관찰**한다.
- 다중 작업은 **분해하고, 에이전트가 검증 가능한 `/goal` 문장으로 변환해 사용자에게 제안**한다. 사용자가 `/goal`(또는 `/goal replace`)로 세팅하면, 그때부터 ZCode 런타임이 매 턴 완료를 판정한다.
- 버그는 **재현 → 경쟁 가설 → 인과사슬**로 추적한다.

모델의 천장(ceiling)은 못 올린다. 하지만 천장까지 가는 길에 불을 밝혀준다.

## 불이란 무엇인가 (무엇을 가져다주는가)

| 절차 (불씨) | 효과 | 이유 |
|---|:--:|---|
| 검증 접지 (직접 실행·관찰) | 🔥 | 산출물이 정말 동작하는지 직접 본다 |
| 순차 `/goal` multi-story (ZCode 런타임) | 🔥 | 매 턴 끝에 런타임이 완료 판정 — 증거 없는 "완료"를 거부 |
| 체계적 조사 (재현 → 가설 → 인과사슬) | 🔥 | 첫 가설에 매달리지 않고 전체 사슬을 본다 |
| 능력(capability) | ❌ | 발견의 깊이, 창의적 디테일 — 모델의 몫 |

능력에 닿으면, prometheus는 흉내 내지 않고 에스컬레이션한다 (SKILL.md §5).

## Why we dropped goals.py

prometheus originally shipped its own multi-story engine, `scripts/goals.py` — four commands (`create`/`next`/`checkpoint`/`status`) that forced non-empty evidence on `complete` and a verify gate on the final story. It came from fablize.

**Once we learned ZCode provides a `/goal` command, we dropped goals.py.** The reason isn't mere feature overlap — it's a difference in enforcement strength.

- **goals.py's gate only fires if the agent voluntarily calls goals.py.** If the agent never runs `goals.py checkpoint`, the gate never gets a chance to fire. The most deterministic device has its **entrance** on the least deterministic layer (agent spontaneity).
- **ZCode `/goal` has no entrance.** At the end of every turn the runtime evaluates whether the goal is reached, and if the evidence doesn't support it, it continues to the next turn automatically. The agent cannot declare "done" — the system judges.
- **This isn't an abstract comparison — it was observed in this very project.** While demoing prometheus, the agent once finished a task that should have gone through the decomposition→verify-gate loop **without ever calling goals.py**. A hard gate is useless if you don't walk through its door. Had ZCode `/goal` been set instead, the system would have intervened every turn.

Conclusion: **the verification gate is delegated to ZCode `/goal`, not goals.py.** Note that the agent cannot set `/goal` directly — it designs verifiable goal sentences and proposes them to the user, who sets them via `/goal` (or `/goal replace`), after which the runtime enforces them each turn. What remains in prometheus is not a gate engine but the procedural wisdom to design good goal sentences for that gate (§1 context-gathering, independent-unit decomposition), plus the render-artifact verification (§4) and investigation discipline (§3) that the gate doesn't cover. That is prometheus's real value in the ZCode environment.

## What prometheus adds — 불씨를 키우는 것

핵심 설계상 결정: **분해의 품질**을 끌어올리는 절차를 강제한다.

1. **맥락 먼저** — 분해 전, 맥락 충분성을 4가지 질문으로 자가 진단.
2. **탐색 전용 서브에이전트** — 맥락이 부족하면 메인이 추측하지 말고 Explore 서브에이전트로 코드베이스/리서치 탐색.
3. **독립 단위 분해** — 맥락이 확보된 뒤에야, 각 스토리가 "독립 검증 가능 + 독립 수행 가능 + 1차원 목표"를 만족하도록 분해.

"분해하라"고만 하면 모델은 추측으로 스토리를 만든다. prometheus는 **어떻게 분해할지**를 절차로 강제한다.

## Status — 아직 검증 전

이 절차들이 **GLM-5.2에서 실제로 도움이 되는지는 아직 측정하지 않았다.** prometheus 자체가 그 측정을 할 계획이다 — 동일 작업을 prometheus on / off로 A/B하고, 완료율·증거 기반 완료 비율·재작업 횟수를 비교한다. 측정 전까지 "검증됨"이라고 주장하지 않는다. (코드의 초기 형태는 fivetaku/fablize에서 왔지만, 그곳의 효과 수치는 다른 모델 패밀리 기준이라 여기로 옮겨담을 수 없다.)

## Install — 불 받기

ZCode skill 표준 경로에 설치:

```bash
git clone https://github.com/tmdgusya/prometheus.git
cp -R prometheus/skills/prometheus ~/.agents/skills/
```

재시작 후 `/prometheus` 또는 "끝까지 해줘", "목표로 쪼개줘" 같은 트리거로 발동.

멀티 스토리의 완료 판정은 ZCode `/goal` 런타임이 내린다 — 단, 에이전트가 직접 goal을 세팅할 수는 없다. 에이전트는 검증 가능한 goal 문장을 설계해 사용자에게 제안하고, 사용자가 `/goal`로 세팅하면 그때부터 런타임이 매 턴 판정한다.

## How it burns — 동작

- **2+ 순차 목표** → 맥락 확보 → 독립 단위 분해 → **에이전트가 각 스토리를 `/goal` 문장으로 변환해 제안**, 사용자가 `/goal`(첫 스토리)·`/goal replace`(이후)로 세팅 → 런타임이 매 턴 완료 판정.
- **디버깅/원인 불명** → 재현 → 3+ 경쟁 가설 → 인과사슬 → 기각한 가설까지 보고.
- **렌더 산출물**(HTML/SVG/게임/차트) → 직접 실행하고 관찰. 정적 파싱은 well-formed만 확인할 뿐.
- **능력 천장** → 더 강한 추론 모드 권고 → 더 강한 모델에게 인계 → 인간에게 에스컬레이션.

## Honest limits — 불이 닿지 않는 곳

- **능력은 못 올린다.** 개방형 창작의 완성도·자발적 발견은 모델 선택의 영역이다.
- **GLM-5.2 효과는 미측정.** 위 Status 참고. prometheus는 절차를 제안할 뿐, 그 효과를 장담하지 않는다.

## Credits — 불의 출처

prometheus의 **코드**는 **[`fivetaku/fablize`](https://github.com/fivetaku/fablize)** (MIT)에서 출발했다 — 조사/검증 팩, 라우팅 설계가 그 형태다. (원래 `goals.py` 다중 스토리 엔진도 함께 가져왔으나, ZCode `/goal`과의 중복·강제 수준 차이로 폐기했다 — 위 "Why we dropped goals.py" 참고.) 원본은 다른 모델 패밀리를 위해 만들어졌고, 그곳의 효과 수치는 이곳으로 옮겨담지 않는다.

prometheus가 추가한 것: ZCode skill 포맷 적응, 분해 강제 절차(맥락 확보 + 서브에이전트 탐색 + 독립 단위 분해), **검증 게이트를 ZCode `/goal`에 위임하는 설계**, 불 메타포, 그리고 **GLM-5.2에서의 독자 벤치마크 계획**.

## License

MIT — [LICENSE](./LICENSE). 원본 fivetaku/fablize의 MIT 저작권은 보존된다.
