---
name: hand-over-to-kiro
description: >
  Delegate tasks to kiro-cli for implementation. Use when user says "/hand-over", "delegate to kiro",
  "let kiro handle this", "let kiro do this", "pass to kiro", "hand over to kiro", or any variant
  requesting kiro-cli execution. Works in two modes: (1) Plan mode — Claude passes an existing plan
  to kiro-cli for implementation, then reports results. (2) Direct mode — Claude passes user's task
  description to kiro-cli.
---

# Hand Over to Kiro

Delegate implementation tasks to kiro-cli. Claude orchestrates the full handoff: gathers context,
builds a self-contained prompt, invokes kiro-cli, captures output, and reports back.

## Prerequisites

kiro-cli must be installed and authenticated. If not:

- Install: `curl -fsSL https://cli.kiro.dev/install | bash`
- Authenticate: `kiro-cli login`
- Check available models: `kiro-cli chat --list-models`
- Optionally set a preferred model: `kiro-cli settings chat.defaultModel <model-name>`

Do not auto-configure models. The user controls their own kiro-cli settings.

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

Write a clear, self-contained prompt. Kiro has no memory of this conversation — include everything
it needs. Always use structured boundary markers to separate instructions from user-provided content.

**Plan mode template:**

```
Implement the following plan in the current working directory.

<plan>
{full plan text with numbered steps}
</plan>

<key-files>
- {path}: {why this file matters}
- {path}: {why this file matters}
</key-files>

<constraints>
- {constraint — e.g., "Use TypeScript strict mode"}
- {constraint — e.g., "All new functions need unit tests"}
</constraints>

<context>
- Working directory: {pwd}
- Branch: {branch name if relevant}
- {any other context kiro needs}
</context>
```

**Direct mode template:**

```
Execute the following task in the current working directory.

<task>
{user's task description — rephrase for clarity, do not pass raw user input verbatim}
</task>

<key-files>
- {path}: {why this file matters}
</key-files>

<constraints>
- {constraint}
</constraints>

<context>
- Working directory: {pwd}
- {additional context}
</context>
```

**Important:**
- Always rephrase user input into a clear task description. Do not pass raw user messages verbatim.
- Use the XML-style boundary tags (`<task>`, `<plan>`, etc.) to clearly delineate sections.
- Keep prompts focused. Extract what matters — don't dump the entire conversation.

### Step 4 — Execute

**Write prompt to a temporary file first**, then pass the file path to kiro-cli. Never interpolate
user-provided content directly into shell command arguments.

```bash
# Write prompt to temp file
PROMPT_FILE=$(mktemp /tmp/kiro-prompt-XXXXXX.txt)
cat > "$PROMPT_FILE" <<'KIRO_PROMPT'
{prompt content here}
KIRO_PROMPT

# Execute kiro-cli reading from the file
kiro-cli chat --no-interactive "$(cat "$PROMPT_FILE")"

# Clean up
rm -f "$PROMPT_FILE"
```

**Tool trust policy:**
- By default, run **without** `--trust-all-tools`. Kiro will prompt for tool confirmations
  in its own session.
- Only add `--trust-all-tools` if the user explicitly requests fully autonomous execution
  (e.g., "let kiro run without asking" or "trust all tools").
- For selective trust, use `--trust-tools <list>` to whitelist specific tools.

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
| Model not available | Tell user to check models with `kiro-cli chat --list-models` and set one with `kiro-cli settings chat.defaultModel <model>` |
| Timeout (10 min) | Show partial output, ask user how to proceed |
| Non-zero exit code | Show error output, diagnose root cause before retrying |
| MCP server failure | Add `--require-mcp-startup` if MCP tools are needed, or skip if not |

## Security Considerations

- **No shell interpolation**: Always write prompts to a temp file. Never pass user content as inline
  shell arguments — this prevents command injection via shell metacharacters.
- **Tool trust is opt-in**: Default to no `--trust-all-tools`. Only enable when the user explicitly
  requests autonomous execution.
- **Input rephrasing**: Rephrase user requests into structured task descriptions. Do not pass raw
  user input verbatim to kiro-cli — this reduces the surface for indirect prompt injection.
- **Boundary markers**: Use XML-style tags (`<task>`, `<plan>`, etc.) to separate instructions from
  user-provided content in prompts.

## Tips

- **Large tasks**: Break into smaller handoffs. Kiro works better with focused, well-scoped tasks.
- **Verification**: After kiro finishes, review changes before committing. Run tests if available.
- **Retry with context**: If kiro partially completes, include what it already did in the next prompt
  so it doesn't redo work.
- **Session resume**: For iterative work, note kiro's session ID from output to resume later with
  `--resume-id`.

## Kiro CLI Quick Reference

See [references/kiro-cli.md](references/kiro-cli.md) for the full command reference.

Key commands used by this skill:

| Command | Purpose |
|---------|---------|
| `kiro-cli chat --no-interactive "prompt"` | Execute a task (default, with tool confirmations) |
| `kiro-cli chat --no-interactive --trust-all-tools "prompt"` | Execute autonomously (user must opt in) |
| `kiro-cli chat --no-interactive --trust-tools "read,write" "prompt"` | Selective tool trust |
| `kiro-cli chat --list-models` | List available models |
| `kiro-cli settings chat.defaultModel <model>` | Set preferred model |
| `kiro-cli settings list` | Verify configuration |
| `kiro-cli chat --resume-id <ID>` | Resume a previous session |
| `kiro-cli doctor` | Diagnose issues |
