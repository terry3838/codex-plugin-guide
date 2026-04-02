# Codex Plugin for Claude Code 개요

## 원본 저장소 역할

- repo: `codex-plugin-cc`
- source: `https://github.com/openai/codex-plugin-cc`
- latest synced commit: `8e403f9d4b49`
- one-line summary: **Claude Code 안에서 Codex를 리뷰, 도전적 검토, 작업 위임 레인으로 호출하게 해 주는 플러그인**

## 이번 판단

기존 가이드는 설치 절차와 명령 설명은 충분했지만, 이 플러그인의 진짜 강점인 **lane 분리**가 전면에 잘 안 보였다.

그래서 이번 개정은 질문을 바꿨다.

- “무슨 명령이 있나?”
- 가 아니라
- **“지금 상황에서 어느 lane을 써야 하나?”**

를 더 빨리 답하게 만드는 방향으로 재구성했다.

## 최근 upstream 커밋

- `8e403f9 tests: reduce background task timing flakiness (#37)`
- `6360899 chore: bump plugin version to 1.0.2 (#74)`
- `65486f2 ci: add pull request workflow for tests and build (#73)`
- `64dc174 Fix /codex:rescue AskUserQuestion contract (#43)`
- `a126634 fix: make test suite portable across platforms (#56)`
- `1a79ae5 fix: resolve Windows ENOENT when spawning codex app-server (#55)`

## 원본에서 확인한 핵심 현실

### 1. 플러그인의 중심은 세 개의 Codex 역할이다

원본 README의 핵심 surface는 분명하다.

- `/codex:review`
- `/codex:adversarial-review`
- `/codex:rescue`

그리고 여기에 상태 관리 명령이 붙는다.

- `/codex:status`
- `/codex:result`
- `/codex:cancel`
- `/codex:setup`

즉 이 플러그인은 단일 “Codex 실행” 명령이 아니라, **역할이 나뉜 Codex 진입점 묶음**이다.

### 2. 일반 리뷰와 adversarial 리뷰는 대체 관계가 아니다

이 둘을 같은 축으로 보면 곧바로 헷갈린다.

- `review`는 구현 결함, 위험 신호, 읽기 전용 코드 검토에 맞다
- `adversarial-review`는 설계 선택, 트레이드오프, 숨은 가정을 흔드는 데 맞다

즉 둘의 차이는 강도보다 **질문 자체**에 있다.

### 3. `/codex:rescue`는 단순 위임보다 더 큰 runtime surface다

원본 README와 가이드 범주 문서를 같이 보면 `rescue`의 본질은 다음에 있다.

- rescue thread 유지
- `resume` / `fresh` 분기
- model / effort 선택
- foreground / background 전환
- 결과물과 session ID 추적

즉 rescue는 “Codex한테 시켜봐” 수준이 아니라, **조사/수정 작업을 lane 단위로 넘기는 메커니즘**이다.

### 4. 최근 upstream 변화도 안정성 쪽에 무게가 있다

최근 커밋에서 눈에 띄는 건 기능 폭발이 아니다.

- background task timing flakiness 완화
- AskUserQuestion contract 보정
- cross-platform portability 보강
- CI/PR workflow 추가

즉 이 플러그인은 이미 기본 기능을 세운 뒤, **실제 운영 안정성**을 더 다듬는 단계에 들어가 있다.

## 지금 학습자가 먼저 기억해야 할 것

1. 이 플러그인은 Claude 안에서 Codex를 “역할별 lane”으로 호출하게 한다.
2. `review`와 `adversarial-review`는 서로 대체재가 아니다.
3. `rescue`는 단순 장기 실행이 아니라 thread와 후속 작업을 포함한 위임 surface다.
4. `status/result/cancel`을 이해하지 못하면 background lane을 제대로 못 쓴다.
5. Review Gate는 기본값이 아니라, 중요 세션용 자동 안전망으로 보는 편이 맞다.
