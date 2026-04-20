# Kiro CLI Reference

Source: https://kiro.dev/docs/cli/reference/cli-commands/

## Global Flags

- `--verbose` / `-v`: Increase logging (repeatable: `-v`, `-vv`, `-vvv`)
- `--agent <name>`: Use specific agent config
- `--help` / `-h`: Show help
- `--version` / `-V`: Show version

## chat

Start chat session or execute one-shot task.

```bash
kiro-cli chat "prompt"                          # interactive
kiro-cli chat "prompt" --no-interactive         # stdout only
kiro-cli chat --no-interactive --trust-all-tools "prompt"  # full auto
```

**Flags:**
- `--no-interactive`: Print response to stdout, no TUI
- `--trust-all-tools`: Allow all tool usage without confirmation
- `--trust-tools <list>`: Trust only specified tools (comma-separated)
- `--resume` / `-r`: Resume previous conversation
- `--resume-id <ID>`: Resume specific session
- `--list-sessions`: Show saved sessions
- `--list-models`: Show available models
- `--agent <name>`: Use specific agent
- `--wrap <mode>`: Wrapping (always/never/auto)

## settings

Manage configuration.

```bash
kiro-cli settings list                    # show configured settings
kiro-cli settings list --all              # show all available settings
kiro-cli settings chat.defaultModel claude-opus-4.7  # set model
kiro-cli settings --delete chat.defaultModel         # remove setting
kiro-cli settings list --format json-pretty          # JSON output
```

## agent

Manage agent configurations.

```bash
kiro-cli agent create <name>
kiro-cli agent edit [name]
kiro-cli agent list
kiro-cli agent set-default <name>
```

## Auth & System

```bash
kiro-cli login        # authenticate
kiro-cli logout       # sign out
kiro-cli whoami       # current user
kiro-cli doctor       # diagnose issues
kiro-cli update       # update CLI
kiro-cli diagnostic   # system info report
```

## Environment Variables

- `KIRO_LOG_LEVEL`: error/warn/info/debug/trace
- `KIRO_LOG_NO_COLOR`: disable colors (1/true/yes)
