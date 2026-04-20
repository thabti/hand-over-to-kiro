# hand-over-to-kiro

A [skills.sh](https://skills.sh) skill for Claude Code that delegates implementation tasks to [kiro-cli](https://kiro.dev).

## What it does

Lets you hand off tasks from Claude Code to kiro-cli for implementation. Works in two modes:

- **Plan mode** — Pass an existing plan to kiro-cli for execution
- **Direct mode** — Pass a task description directly

Claude orchestrates the handoff: prepares context, invokes kiro-cli with `--no-interactive --trust-all-tools`, captures output, and reports results back.

## Install

```bash
npx skills add thabti/hand-over-to-kiro
```

## Usage

In Claude Code, say any of:

- `/hand-over`
- `"delegate to kiro"`
- `"let kiro handle this"`
- `"hand over to kiro"`
- `"pass to kiro"`

## Prerequisites

- [kiro-cli](https://kiro.dev/docs/cli/) installed and authenticated (`kiro-cli login`)
- The skill auto-configures kiro-cli to use `claude-opus-4.7` (falls back to `claude-opus-4.6`)

## How it works

1. Detects whether you have an active plan or a direct request
2. Builds a self-contained prompt with task, file paths, and constraints
3. Executes via `kiro-cli chat --no-interactive --trust-all-tools`
4. Parses output and reports what kiro did — files changed, errors, completion status

## Credits

Created by [Sabeur Thabti](https://github.com/thabti)

## License

MIT
