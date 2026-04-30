# deepseek

[English](./README.md) · [日本語](./README.ja.md) · [简体中文](./README.zh-CN.md) · [한국어](./README.ko.md) · [Español](./README.es.md) · [Français](./README.fr.md)

Installs a global `deepseek` command that launches Claude Code with DeepSeek as the backend.

It uses DeepSeek's official Anthropic-compatible endpoint, so no proxy or translation layer is required.

> **Reference**: The environment-variable layout and connection method are based on the official DeepSeek docs:
> [Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code).
> This repo is a thin wrapper that turns the manual `export` steps from that guide into a reproducible,
> globally-installable `.env` + `make install` workflow.

## Requirements

- macOS / Linux with `bash`
- `make`
- [Claude Code](https://docs.anthropic.com/claude-code) (`claude` must be on your PATH)
- A DeepSeek API key (issue one at [platform.deepseek.com](https://platform.deepseek.com/))

## Setup

```bash
make install
```

This will:

1. Copy `.env.example` to `.env` if it doesn't exist (with `chmod 600`)
2. Generate `~/.local/bin/deepseek`, with the absolute path to *this directory's* `.env` baked in
3. Warn you if `~/.local/bin` is not on your PATH

After install, open `.env` and fill in your API key.

## `.env` variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DEEPSEEK_API_KEY` | yes | — | Your DeepSeek API key (`sk-...`) |
| `ANTHROPIC_MODEL` | no | `deepseek-v4-pro` | Primary model |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | no | `deepseek-v4-pro` | Model for the Opus slot |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | no | `deepseek-v4-pro` | Model for the Sonnet slot |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | no | `deepseek-v4-flash` | Model for the Haiku slot |
| `CLAUDE_CODE_SUBAGENT_MODEL` | no | `deepseek-v4-flash` | Model used for subagent invocations |
| `CLAUDE_CODE_EFFORT_LEVEL` | no | `max` | Claude Code effort level |

The minimum viable `.env` is just:

```dotenv
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Usage

From any directory:

```bash
deepseek
```

This loads this repo's `.env` and starts Claude Code with DeepSeek as the backend.

Arguments are passed through to `claude` verbatim:

```bash
deepseek --help
deepseek -p "Review my TypeScript type definitions"
```

## How it works

The `deepseek` command is a thin shell script. At install time, the absolute path to this directory's `.env`
is baked in. At runtime, it sources that `.env` and exports the Anthropic-compatible variables before
`exec`-ing `claude`:

- `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
- `ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY`
- The model overrides listed above

If `ANTHROPIC_API_KEY` is set in your environment it would shadow `AUTH_TOKEN`, so the script `unset`s it
before launching `claude`.

## Uninstall

```bash
make uninstall
```

This removes `~/.local/bin/deepseek`. The `.env` file is left in place — delete it manually if you no
longer need it.

## Troubleshooting

**`deepseek: command not found`**
`~/.local/bin` is not on your PATH. Add this to your shell rc (e.g. `~/.zshrc`):
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**`deepseek: DEEPSEEK_API_KEY is empty`**
The right-hand side of `DEEPSEEK_API_KEY=` in `.env` is blank. Fill it in.

**You moved the project to a different directory**
The script has the old absolute path baked in. Re-run `make install` from the new location.

**You want a different install prefix**
```bash
make install PREFIX=/opt/local
```
This installs to `/opt/local/bin/deepseek` instead.

## References

- [DeepSeek: Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code) — the official guide this repo is built on
- [DeepSeek Platform](https://platform.deepseek.com/) — issue API keys
- [Claude Code](https://docs.anthropic.com/claude-code) — upstream docs
