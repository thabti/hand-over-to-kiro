---
name: hand-over
description: >
  Delegate tasks to kiro-cli for implementation. Use when user says "/hand-over", "delegate to kiro",
  "let kiro handle this", "let kiro do this", "pass to kiro", "hand over to kiro", or any variant
  requesting kiro-cli execution. Works in two modes: (1) Plan mode — Claude passes an existing plan
  to kiro-cli for implementation, then reports results. (2) Direct mode — Claude passes user's task
  description to kiro-cli. Ensures kiro-cli uses claude-opus-4.7 model via settings configuration.
---

# Hand Over to Kiro

Delegate implementation tasks to kiro-cli. Claude orchestrates: prepares context, invokes kiro-cli,
captures output, reports back.

## First-Time Setup

On first invocation, configure kiro-cli model:

```bash
kiro-cli settings chat.defaultModel claude-opus-4.7
```

If that fails (model not available), fall back:

```bash
kiro-cli settings chat.defaultModel claude-opus-4.6
```

Verify with `kiro-cli settings list` — confirm `chat.defaultModel` is set.
Only run once per environment. Skip if already configured.

## Workflow

### 1. Determine Mode

- **Plan mode**: Claude has an active plan (from EnterPlanMode or user-provided plan). Pass plan to kiro for implementation.
- **Direct mode**: No plan. Pass user's request text directly.

### 2. Build the Prompt

Construct a clear, self-contained prompt for kiro-cli. Include:

- **Task description**: What needs to be done
- **File paths**: Relevant files kiro should read/modify
- **Plan steps** (plan mode): Full plan text with steps
- **Constraints**: Any requirements from conversation context (framework, style, tests)
- **Working directory context**: kiro runs in same directory, state this explicitly

For plan mode, format:
```
Implement the following plan in this directory:

<plan>
{full plan text}
</plan>

Key files:
- {file1}: {why relevant}
- {file2}: {why relevant}

Constraints:
- {constraint1}
- {constraint2}
```

For direct mode, pass user request with relevant context:
```
{user's task description}

Key files:
- {file1}: {why relevant}

Working directory: {pwd}
```

### 3. Execute

Run kiro-cli with `--no-interactive` and `--trust-all-tools`:

```bash
kiro-cli chat --no-interactive --trust-all-tools "{prompt}"
```

For long prompts, use heredoc:

```bash
kiro-cli chat --no-interactive --trust-all-tools "$(cat <<'KIRO_PROMPT'
{multi-line prompt here}
KIRO_PROMPT
)"
```

Timeout: allow up to 10 minutes (600000ms) for complex tasks.

### 4. Process Results

- Capture stdout from kiro-cli
- Parse for: files created/modified, errors, completion status
- If kiro reports errors or partial completion, diagnose and decide whether to retry or escalate to user

### 5. Report Back

Tell user:
- What kiro did (files changed, features implemented)
- Any issues or warnings from kiro output
- If plan mode: which plan steps completed vs remaining

## Error Handling

| Error | Action |
|-------|--------|
| `kiro-cli` not found | Tell user to install: see https://kiro.dev/docs/cli/ |
| Auth failure | Tell user to run `kiro-cli login` |
| Model not available | Verify settings, suggest `kiro-cli settings chat.defaultModel claude-opus-4.7` |
| Task timeout | Show partial output, ask user how to proceed |
| Non-zero exit | Show error output, diagnose |

## Kiro CLI Quick Reference

See [references/kiro-cli.md](references/kiro-cli.md) for full command reference.

Key commands used by this skill:
- `kiro-cli chat "prompt" --no-interactive --trust-all-tools` — execute task
- `kiro-cli settings chat.defaultModel claude-opus-4.7` — set model
- `kiro-cli settings list` — verify settings
