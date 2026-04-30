# deepseek

[English](./README.md) · [日本語](./README.ja.md) · [简体中文](./README.zh-CN.md) · [한국어](./README.ko.md) · [Español](./README.es.md) · [Français](./README.fr.md)

DeepSeek를 백엔드로 사용하여 Claude Code를 실행하는 글로벌 `deepseek` 명령어를 설치합니다.

DeepSeek가 공식적으로 제공하는 Anthropic 호환 엔드포인트를 사용하므로 별도의 프록시나 변환 계층이 필요하지 않습니다.

> **참고**: 환경 변수 구성과 연결 방식은 DeepSeek 공식 문서
> [Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code)
> 를 기반으로 합니다. 이 저장소는 해당 가이드의 수동 `export` 절차를
> `.env` + `make install` 형태로 재현 가능하고 글로벌 사용이 가능하도록 만든 얇은 래퍼입니다.

## 요구 사항

- macOS / Linux 와 `bash`
- `make`
- [Claude Code](https://docs.anthropic.com/claude-code) (`claude` 명령어가 PATH에 있어야 함)
- DeepSeek API 키 ([platform.deepseek.com](https://platform.deepseek.com/) 에서 발급)

## 설치

```bash
make install
```

이 명령은 다음을 수행합니다:

1. `.env`가 없으면 `.env.example`에서 복사 (권한 600)
2. `~/.local/bin/deepseek` 생성 (이 디렉터리의 `.env` 절대 경로가 내장됨)
3. `~/.local/bin`이 PATH에 없으면 경고 표시

설치 후 `.env`를 열어 API 키를 입력하세요.

## `.env` 변수

| 변수 | 필수 | 기본값 | 설명 |
|------|------|--------|------|
| `DEEPSEEK_API_KEY` | 예 | — | DeepSeek API 키 (`sk-...`) |
| `ANTHROPIC_MODEL` | 아니오 | `deepseek-v4-pro` | 메인 모델 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 아니오 | `deepseek-v4-pro` | Opus 슬롯 모델 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 아니오 | `deepseek-v4-pro` | Sonnet 슬롯 모델 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 아니오 | `deepseek-v4-flash` | Haiku 슬롯 모델 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 아니오 | `deepseek-v4-flash` | 서브에이전트 실행 모델 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 아니오 | `max` | Claude Code effort 레벨 |

최소한의 `.env`는 다음 한 줄이면 충분합니다:

```dotenv
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## 사용법

어떤 디렉터리에서든:

```bash
deepseek
```

이 명령으로 본 저장소의 `.env`가 로드되고, DeepSeek를 백엔드로 사용하는 Claude Code가 실행됩니다.

`claude`에 대한 인수는 그대로 전달됩니다:

```bash
deepseek --help
deepseek -p "TypeScript 타입 정의를 검토해줘"
```

## 작동 방식

`deepseek` 명령어는 얇은 셸 스크립트입니다. 설치 시점에 이 디렉터리의 `.env` 절대 경로가 내장되며, 실행 시 해당 `.env`를 읽고 다음과 같은 Anthropic 호환 환경 변수를 설정한 뒤 `claude`를 `exec`합니다:

- `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
- `ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY`
- 위 표의 모델 지정값들

환경에 남아있는 `ANTHROPIC_API_KEY`는 `AUTH_TOKEN`을 덮어쓸 수 있으므로 시작 전에 `unset` 됩니다.

## 제거

```bash
make uninstall
```

`~/.local/bin/deepseek`만 제거됩니다. `.env`는 그대로 남으니 더 이상 필요 없으면 직접 삭제하세요.

## 문제 해결

**`deepseek: command not found`**
`~/.local/bin`이 PATH에 등록되지 않았습니다. `~/.zshrc` 등에 다음을 추가하세요:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**`deepseek: DEEPSEEK_API_KEY is empty`**
`.env` 안의 `DEEPSEEK_API_KEY=` 우측 값이 비어 있습니다. 키를 입력하세요.

**프로젝트를 다른 위치로 이동했음**
스크립트에 이전 경로가 내장되어 있습니다. 이동한 위치에서 `make install`을 다시 실행하세요.

**설치 경로를 바꾸고 싶음**
```bash
make install PREFIX=/opt/local
```
하면 `/opt/local/bin/deepseek`에 설치됩니다.

## 참고 링크

- [DeepSeek 공식: Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code) — 본 저장소가 기반으로 한 공식 가이드
- [DeepSeek Platform](https://platform.deepseek.com/) — API 키 발급
- [Claude Code](https://docs.anthropic.com/claude-code) — 본체 문서
