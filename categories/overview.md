# codex-plugin-cc 개요

## 원본 저장소 역할

- repo: `codex-plugin-cc`
- source: `https://github.com/openai/codex-plugin-cc.git`
- latest synced commit: `6a5c2ba53b73`
- summary: Use Codex from inside Claude Code for code reviews or to delegate tasks to Codex.

## 이번 싸이클 판단

- sync mode: `update`
- impact labels: README/소개, 스킬/플러그인, 테스트/검증
- 판단: origin 변경 파일을 기준으로 guide 문서의 관련 섹션을 다시 읽고 반영했습니다. 핵심 영향 영역: README/소개, 스킬/플러그인, 테스트/검증.

## 최근 upstream 커밋

- `6a5c2ba fix: quote \$ARGUMENTS in cancel, result, and status commands (#168)`
- `8e9a38c fix: correct invalid 'xhigh' reasoning effort in README (#99)`
- `b115623 fix: declare model in codex-rescue agent frontmatter (#169)`
- `c24afe8 codex: honor --cwd when reporting session runtime (#35)`
- `11a720b bump: update plugin version to 1.0.3 (#180)`
- `bc8fa66 fix: avoid embedding large adversarial review diffs (#179)`

## 확인한 원본 구조

- `.claude-plugin/`
- `LICENSE`
- `NOTICE`
- `package-lock.json`
- `package.json`
- `plugins/`
- `README.md`
- `scripts/`
- `tests/`
- `tsconfig.app-server.json`

## guide 업데이트 포인트

- README 관리 블록 갱신
- `UPSTREAM-SNAPSHOT.md` 갱신
- `SYNC-LOG.md` 갱신
- 개요 문서 재작성

## 변경 파일 샘플

- `.claude-plugin/marketplace.json`
- `README.md`
- `package-lock.json`
- `package.json`
- `plugins/codex/.claude-plugin/plugin.json`
- `plugins/codex/agents/codex-rescue.md`
- `plugins/codex/commands/cancel.md`
- `plugins/codex/commands/result.md`
- `plugins/codex/commands/status.md`
- `plugins/codex/prompts/adversarial-review.md`
- `plugins/codex/scripts/codex-companion.mjs`
- `plugins/codex/scripts/lib/app-server-protocol.d.ts`
- `plugins/codex/scripts/lib/app-server.mjs`
- `plugins/codex/scripts/lib/codex.mjs`
- `plugins/codex/scripts/lib/git.mjs`
- `plugins/codex/scripts/lib/job-control.mjs`
- `plugins/codex/scripts/lib/process.mjs`
- `plugins/codex/scripts/stop-review-gate-hook.mjs`
- `scripts/bump-version.mjs`
- `tests/bump-version.test.mjs`
