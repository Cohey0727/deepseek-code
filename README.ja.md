# deepseek

[English](./README.md) · [日本語](./README.ja.md) · [简体中文](./README.zh-CN.md) · [한국어](./README.ko.md) · [Español](./README.es.md) · [Français](./README.fr.md)

Claude Code を DeepSeek バックエンドで起動するためのグローバルコマンド `deepseek` をインストールします。

DeepSeek が公式に提供する Anthropic 互換エンドポイントを経由するため、プロキシや変換層は不要です。

> **参考**: 環境変数の設計および接続方式は DeepSeek 公式ドキュメント
> [Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code)
> を元にしています。本リポジトリは、そこで案内されている手動 `export` 手順を
> `.env` + `make install` で再現可能・グローバル利用可能にする薄いラッパーです。

## 必要なもの

- macOS / Linux と `bash`
- `make`
- [Claude Code](https://docs.anthropic.com/claude-code) (`claude` コマンドが PATH 上にあること)
- DeepSeek の API キー（[platform.deepseek.com](https://platform.deepseek.com/) で発行）

## セットアップ

```bash
make install
```

これで以下が起こります:

1. `.env` が無ければ `.env.example` からコピーされる（パーミッション 600）
2. `~/.local/bin/deepseek` が生成される（このディレクトリの `.env` への絶対パスを焼き込む）
3. `~/.local/bin` が PATH に無ければ警告が出る

セットアップ後、`.env` を開いて API キーを記入してください。

## `.env` の中身

| 変数 | 必須 | デフォルト | 説明 |
|------|------|------------|------|
| `DEEPSEEK_API_KEY` | はい | （無し） | DeepSeek の API キー（`sk-...`） |
| `ANTHROPIC_MODEL` | いいえ | `deepseek-v4-pro` | メインのモデル |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | いいえ | `deepseek-v4-pro` | Opus スロットに割り当てるモデル |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | いいえ | `deepseek-v4-pro` | Sonnet スロットに割り当てるモデル |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | いいえ | `deepseek-v4-flash` | Haiku スロットに割り当てるモデル |
| `CLAUDE_CODE_SUBAGENT_MODEL` | いいえ | `deepseek-v4-flash` | サブエージェント実行時のモデル |
| `CLAUDE_CODE_EFFORT_LEVEL` | いいえ | `max` | Claude Code の effort レベル |

最低限 `DEEPSEEK_API_KEY` だけ書けば動きます:

```dotenv
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## 使い方

任意のディレクトリで:

```bash
deepseek
```

これだけで、このリポジトリの `.env` が読み込まれ、DeepSeek を裏に持った Claude Code が起動します。

`claude` 本体への引数はそのまま透過されます:

```bash
deepseek --help
deepseek -p "TypeScript の型定義を見直して"
```

## 仕組み

`deepseek` コマンドは生成時にこのディレクトリの `.env` への絶対パスが焼き込まれた薄いシェルスクリプトです。実行時に `.env` を読み込み、以下の Anthropic 互換環境変数を設定してから `claude` を `exec` します:

- `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
- `ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY`
- 上表の各モデル指定

環境に残っている `ANTHROPIC_API_KEY` は `AUTH_TOKEN` を上書きしてしまうので、起動前に `unset` されます。

## アンインストール

```bash
make uninstall
```

`~/.local/bin/deepseek` が削除されます。`.env` は残ります（手動で削除してください）。

## トラブルシューティング

**`deepseek: command not found`**
`~/.local/bin` が PATH に通っていません。`~/.zshrc` 等に追記してください:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**`deepseek: DEEPSEEK_API_KEY is empty`**
`.env` の `DEEPSEEK_API_KEY=` の右辺が空です。キーを記入してください。

**プロジェクトを別の場所に移動した**
スクリプトには移動前の絶対パスが焼き込まれています。移動先で `make install` を再実行してください。

**インストールパスを変えたい**
```bash
make install PREFIX=/opt/local
```
で `/opt/local/bin/deepseek` にインストールされます。

## 参考リンク

- [DeepSeek 公式: Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code) — 本リポジトリのベースとなった公式手順
- [DeepSeek Platform](https://platform.deepseek.com/) — API キー発行
- [Claude Code](https://docs.anthropic.com/claude-code) — 本体ドキュメント
