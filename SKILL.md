---
name: hand-over-to-kiro
description: >
  Delegate tasks to kiro-cli for implementation. Use when user says "/hand-over", "delegate to kiro",
  "let kiro handle this", "let kiro do this", "pass to kiro", "hand over to kiro", or any variant
  requesting kiro-cli execution. Works in two modes: (1) Plan mode — Claude passes an existing plan
  to kiro-cli for implementation, then reports results. (2) Direct mode — Claude passes user's task
  description to kiro-cli. Configures kiro-cli to use the best available Claude model automatically.
---

# Hand Over to Kiro

Delegate implementation tasks to kiro-cli. Claude orchestrates the full handoff: gathers context,
builds a self-contained prompt, invokes kiro-cli, captures output, and reports back.

## First-Time Setup

On first invocation, configure kiro-cli to use the best available model:

```bash
kiro-cli settings chat.defaultModel claude-opus-4.7
```

If that fails (model not available), fall back:

```bash
kiro-cli settings chat.defaultModel claude-opus-4.6
```

Verify: `kiro-cli settings list` — confirm `chat.defaultModel` is set.
Run once per environment. Skip if already configured.

## Trigger Phrases

Activate on any of: `/hand-over`, `delegate to kiro`, `let kiro handle this`, `let kiro do this`,
`pass to kiro`, `hand over to kiro`, or similar phrasing requesting kiro-cli execution.

## Workflow

### Step 1 — Determine Mode

| Mode | When | What to pass |
|------|------|--------------|
| **Plan** | Claude has an active plan (from EnterPlanMode or user-provided) | Full plan with steps |
| **Direct** | No plan exists | User's request with relevant context |

### Step 2 — Gather Context

Before building the prompt, collect:

- **Task scope**: What needs to be done (from plan steps or user request)
- **Relevant files**: Paths kiro should read or modify, with brief reasons
- **Constraints**: Framework, coding style, test requirements, or other rules from conversation
- **Current state**: Working directory, branch name, any in-progress changes

### Step 3 — Build the Prompt

Write a clear, self-contained prompt. Kiro has no memory of this conversation — include everything it needs.

**Plan mode template:**

```
Implement the following plan in the current working directory.

## Plan

{full plan text with numbered steps}

## Key Files

- {path}: {why this file matters}
- {path}: {why this file matters}

## Constraints

- {constraint — e.g., "Use TypeScript strict mode"}
- {constraint — e.g., "All new functions need unit tests"}

## Notes

- Working directory: {pwd}
- Branch: {branch name if relevant}
- {any other context kiro needs}
```

**Direct mode template:**

```
{user's task description — rephrase for clarity if needed}

## Key Files

- {path}: {why this file matters}

## Constraints

- {constraint}

## Context

- Working directory: {pwd}
- {additional context}
```

Keep prompts focused. Don't dump the entire conversation — extract what matters.

### Step 4 — Execute

Run kiro-cli in non-interactive mode with full tool access:

```bash
kiro-cli chat --no-interactive --trust-all-tools "{prompt}"
```

For multi-line prompts, use a heredoc:

```bash
kiro-cli chat --no-interactive --trust-all-tools "$(cat <<'KIRO_PROMPT'
{multi-line prompt here}
KIRO_PROMPT
)"
```

Set timeout to 600000ms (10 minutes) for complex tasks.

### Step 5 — Process and Report

After kiro-cli completes:

1. **Parse output** — Identify files created/modified, errors, warnings, and completion status
2. **Verify changes** — Run `git diff --stat` to confirm what changed on disk
3. **Report to user**:
   - Files changed and what was done
   - Any errors or warnings from kiro's output
   - Plan mode: which steps completed vs. remaining
   - Suggest next steps if work is incomplete

## Error Handling

| Error | Action |
|-------|--------|
| `kiro-cli: command not found` | Tell user to install: `curl -fsSL https://cli.kiro.dev/install \| bash` |
| Authentication failure | Tell user to run `kiro-cli login` |
| Model not available | Try fallback model, verify with `kiro-cli settings list` |
| Timeout (10 min) | Show partial output, ask user how to proceed |
| Non-zero exit code | Show error output, diagnose root cause before retrying |
| MCP server failure | Add `--require-mcp-startup` if MCP tools are needed, or skip if not |

## Tips

- **Large tasks**: Break into smaller handoffs. Kiro works better with focused, well-scoped tasks.
- **Verification**: After kiro finishes, review changes before committing. Run tests if available.
- **Retry with context**: If kiro partially completes, include what it already did in the next prompt so it doesn't redo work.
- **Session resume**: For iterative work, note kiro's session ID from output to resume later with `--resume-id`.

## Kiro CLI Quick Reference

See [references/kiro-cli.md](references/kiro-cli.md) for the full command reference.

Key commands used by this skill:

| Command | Purpose |
|---------|---------|
| `kiro-cli chat --no-interactive --trust-all-tools "prompt"` | Execute a task autonomously |
| `kiro-cli settings chat.defaultModel claude-opus-4.7` | Set the model |
| `kiro-cli settings list` | Verify configuration |
| `kiro-cli chat --resume-id <ID>` | Resume a previous session |
| `kiro-cli doctor` | Diagnose issues |
