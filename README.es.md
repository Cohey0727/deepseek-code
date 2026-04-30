# deepseek

[English](./README.md) · [日本語](./README.ja.md) · [简体中文](./README.zh-CN.md) · [한국어](./README.ko.md) · [Español](./README.es.md) · [Français](./README.fr.md)

Instala un comando global `deepseek` que arranca Claude Code usando DeepSeek como backend.

Utiliza el endpoint compatible con Anthropic que DeepSeek ofrece oficialmente, así que no se requiere ningún proxy ni capa de traducción.

> **Referencia**: el diseño de variables de entorno y el método de conexión están basados en la documentación oficial de DeepSeek:
> [Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code).
> Este repositorio es un envoltorio ligero que convierte los pasos manuales de `export`
> de esa guía en un flujo `.env` + `make install` reproducible y disponible globalmente.

## Requisitos

- macOS / Linux con `bash`
- `make`
- [Claude Code](https://docs.anthropic.com/claude-code) (`claude` debe estar en tu PATH)
- Una clave de API de DeepSeek (obtenla en [platform.deepseek.com](https://platform.deepseek.com/))

## Instalación

```bash
make install
```

Esto hace lo siguiente:

1. Copia `.env.example` a `.env` si no existe (con `chmod 600`)
2. Genera `~/.local/bin/deepseek` con la ruta absoluta al `.env` de *este directorio* incrustada
3. Avisa si `~/.local/bin` no está en tu PATH

Después de instalar, abre `.env` y añade tu clave de API.

## Variables de `.env`

| Variable | Obligatoria | Por defecto | Descripción |
|----------|-------------|-------------|-------------|
| `DEEPSEEK_API_KEY` | sí | — | Tu clave de API de DeepSeek (`sk-...`) |
| `ANTHROPIC_MODEL` | no | `deepseek-v4-pro` | Modelo principal |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | no | `deepseek-v4-pro` | Modelo para el slot Opus |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | no | `deepseek-v4-pro` | Modelo para el slot Sonnet |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | no | `deepseek-v4-flash` | Modelo para el slot Haiku |
| `CLAUDE_CODE_SUBAGENT_MODEL` | no | `deepseek-v4-flash` | Modelo usado por los subagentes |
| `CLAUDE_CODE_EFFORT_LEVEL` | no | `max` | Nivel de effort de Claude Code |

El `.env` mínimo viable es solo:

```dotenv
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Uso

Desde cualquier directorio:

```bash
deepseek
```

Esto carga el `.env` de este repositorio y arranca Claude Code con DeepSeek como backend.

Los argumentos se pasan a `claude` tal cual:

```bash
deepseek --help
deepseek -p "Revisa mis definiciones de tipos en TypeScript"
```

## Cómo funciona

El comando `deepseek` es un script de shell ligero. En el momento de la instalación se incrusta la ruta absoluta del `.env` de este directorio. En tiempo de ejecución carga ese `.env` y exporta las siguientes variables compatibles con Anthropic antes de hacer `exec` de `claude`:

- `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
- `ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY`
- Las variables de modelo de la tabla anterior

Si `ANTHROPIC_API_KEY` está definida en el entorno, anularía a `AUTH_TOKEN`, así que el script la elimina con `unset` antes de lanzar `claude`.

## Desinstalación

```bash
make uninstall
```

Esto elimina `~/.local/bin/deepseek`. El archivo `.env` se conserva — bórralo manualmente si ya no lo necesitas.

## Solución de problemas

**`deepseek: command not found`**
`~/.local/bin` no está en tu PATH. Añade esto a tu shell rc (p. ej. `~/.zshrc`):
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**`deepseek: DEEPSEEK_API_KEY is empty`**
El lado derecho de `DEEPSEEK_API_KEY=` en `.env` está vacío. Rellénalo.

**Has movido el proyecto a otra carpeta**
El script tiene la ruta antigua incrustada. Vuelve a ejecutar `make install` desde la nueva ubicación.

**Quieres un prefijo de instalación distinto**
```bash
make install PREFIX=/opt/local
```
Esto instala en `/opt/local/bin/deepseek`.

## Referencias

- [DeepSeek: Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code) — la guía oficial sobre la que se basa este repositorio
- [DeepSeek Platform](https://platform.deepseek.com/) — emisión de claves de API
- [Claude Code](https://docs.anthropic.com/claude-code) — documentación oficial
