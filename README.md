# Codex Plugin for Claude Code 학습 가이드

Codex Plugin for Claude Code는 단순히 Claude Code 안에서 Codex를 부르는 연결 장치가 아니다. 이 플러그인의 진짜 가치는 **Claude가 하던 흐름 속에 Codex용 lane을 끼워 넣는 것**에 있다.

핵심 lane은 세 가지다.

- **review lane** — 현재 변경을 읽기 전용으로 검토
- **challenge lane** — 설계 판단을 일부러 압박하며 검증
- **delegate lane** — 조사/수정 작업 자체를 Codex에 넘김

이 가이드는 원본 README의 명령 설명을 한국어로 반복하는 데서 멈추지 않고, **실무에서 언제 어떤 lane을 써야 하는지**가 보이도록 구조를 다시 세운다.

---

## 현재 기준선

- upstream plugin repo: `codex-plugin-cc`
- latest synced commit: `8e403f9d4b49`
- 최근 upstream 신호:
  - background task timing 안정화
  - plugin version `1.0.2` 정렬
  - PR workflow 추가
  - `/codex:rescue` AskUserQuestion contract 수정
  - Windows app-server spawn 호환성 보강

즉 이 플러그인은 “명령 몇 개 추가한 편의 도구”가 아니라, **Claude Code ↔ Codex 사이의 실무 runtime bridge**로 읽는 편이 맞다.

---

## 이 플러그인을 한 문장으로 말하면

**Claude Code 안에서 Codex를 리뷰어·도전자·작업 위임자 역할로 호출하게 해 주는 워크플로우 플러그인**이다.

원본 README가 강조하는 것도 결국 이 세 축이다.

- `/codex:review`
- `/codex:adversarial-review`
- `/codex:rescue` + `status/result/cancel`

핵심은 Codex를 “추가 모델”처럼 쓰는 게 아니라, **역할이 다른 별도 작업 레인**으로 다루는 데 있다.

---

## 왜 이 가이드를 다시 봐야 하나

### 1. 기존 frontdoor는 설치/명령 설명이 중심이었다

그 자체로는 쓸 만했지만, 처음 읽는 사람은 여전히 이런 질문이 남는다.

- 언제 `review`면 충분하고 언제 `adversarial-review`까지 가야 하나?
- `rescue`는 단순한 서브에이전트 호출과 무엇이 다른가?
- foreground / background를 어떤 기준으로 고르나?
- Review Gate는 멋져 보이는데 실제로 언제 켜야 하나?

즉 정보는 있었지만 **의사결정 순서**가 약했다.

### 2. 이 플러그인의 진짜 가치 포인트는 lane 분리다

이 플러그인을 잘 쓰는 사람은 명령어를 많이 아는 사람이 아니라, 상황별로 lane을 올바르게 고르는 사람이다.

- 코드 변경을 빠르게 훑고 싶다 → `review`
- 설계 판단을 공격적으로 검증하고 싶다 → `adversarial-review`
- 조사/수정 자체를 넘기고 싶다 → `rescue`
- 오래 걸리는 작업을 추적해야 한다 → `status/result/cancel`
- Claude의 수정 턴마다 자동 안전망을 걸고 싶다 → Review Gate

즉 문서의 중심도 명령 나열보다 **lane 선택 규칙**이 되어야 한다.

### 3. 최근 upstream도 runtime 안정성 쪽을 강화하고 있다

최근 커밋을 보면 관심사가 분명하다.

- background timing flakiness 완화
- AskUserQuestion contract 보정
- Windows spawn portability 보강
- CI/PR workflow 추가

즉 이 플러그인은 설명문보다 **실제 실행 안정성**이 중요해지는 단계다.

---

## 빠른 시작: 가장 먼저 익혀야 할 3개 lane

### 1) 리뷰 lane

```bash
/codex:review --background
/codex:status
/codex:result
```

언제 쓰나:
- 커밋 전 셀프 리뷰
- 변경이 여러 파일에 걸쳐 있을 때
- Claude가 작성한 패치를 한 번 더 읽기 전용으로 검토하고 싶을 때

### 2) 챌린지 lane

```bash
/codex:adversarial-review --base main 인증과 롤백 설계가 충분히 안전한지 검토해줘
```

언제 쓰나:
- 중요한 설계 결정 전
- 배포 직전 리스크 검증
- 일반 리뷰로는 놓치기 쉬운 트레이드오프를 흔들어 보고 싶을 때

### 3) 위임 lane

```bash
/codex:rescue --background flaky integration test의 원인을 조사하고 가장 작은 안전한 수정안을 제시해줘
/codex:status
/codex:result
```

언제 쓰나:
- 원인 조사 자체를 넘기고 싶을 때
- Claude가 막혔을 때
- 장시간 작업을 백그라운드로 돌리고 싶을 때

---

## 전체 구조를 어떻게 이해하면 좋은가

```mermaid
graph TD
  Claude[Claude Code] --> Review[/codex:review]
  Claude --> Adversarial[/codex:adversarial-review]
  Claude --> Rescue[/codex:rescue]
  Review --> Companion[codex-companion runtime]
  Adversarial --> Companion
  Rescue --> Companion
  Companion --> Codex[Codex CLI / app server]
  Companion --> Jobs[tracked jobs]
  Jobs --> Status[/codex:status]
  Jobs --> Result[/codex:result]
  Jobs --> Cancel[/codex:cancel]
  Hooks[hooks + review gate] --> Companion
```

이 구조에서 중요한 건 하나다.

- Claude Code는 사용자-facing 진입점
- Codex는 실제 리뷰/위임 실행 엔진
- 플러그인은 그 사이에서 **명령을 lane으로 정리하고 상태를 추적하는 bridge** 역할을 한다

---

## 추천 읽기 순서

1. `sections/01-overview.md` — 이 플러그인이 실제로 무엇인지
2. `01-learning-paths.md` — 어떤 순서로 써야 하는지
3. `01-usage-scenarios.md` — 대표 실무 플로우
4. `categories/code-review.md` — review / adversarial / review gate
5. `categories/task-delegation.md` — rescue와 thread/모델/effort
6. `categories/operations.md` — setup / status / result / cancel / hook
7. `02-glossary.md` — 헷갈리는 용어 정리

---

## 흔한 오해

### 오해 1 — `/codex:review`가 있으면 `/codex:adversarial-review`는 덜 중요하다

아니다. 둘은 엄격도 차이가 아니라 **관점 차이**다.

### 오해 2 — `/codex:rescue`는 그냥 길게 실행되는 Codex 호출일 뿐이다

아니다. rescue는 thread, resume/fresh, model/effort, background job tracking까지 포함한 **작업 위임 lane**이다.

### 오해 3 — Review Gate는 항상 켜 두는 게 좋다

아니다. 강력하지만 루프와 사용량 오버헤드가 있다. 중요한 세션에서만 의도적으로 켜는 편이 맞다.

---

## 다음 문서

- lane 구조를 먼저 잡으려면 `sections/01-overview.md`
- 대표 실무 흐름은 `01-usage-scenarios.md`
- 리뷰/위임/운영 세부는 `categories/` 문서들

<!-- GUIDE_SYNC:START -->
## 자동 동기화 상태

- origin repo: `codex-plugin-cc`
- latest source commit: `807e03ac9d5a`
- sync mode: `update`
- 영향 분류: 스킬/플러그인, 테스트/검증

### 이번 반영 포인트

origin 변경 파일을 기준으로 guide 문서의 관련 섹션을 다시 읽고 반영했습니다. 핵심 영향 영역: 스킬/플러그인, 테스트/검증.

### 최근 upstream 커밋

- `807e03a fix: bump plugin version to 1.0.4 (#244)`
- `bb38412 fix: route /codex:rescue through the Agent tool to stop Skill recursion (#234) (#235)`

### 변경 파일 샘플

- `.claude-plugin/marketplace.json`
- `package-lock.json`
- `package.json`
- `plugins/codex/.claude-plugin/plugin.json`
- `plugins/codex/commands/rescue.md`
- `tests/commands.test.mjs`

> 이 블록은 guide sync가 자동 갱신합니다.
<!-- GUIDE_SYNC:END -->
