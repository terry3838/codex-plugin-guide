# Obsidian Vault Map

이 문서는 `codex-plugin-guide` 의 repo guide 문서를 Obsidian note pack과 어떻게 대응시키는지 정리한다.

## 출력 모드

- mode: `hybrid`
- repo guide: `codex-plugin-guide/`
- repo-local note pack: `obsidian/codex-plugin Guide/`
- live vault target: `unset`
- live sync status: `not applied`

## 안전 원칙

- repo-local pack이 정본이다.
- live vault sync는 **의도된 vault target이 명시적으로 확인된 뒤에만** 수행한다.
- default vault나 CLI 반환값은 참고 정보일 뿐, 목적지 확정 근거가 아니다.

## 매핑

| repo guide | note pack |
|---|---|
| `README.md` | `codex-plugin Guide/codex-plugin Guide - MOC.md` |
| `01-learning-paths.md` | `codex-plugin Guide/02 Learning Paths.md` |
| `02-glossary.md` | `codex-plugin Guide/03 Glossary.md` |
| `01-usage-scenarios.md` | `codex-plugin Guide/Deep Dives/Usage scenarios.md` |
| `sections/01-overview.md` | `codex-plugin Guide/01 Overview.md` |
| `categories/code-review.md` | `codex-plugin Guide/Categories/Code review.md` |
| `categories/operations.md` | `codex-plugin Guide/Categories/Operations.md` |
| `categories/overview.md` | `codex-plugin Guide/Categories/Overview.md` |
| `categories/task-delegation.md` | `codex-plugin Guide/Categories/Task delegation.md` |

## note map 의도

- MOC가 frontdoor 역할을 한다.
- Learning Paths / Glossary가 기본 3축이 된다.
- deep dive는 source-backed reading surface로 붙인다.
- References와 Workflows는 길찾기/증거를 붙잡아 둔다.
