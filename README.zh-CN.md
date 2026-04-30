# deepseek

[English](./README.md) · [日本語](./README.ja.md) · [简体中文](./README.zh-CN.md) · [한국어](./README.ko.md) · [Español](./README.es.md) · [Français](./README.fr.md)

安装一个全局 `deepseek` 命令，使用 DeepSeek 作为后端来启动 Claude Code。

它通过 DeepSeek 官方提供的 Anthropic 兼容接入端点工作，因此无需任何代理或转换层。

> **参考**: 环境变量设计和接入方式基于 DeepSeek 官方文档：
> [Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code)。
> 本仓库是一个轻量包装，将该指南中的手动 `export` 步骤
> 转化为可复现、可全局使用的 `.env` + `make install` 工作流。

## 环境要求

- macOS / Linux 且具备 `bash`
- `make`
- [Claude Code](https://docs.anthropic.com/claude-code)（`claude` 命令需在 PATH 中）
- DeepSeek API 密钥（在 [platform.deepseek.com](https://platform.deepseek.com/) 申请）

## 安装

```bash
make install
```

执行流程：

1. 若 `.env` 不存在则从 `.env.example` 复制一份（权限 `600`）
2. 生成 `~/.local/bin/deepseek`，其中烘焙了**本目录**下 `.env` 的绝对路径
3. 若 `~/.local/bin` 不在 PATH 中将给出警告

安装完成后，请打开 `.env` 填写 API 密钥。

## `.env` 变量

| 变量 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `DEEPSEEK_API_KEY` | 是 | — | DeepSeek API 密钥（`sk-...`） |
| `ANTHROPIC_MODEL` | 否 | `deepseek-v4-pro` | 主模型 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 否 | `deepseek-v4-pro` | Opus 槽位使用的模型 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 否 | `deepseek-v4-pro` | Sonnet 槽位使用的模型 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 否 | `deepseek-v4-flash` | Haiku 槽位使用的模型 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 否 | `deepseek-v4-flash` | 子代理调用使用的模型 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 否 | `max` | Claude Code effort 等级 |

最简 `.env` 仅需：

```dotenv
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## 使用

在任意目录下：

```bash
deepseek
```

将自动加载本仓库的 `.env`，并以 DeepSeek 为后端启动 Claude Code。

传给 `claude` 的参数会被原样透传：

```bash
deepseek --help
deepseek -p "审查我的 TypeScript 类型定义"
```

## 工作原理

`deepseek` 命令是一个轻量 shell 脚本。在安装时，本目录下 `.env` 的绝对路径被烘焙进去；运行时它读取该 `.env`，导出以下 Anthropic 兼容环境变量后再 `exec` 调用 `claude`：

- `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
- `ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY`
- 上表中的各项模型设置

如果环境中残留 `ANTHROPIC_API_KEY`，会覆盖 `AUTH_TOKEN`，因此脚本在启动前会 `unset` 它。

## 卸载

```bash
make uninstall
```

将删除 `~/.local/bin/deepseek`。`.env` 文件会保留——如不再需要请手动删除。

## 故障排查

**`deepseek: command not found`**
`~/.local/bin` 不在 PATH 中。请在 `~/.zshrc` 等 shell 配置文件中追加：
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**`deepseek: DEEPSEEK_API_KEY is empty`**
`.env` 中 `DEEPSEEK_API_KEY=` 等号右侧为空。请填入密钥。

**项目目录被移动**
脚本中已烘焙旧路径。请在新位置重新执行 `make install`。

**想要更改安装前缀**
```bash
make install PREFIX=/opt/local
```
将安装到 `/opt/local/bin/deepseek`。

## 参考链接

- [DeepSeek 官方：Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code) — 本仓库所依据的官方指南
- [DeepSeek Platform](https://platform.deepseek.com/) — 申请 API 密钥
- [Claude Code](https://docs.anthropic.com/claude-code) — 上游文档
