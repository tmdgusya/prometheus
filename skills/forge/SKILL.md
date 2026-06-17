---
name: forge
description: align이 만든 handoff 문서의 "완료" 조건을 ZCode /goal이 판정할 수 있는 객관적 인수조건(실행 명령 + 예상 결과 쌍)으로 변환합니다. 산출물은 인수조건 목록이며 코드는 쓰지 않습니다. handoff가 있을 때, 또는 사용자가 "forge", "인수조건 만들어", "검증 조건 세워"라고 할 때 사용하세요.
---

# forge ⚒️

> **원칙**: "완료"가 객관적이지 않으면 `/goal` 판정도, prometheus 검증도 자기기만이 된다. forge의 일은 handoff에 적힌 "What done looks like"를 **명령 + 예상 결과**로 변환해, 누가 봐도 참/거짓이 갈리는 조건으로 만드는 것이다. 코드는 쓰지 않는다 — 산출물은 인수조건 목록 하나뿐이다.

## 0. 번들 파일

- `~/.agents/skills/forge/packs/acceptance-criteria-template.md` — forge가 채우는 인수조건 템플릿.

## 1. 절차

1. **handoff를 읽는다.** 특히 "What done looks like", "In/Out scope", "Relevant context" — 인수조건은 이 세 항목에서 나온다. handoff가 없으면 align을 먼저 돌리라고 안내하고 멈춘다.
2. **각 "완료" 조건을 추출해 객관화한다.** 모호한 표현("깔끔하게", "잘 동작한다")은 전부 "명령 → 예상 결과" 쌍으로 바꾼다. 한 조건이 두 개 이상의 검증으로 쪼개지면 쪼갠다.
3. **누락을 찾는다.** scope에 있는데 인수조건에 안 나타난 항목이 있는지, 반대로 out-of-scope를 검증하려는 조건이 섞였는지 확인한다.
4. **목록을 사용자에게 제시하고 확인받는다.** 통과하면 끝. prometheus는 별도 호출로 이어받는다.

## 2. 좋은 / 나쁜 인수조건

| 나쁜 조건 (검증 불가) | 좋은 조건 (검증 가능) |
|---|---|
| 로그인 에러가 해결된다 | `npm test auth/login` → 9 passed, exit 0 |
| 코드가 깔끔해진다 | `ruff check .` → "All checks passed!" |
| UI가 잘 보인다 | 헤드리스 브라우저로 `/login` 렌더 → 에러 토스트 없음 |

차이: 좋은 조건은 판정자(사람이든 `/goal` 런타임이든)가 주관 없이 참/거짓을 가릴 수 있다.

## 3. 다른 스킬과의 관계

```
align  →  forge   →  prometheus
의도       인수조건     실행 + 검증
```

- **align**은 *무엇*을 하는지 확정한다. 산출물: handoff.
- **forge**는 *완료*를 판정 가능하게 만든다. 산출물: 인수조건 목록.
- **prometheus**는 인수조건을 받아 `/goal` 문장으로 옮기고 실행·검증한다.

forge는 prometheus §2-C("good goal sentence")가 요구하는 객관성을 미리 확보해준다. forge를 거친 조건은 prometheus가 다시 해석할 필요 없이 그대로 `/goal` 문장에 옮겨 담을 수 있어야 한다.
