# hand-over-to-kiro

A [skills.sh](https://skills.sh) skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that delegates implementation tasks to [kiro-cli](https://kiro.dev).

## Why

Sometimes you want Claude to plan and kiro to build — or you want a second AI agent to handle a chunk of work independently. This skill bridges the two: Claude gathers context, builds a focused prompt, hands it to kiro-cli, and reports back what happened.

## Modes

- **Plan mode** — Claude has an active plan. It passes the full plan to kiro-cli for implementation.
- **Direct mode** — No plan. Claude forwards your task description with relevant file paths and constraints.

## Install

```bash
npx skills add thabti/hand-over-to-kiro
```

## Usage

In Claude Code, use any of these:

```
/hand-over
delegate to kiro
let kiro handle this
hand over to kiro
pass to kiro
```

## Prerequisites

1. **kiro-cli** installed — [install guide](https://kiro.dev/docs/cli/)
   ```bash
   curl -fsSL https://cli.kiro.dev/install | bash
   ```
2. **Authenticated** — run `kiro-cli login` once

## How It Works

1. Detects whether you have an active plan or a direct request
2. Gathers relevant file paths, constraints, and project context
3. Builds a self-contained prompt with structured boundary markers (kiro has no memory of your Claude session)
4. Writes prompt to a temp file and executes via `kiro-cli chat --no-interactive`
5. Parses output, verifies changes on disk, reports results

## Security

- **No shell interpolation** — prompts are written to temp files, not interpolated into shell arguments
- **Tool trust is opt-in** — runs without `--trust-all-tools` by default; only enables when you explicitly request autonomous execution
- **Input rephrasing** — user requests are rephrased into structured task descriptions, not passed verbatim
- **Boundary markers** — XML-style tags separate instructions from user-provided content

## What's Included

```
SKILL.md                  # Skill definition with agent instructions
references/kiro-cli.md    # Full kiro-cli command reference
README.md                 # This file
LICENSE                   # MIT
```

## Credits

Created by [Sabeur Thabti](https://github.com/thabti)

## License

MIT
