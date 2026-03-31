# 용어 사전

이 문서는 Codex 플러그인에서 사용하는 고유 개념과 혼동하기 쉬운 용어를 정리합니다.

---

## 플러그인 고유 개념

| 용어 | 설명 |
|------|------|
| **Review Gate** | Claude Code의 Stop 훅에 연결된 자동 리뷰 메커니즘. Claude Code가 코드를 수정하면 Codex가 해당 변경을 자동 리뷰하고, ALLOW(통과) 또는 BLOCK(차단) 판정을 내린다. `/codex:setup --enable-review-gate`로 활성화한다. |
| **Rescue Thread** | `/codex:rescue`로 시작된 Codex 작업 스레드. 하나의 rescue thread는 연속적인 조사/수정 맥락을 유지하며, `--resume-last`로 이전 스레드를 이어갈 수 있다. |
| **Adversarial Review** | 일반 리뷰와 달리 구현의 정당성을 의도적으로 도전(adversarial stance)하는 리뷰 방식. 코드 결함이 아니라 설계 판단, 숨은 가정, 트레이드오프를 지적한다. |
| **Codex Companion** | `codex-companion.mjs` 스크립트. 플러그인의 모든 명령어를 처리하는 런타임 코어로, review, task, status, result, cancel 등의 하위 명령을 Codex CLI로 라우팅한다. |
| **App Server** | Codex가 제공하는 로컬 서버 프로토콜. 플러그인은 이 프로토콜을 통해 Codex CLI와 통신한다. [Codex App Server 문서](https://developers.openai.com/codex/app-server/) 참고. |
| **Effort Level** | Codex의 추론 깊이를 조절하는 설정. `none`, `minimal`, `low`, `medium`, `high`, `xhigh` 중 선택한다. 높을수록 깊이 분석하지만 시간과 비용이 증가한다. |
| **Background Execution** | `--background` 플래그로 실행하는 비동기 방식. Codex 작업이 백그라운드에서 진행되며, `/codex:status`로 상태를 확인하고 `/codex:result`로 결과를 조회한다. |
| **Foreground Execution** | `--wait` 플래그 또는 기본 동작으로 실행하는 동기 방식. 작업이 끝날 때까지 Claude Code가 결과를 기다린다. |
| **Resume vs Fresh** | `--resume`(또는 `--resume-last`)는 이전 Codex rescue thread를 이어서 작업한다. `--fresh`는 새 스레드를 시작한다. 둘 다 지정하지 않으면 이전 스레드가 있을 때 선택을 묻는다. |
| **Scope** | 리뷰 대상 범위. `working-tree`는 커밋하지 않은 변경사항, `branch`는 `--base <ref>`로 지정한 브랜치와의 diff를 대상으로 한다. `auto`는 상황에 따라 자동 선택한다. |
| **ALLOW / BLOCK 판정** | Review Gate의 결과. ALLOW는 "문제 없음, 계속 진행", BLOCK은 "문제 발견, 수정 필요"를 의미한다. 코드 변경이 없는 턴은 항상 ALLOW를 반환한다. |

---

## 혼동하기 쉬운 개념

| 비교 | 설명 |
|------|------|
| **`/codex:rescue` vs `codex-rescue` 에이전트** | `/codex:rescue`는 사용자가 직접 호출하는 슬래시 명령어다. `codex-rescue`는 Claude Code의 서브에이전트로, Claude Code가 막혔을 때 **자동으로** Codex에 작업을 위임하는 역할을 한다. `/codex:rescue`를 실행하면 내부적으로 `codex-rescue` 에이전트를 경유하여 `codex-companion.mjs task`를 호출한다. |
| **`--wait` vs `--background`** | `--wait`은 동기 실행으로, 결과가 나올 때까지 대기한다. `--background`는 비동기 실행으로, 작업을 백그라운드에 띄우고 즉시 돌아온다. 플래그를 지정하지 않으면 review는 크기에 따라 추천하고, rescue는 foreground가 기본이다. |
| **Model Alias** | `--model spark`이라고 입력하면 내부적으로 `gpt-5.3-codex-spark`으로 매핑된다. `gpt-5.4-mini` 같은 정식 모델명은 그대로 전달된다. 모델을 지정하지 않으면 Codex의 기본값 또는 config.toml 설정을 따른다. |
