# 운영

이 문서는 Codex 플러그인의 초기 설정, 설정 파일, 훅 시스템, 그리고 작업 관리 명령어를 다룹니다.

---

## `/codex:setup`

Codex CLI가 설치되어 있고 인증이 완료되었는지 확인합니다. Review Gate의 활성화/비활성화도 이 명령어로 수행합니다.

### 문법

```bash
/codex:setup [--enable-review-gate|--disable-review-gate]
```

### 초기 설정 체크리스트

1. `/codex:setup` 실행
2. Codex CLI가 없으면 → 설치 여부를 묻고, npm을 통해 설치 가능
3. Codex CLI가 있지만 미인증이면 → `!codex login` 실행 안내
4. 모두 정상이면 → 설치 상태와 Review Gate 상태 출력

### Codex CLI 수동 설치

```bash
npm install -g @openai/codex
```

### Review Gate 관리

```bash
# Review Gate 활성화
/codex:setup --enable-review-gate

# Review Gate 비활성화
/codex:setup --disable-review-gate
```

Review Gate에 대한 자세한 설명은 [code-review.md](code-review.md)의 Review Gate 섹션을 참고하세요.

---

## 설정 파일 (config.toml)

플러그인은 Codex CLI의 표준 설정 파일을 그대로 사용합니다. 별도의 플러그인 설정 파일은 없습니다.

### 위치

| 위치 | 용도 |
|------|------|
| `~/.codex/config.toml` | 사용자 전역 설정 |
| `.codex/config.toml` | 프로젝트별 설정 (프로젝트 루트에 배치) |

### 설정 우선순위

프로젝트 설정 > 사용자 전역 설정

> 프로젝트 설정(`.codex/config.toml`)은 해당 프로젝트가 [신뢰됨(trusted)](https://developers.openai.com/codex/config-advanced#project-config-files-codexconfigtoml)으로 설정된 경우에만 로드됩니다.

### 주요 설정 옵션

| 키 | 설명 | 예시 값 |
|----|------|---------|
| `model` | 기본 모델 | `"gpt-5.4-mini"` |
| `model_reasoning_effort` | 기본 추론 effort | `"high"`, `"xhigh"` |
| `openai_base_url` | OpenAI API 엔드포인트 (커스텀) | `"https://..."` |

### 설정 예시

```toml
# ~/.codex/config.toml (사용자 전역)
model = "gpt-5.4-mini"
model_reasoning_effort = "high"
```

```toml
# .codex/config.toml (프로젝트별 -- 이 설정이 우선 적용)
model = "gpt-5.4-mini"
model_reasoning_effort = "xhigh"
```

더 많은 설정 옵션은 [Codex 설정 레퍼런스](https://developers.openai.com/codex/config-reference)를 참고하세요.

---

## 훅 시스템

플러그인은 3가지 Claude Code 훅을 사용하여 세션 라이프사이클과 Review Gate를 관리합니다.

### SessionStart 훅

```
node "${CLAUDE_PLUGIN_ROOT}/scripts/session-lifecycle-hook.mjs" SessionStart
```

- Claude Code 세션이 시작될 때 실행
- Codex 런타임 초기화 및 상태 설정
- 타임아웃: 5초

### SessionEnd 훅

```
node "${CLAUDE_PLUGIN_ROOT}/scripts/session-lifecycle-hook.mjs" SessionEnd
```

- Claude Code 세션이 종료될 때 실행
- 리소스 정리 및 상태 저장
- 타임아웃: 5초

### Stop 훅 (Review Gate)

```
node "${CLAUDE_PLUGIN_ROOT}/scripts/stop-review-gate-hook.mjs"
```

- Claude Code가 응답을 완료할 때마다 실행
- Review Gate가 활성화된 경우에만 실제 리뷰 수행
- 이전 턴에서 코드 변경이 있었는지 확인 후 ALLOW/BLOCK 판정
- 타임아웃: 900초 (15분)

### 훅 파일 위치

훅 설정은 `plugins/codex/hooks/hooks.json`에 정의되어 있습니다. 플러그인 설치 시 자동으로 등록되므로 수동 설정은 불필요합니다.

### 커스터마이징

훅 자체를 커스터마이징할 필요는 거의 없습니다. Review Gate의 활성화/비활성화는 `/codex:setup`으로 제어하면 됩니다.

---

## 작업 관리 명령어 상세

### `/codex:status`

실행 중이거나 완료된 Codex 작업의 상태를 Markdown 테이블로 표시합니다.

#### 문법

```bash
/codex:status [job-id] [--wait] [--timeout-ms <ms>] [--all]
```

#### 옵션

| 옵션 | 설명 |
|------|------|
| `job-id` | 특정 작업의 상세 정보 표시 (생략 시 목록 표시) |
| `--wait` | 실행 중인 작업이 완료될 때까지 대기 |
| `--timeout-ms <ms>` | 대기 최대 시간 (밀리초) |
| `--all` | 완료된 작업을 포함하여 모든 작업 표시 |

#### 실전 예시

```bash
# 현재 세션의 작업 목록 (테이블)
/codex:status

# 모든 작업 (완료 포함)
/codex:status --all

# 특정 작업의 상세 상태
/codex:status task-abc123

# 작업 완료까지 최대 60초 대기
/codex:status --wait --timeout-ms 60000
```

#### 출력 내용

job-id 없이 실행하면: job ID, 종류, 상태, 단계, 경과 시간, 요약, 후속 명령어를 포함하는 Markdown 테이블

job-id를 지정하면: 해당 작업의 전체 상세 정보

---

### `/codex:result`

완료된 Codex 작업의 최종 결과를 조회합니다.

#### 문법

```bash
/codex:result [job-id]
```

#### 옵션

| 옵션 | 설명 |
|------|------|
| `job-id` | 특정 작업의 결과 (생략 시 최근 완료 작업) |

#### 출력 내용

- Job ID 및 상태
- 전체 결과: verdict, summary, findings, details, artifacts, next steps
- 파일 경로 및 라인 번호
- 에러 메시지 (있는 경우)
- Codex session ID (있는 경우)
- 후속 명령어 안내

#### Codex에서 작업 이어가기

결과에 포함된 session ID를 사용하여 Codex에서 직접 작업을 이어갈 수 있습니다:

```bash
codex resume <session-id>
```

---

### `/codex:cancel`

실행 중인 백그라운드 Codex 작업을 취소합니다.

#### 문법

```bash
/codex:cancel [job-id]
```

#### 옵션

| 옵션 | 설명 |
|------|------|
| `job-id` | 취소할 작업 (생략 시 현재 실행 중인 작업) |

#### 실전 예시

```bash
# 현재 실행 중인 작업 취소
/codex:cancel

# 특정 작업 취소
/codex:cancel task-abc123
```
