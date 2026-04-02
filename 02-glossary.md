# 용어 사전

이 문서는 Codex Plugin for Claude Code를 이해할 때 자주 헷갈리는 개념을 **lane / runtime / 작업 관리** 기준으로 다시 정리한다.

---

## A

### Adversarial Review

**정의:** 구현이 “돌아가는가”를 넘어서, **그 접근 자체가 옳은지**를 공격적으로 검증하는 리뷰 방식.

일반 리뷰와의 차이는 엄격도보다 관점에 있다.

- `review` → 구현 결함 중심
- `adversarial-review` → 설계 판단, 트레이드오프, 숨은 가정 중심

---

## B

### Background Execution

**정의:** Codex 작업을 백그라운드로 실행하고, Claude Code 흐름을 바로 돌려받는 방식.

이 플러그인에서 background는 선택 옵션이 아니라 **긴 작업을 다루는 기본 운영 패턴**이다. 이후에는 `/codex:status`, `/codex:result`, `/codex:cancel`이 함께 따라온다.

---

## C

### Codex Companion

**정의:** 플러그인의 런타임 코어. review, task, status, result, cancel 같은 하위 기능을 Codex 쪽 실행 경로로 라우팅한다.

이 개념을 이해하면 플러그인을 명령어 모음이 아니라 **runtime bridge**로 보기 쉬워진다.

### Challenge Lane

**정의:** `/codex:adversarial-review`를 중심으로, 설계 판단을 압박하고 반론을 수집하는 작업 흐름.

배포 전 검증이나 고위험 변경 검토에서 특히 중요하다.

---

## D

### Delegate Lane

**정의:** `/codex:rescue`로 대표되는 작업 위임 흐름.

단순 호출이 아니라 다음을 함께 포함한다.

- 새 thread 시작 또는 기존 thread 이어가기
- model/effort 조절
- foreground/background 선택
- 결과와 후속 작업 추적

---

## E

### Effort Level

**정의:** Codex가 문제를 얼마나 깊게 파고들지 정하는 추론 깊이 설정.

일반적으로:
- 낮은 effort → 빠른 triage, 작은 수정
- 높은 effort → 복잡한 조사, 다단계 분석

이 값은 “좋다/나쁘다”가 아니라 **작업 크기와 비용 사이의 trade-off**다.

---

## F

### Foreground Execution

**정의:** 작업이 끝날 때까지 현재 흐름에서 결과를 기다리는 방식.

짧은 리뷰나 빠른 확인에는 좋지만, 긴 rescue나 멀티파일 리뷰는 background가 더 자연스럽다.

---

## L

### Lane

**정의:** 이 가이드가 플러그인 명령을 이해하는 기본 단위.

권장 lane 구분은 다음 네 가지다.

- **review lane** — `/codex:review`
- **challenge lane** — `/codex:adversarial-review`
- **delegate lane** — `/codex:rescue`
- **operate lane** — `setup`, `status`, `result`, `cancel`

명령어를 낱개로 외우는 대신 lane으로 기억하면 실제 사용 판단이 쉬워진다.

---

## O

### Operate Lane

**정의:** Codex 작업을 설치·확인·추적·취소하는 운영 흐름.

핵심 명령:
- `/codex:setup`
- `/codex:status`
- `/codex:result`
- `/codex:cancel`

이 lane은 화려하진 않지만, background 작업을 실무에서 굴리려면 가장 중요하다.

---

## R

### Rescue Thread

**정의:** `/codex:rescue`로 시작된 Codex 작업 맥락.

이 thread 개념이 있기 때문에:
- 이전 조사 결과를 이어서 쓰는 `--resume`
- 새 시도로 분기하는 `--fresh`

같은 선택이 의미를 가진다.

### Review Gate

**정의:** Claude Code가 코드를 수정한 뒤 자동으로 Codex 리뷰를 돌려, 문제가 있으면 진행을 막는 안전망.

강력하지만 비용이 있다.

- 장점: 자동 품질 안전망
- 단점: 장기 루프, 속도 저하, 사용량 증가

즉 기본값으로 항상 켜는 기능이 아니라, **중요 세션에서 의도적으로 켜는 자동화 옵션**으로 보는 편이 맞다.

### Review Lane

**정의:** `/codex:review`를 중심으로, 현재 변경을 읽기 전용으로 검토하는 흐름.

가장 먼저 익혀야 하는 lane이며, 커밋 전 셀프 리뷰에 가장 잘 맞는다.

---

## S

### Scope

**정의:** Codex가 어떤 변경 범위를 대상으로 볼지 정하는 기준.

예:
- 현재 워킹트리 변경
- 특정 base ref 대비 branch diff
- 자동 선택

좋은 리뷰 품질은 종종 좋은 scope 선택에서 시작한다.

### Spark Model Alias

**정의:** `spark` 같은 짧은 입력을 실제 Codex 모델 이름으로 매핑하는 별칭.

빠른 탐색이나 값싼 패스가 필요할 때 유용하지만, 깊은 조사에는 더 강한 모델/effort 조합이 나을 수 있다.

### Status / Result / Cancel

**정의:** background Codex 작업의 상태 확인, 결과 조회, 취소를 담당하는 작업 관리 명령 묶음.

이 세 개를 이해하지 못하면 background lane은 사실상 반쪽짜리다.

---

## W

### Wait vs Background

**정의:** 현재 자리에서 끝까지 기다릴지, 작업을 뒤로 돌리고 나중에 확인할지를 고르는 실행 모드 선택.

판단 기준은 단순하다.

- 짧고 작은 작업 → `--wait`
- 오래 걸리거나 병행이 필요한 작업 → `--background`
