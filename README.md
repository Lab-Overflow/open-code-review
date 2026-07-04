<div align="center">
  <img src="imgs/logo-core.svg" alt="OpenCodeReview logo" width="160" />
  <h1>OpenCodeReview</h1>
  <p>A personal, open source AI code review CLI.</p>
</div>

<p align="center">
  <a href="https://www.npmjs.com/package/@lab-overflow/open-code-review"><img alt="npm" src="https://img.shields.io/npm/v/@lab-overflow/open-code-review?style=flat-square" /></a>
  <a href="https://github.com/Lab-Overflow/open-code-review/actions/workflows/release.yml"><img alt="Build status" src="https://img.shields.io/github/actions/workflow/status/Lab-Overflow/open-code-review/release.yml?style=flat-square" /></a>
  <a href="https://goreportcard.com/report/github.com/Lab-Overflow/open-code-review"><img alt="Go Report Card" src="https://goreportcard.com/badge/github.com/Lab-Overflow/open-code-review?style=flat-square" /></a>
  <a href="https://github.com/Lab-Overflow/open-code-review/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/github/license/Lab-Overflow/open-code-review?style=flat-square" /></a>
</p>

<p align="center">
  English | <a href="README.zh-CN.md">简体中文</a> | <a href="README.ja-JP.md">日本語</a> | <a href="README.ko-KR.md">한국어</a> | <a href="README.ru-RU.md">Русский</a>
</p>

---

## About

OpenCodeReview is my personal open source project for running AI-assisted code reviews from the command line.

The goal is simple: point the tool at a Git diff, let it gather the surrounding code context, and get structured review comments that are easier to act on than a loose chat transcript.

It can review:

- local workspace changes
- a branch range such as `main..feature`
- a single commit
- whole files or directories with `ocr scan`

The project is intentionally provider-agnostic. You can connect Anthropic, OpenAI-compatible APIs, built-in provider presets, or your own private gateway.

## Why I Built This

General coding agents are useful, but code review has a few sharp edges:

- they may miss files in larger diffs
- they can report comments on the wrong line
- they often mix useful feedback with broad style opinions
- repeated reviews can vary more than I want

OpenCodeReview tries to make the boring parts deterministic: file selection, rule matching, diff parsing, batching, and output formatting. The LLM is then used where it is strongest: reading context, reasoning about behavior, and explaining concrete issues.

This is not meant to replace human review. It is a fast first pass that helps catch obvious mistakes, risky changes, and context-sensitive issues before a human spends time on the PR.

## Features

- Diff-based review for staged, unstaged, and untracked files
- Branch and commit review with `--from`, `--to`, and `--commit`
- Full-file scanning with `ocr scan`
- Configurable LLM providers and custom endpoints
- Per-project and global review rules
- JSON output for CI integrations
- Local session viewer for past review runs
- Optional integrations for Claude Code, Codex, Cursor, GitHub Actions, GitLab CI, and GitFlic CI

## Install

### NPM

```bash
npm install -g @lab-overflow/open-code-review
```

After installation, the `ocr` command should be available globally.

### GitHub Release

macOS and Linux users can install the latest release with:

```bash
curl -fsSL https://raw.githubusercontent.com/Lab-Overflow/open-code-review/main/install.sh | sh
```

You can choose the install directory or pin a version:

```bash
OCR_INSTALL_DIR="$HOME/.local/bin" OCR_VERSION=v1.3.13 \
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/Lab-Overflow/open-code-review/main/install.sh)"
```

### From Source

```bash
git clone https://github.com/Lab-Overflow/open-code-review.git
cd open-code-review
make build
sudo cp dist/opencodereview /usr/local/bin/ocr
```

## Quick Start

### 1. Configure an LLM

Interactive setup:

```bash
ocr config provider
ocr config model
```

The config is stored at:

```text
~/.opencodereview/config.json
```

You can also configure a provider from the CLI:

```bash
ocr config set provider anthropic
ocr config set providers.anthropic.api_key your-api-key-here
ocr config set providers.anthropic.model claude-sonnet-4-6
```

Custom OpenAI-compatible endpoint:

```bash
ocr config set provider my-gateway
ocr config set custom_providers.my-gateway.url https://my-llm-gateway.example/v1
ocr config set custom_providers.my-gateway.protocol openai
ocr config set custom_providers.my-gateway.api_key your-api-key-here
ocr config set custom_providers.my-gateway.model your-model
```

Environment variables can also override the config file:

```bash
export OCR_LLM_URL=https://api.anthropic.com/v1/messages
export OCR_LLM_TOKEN=your-api-key-here
export OCR_LLM_MODEL=claude-sonnet-4-6
export OCR_USE_ANTHROPIC=true
```

### 2. Test the Connection

```bash
ocr llm test
```

### 3. Run a Review

```bash
cd your-project

# Review current workspace changes
ocr review

# Review a branch range
ocr review --from main --to feature-branch

# Review one commit
ocr review --commit abc123

# Scan whole files instead of a diff
ocr scan
ocr scan --path internal/agent
```

## Common Commands

| Command | Description |
|---------|-------------|
| `ocr review` | Review Git changes |
| `ocr scan` | Review whole files or directories |
| `ocr review --preview` | Show which files would be reviewed without calling the LLM |
| `ocr scan --preview` | Show which files would be scanned |
| `ocr config provider` | Interactive provider setup |
| `ocr config model` | Interactive model selection |
| `ocr config set <key> <value>` | Set config values |
| `ocr llm test` | Test LLM connectivity |
| `ocr llm providers` | List built-in provider presets |
| `ocr rules check <file>` | Preview which review rule applies to a file |
| `ocr viewer` | Open the local review session viewer |
| `ocr version` | Print version information |

## Review Rules

OpenCodeReview supports custom review rules at three levels:

| Priority | Source | Path |
|----------|--------|------|
| 1 | CLI flag | `ocr review --rule ./rule.json` |
| 2 | Project config | `<repo>/.opencodereview/rule.json` |
| 3 | Global config | `~/.opencodereview/rule.json` |

Example:

```json
{
  "rules": [
    {
      "path": "**/*.go",
      "rule": "Check error handling, goroutine leaks, context cancellation, and data races."
    },
    {
      "path": "**/*mapper*.xml",
      "rule": "Check SQL placeholders, injection risks, and missing closing tags."
    }
  ],
  "exclude": ["**/generated/**", "vendor/**"]
}
```

The `rule` field can be inline text or a path to a `.md`, `.txt`, or `.markdown` file.

## CI Usage

For CI, use JSON output:

```bash
ocr review \
  --from origin/main \
  --to "$COMMIT_SHA" \
  --format json
```

Examples are available in:

- [`examples/github_actions/`](examples/github_actions/)
- [`examples/gitlab_ci/`](examples/gitlab_ci/)
- [`examples/gitflic_ci/`](examples/gitflic_ci/)

## Agent Integrations

This repository includes optional integrations for coding agents.

Install as a skill:

```bash
npx skills add Lab-Overflow/open-code-review --skill open-code-review
```

Install as a Codex plugin:

```bash
codex plugin marketplace add Lab-Overflow/open-code-review
```

Install as a Cursor plugin:

```bash
cursor-plugin marketplace add Lab-Overflow/open-code-review
```

For Claude Code, you can install the command file from:

```text
plugins/open-code-review/commands/review.md
```

All integrations still require the `ocr` CLI to be installed and configured.

## Local Viewer

Past review sessions can be inspected in a local browser UI:

```bash
ocr viewer
```

The viewer binds locally by default and includes Host-header checks for safer local use. If you bind it to a non-localhost address, configure:

```bash
OCR_VIEWER_ALLOWED_HOSTS=review.internal,ocr.lan ocr viewer --addr :3000
```

## Project Status

This is a personal project maintained by Lab-Overflow. I use it as a practical tool and evolve it based on real review workflows.

The public API and package names may still change while the project settles. For production use, pin versions and test upgrades in your own CI flow.

## Roadmap

- Improve review comment quality and de-duplication
- Keep provider setup simple for local and CI use
- Improve the local session viewer
- Add more focused examples for common CI platforms
- Keep agent integrations lightweight and easy to inspect

## Contributing

Issues, pull requests, and practical bug reports are welcome.

Before opening a large PR, please start with an issue so the direction can be discussed first. For setup notes and development workflow, see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[Apache-2.0](LICENSE) - Copyright 2026 Lab-Overflow
