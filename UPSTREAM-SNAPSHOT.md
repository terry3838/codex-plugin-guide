# Upstream Snapshot — codex-plugin-cc

- source repo: `https://github.com/openai/codex-plugin-cc.git`
- previous synced commit: `6a5c2ba53b734f3cdd8daacbd49f68f3e6c8c167`
- current synced commit: `6a5c2ba53b734f3cdd8daacbd49f68f3e6c8c167`
- sync mode: `no-change`
- impact labels: 일반 변경
- guide repo: `codex-plugin-guide`

## 원본 한줄 요약

Use Codex from inside Claude Code for code reviews or to delegate tasks to Codex.

## recent upstream commits

- `6a5c2ba fix: quote \$ARGUMENTS in cancel, result, and status commands (#168)`
- `8e9a38c fix: correct invalid 'xhigh' reasoning effort in README (#99)`
- `b115623 fix: declare model in codex-rescue agent frontmatter (#169)`
- `c24afe8 codex: honor --cwd when reporting session runtime (#35)`
- `11a720b bump: update plugin version to 1.0.3 (#180)`
- `bc8fa66 fix: avoid embedding large adversarial review diffs (#179)`
- `d216a5f codex: scope default cancel selection to the current Claude session (#84)`
- `40d213d codex: scope implicit resume-last selection to the current Claude session (#83)`

## top-level structure

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

## changed files

- 변경 파일 없음

## README excerpt

```md
# Codex plugin for Claude Code

Use Codex from inside Claude Code for code reviews or to delegate tasks to Codex.

This plugin is for Claude Code users who want an easy way to start using Codex from the workflow
they already have.

<video src="./docs/plugin-demo.webm" controls muted playsinline autoplay></video>

## What You Get

- `/codex:review` for a normal read-only Codex review
- `/codex:adversarial-review` for a steerable challenge review
- `/codex:rescue`, `/codex:status`, `/codex:result`, and `/codex:cancel` to delegate work and manage background jobs

## Requirements

- **ChatGPT subscription (incl. Free) or OpenAI API key.**
  - Usage will contribute to your Codex usage limits. [Learn more](https://developers.openai.com/codex/pricing).
- **Node.js 18.18 or later**

## Install

Add the marketplace in Claude Code:

```bash
/plugin marketplace add openai/codex-plugin-cc
```

Install the plugin:

```bash
/plugin install codex@openai-codex
```

Reload plugins:

```bash
/reload-plugins
```

Then run:

```bash
/codex:setup
```

`/codex:setup` will tell you whether Codex is ready. If Codex is missing and npm is available, it can offer to install Codex for you.

If you prefer to install Codex yourself, use:

```bash
npm install -g @openai/codex
```

If Codex is installed but not logged in yet, run:

```bash
!codex login
```

After install, you should see:

- the slash commands listed below
- the `codex:codex-rescue` subagent in `/agents`

One simple first run is:

```bash
/codex:review --background
/codex:status
/codex:result
```

## Usage

### `/codex:review`

Runs a normal Codex review on your current work. It gives you the same quality of code review as running `/review` inside Codex directly.

> [!NOTE]
> Code review especially for multi-file changes might take a while. It's generally recommended to run it in the background.

Use it when you want:

- a review of your current uncommitted changes
- a review of your branch compared to a base branch like `main`

Use `--base <ref>` for branch review. It also supports `--wait` and `--background`. It is not steerable and does not take custom focus text. Use [`/codex:adversarial-review`](#codexadversarial-review) when you want to challenge a specific decision or risk area.

Examples:

```bash
/codex:review
/codex:review --base main
/codex:review --background
```

This command is read-only and will not perform any changes. When run in the background you can use [`/codex:status`](#codexstatus) to check on the progress and [`/codex:cancel`](#codexcancel) to cancel the ongoing task.

### `/codex:adversarial-review`

Runs a **steerable** review that questions the chosen implementation and design.

It can be used to pressure-test assumptions, tradeoffs, failure modes, and whether a different approach would have been safer or simpler.

It uses the same review target selection as `/codex:review`, including `--base <ref>` for branch review.
It also supports `--wait` and `--background`. Unlike `/codex:review`, it can take extra focus text after the flags.

Use it when you want:

- a review before shipping that challenges the direction, not just the code details
- review focused on design choices, tradeoffs, hidden assumptions, and alternative approaches
- pressure-testing around specific risk areas like auth, data loss, rollback, race conditions, or reliability

Examples:

```bash
/codex:adversarial-review
/codex:adversarial-review --base main challenge whether this was the right caching and retry design
```
