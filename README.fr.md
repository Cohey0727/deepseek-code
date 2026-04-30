# deepseek

[English](./README.md) · [日本語](./README.ja.md) · [简体中文](./README.zh-CN.md) · [한국어](./README.ko.md) · [Español](./README.es.md) · [Français](./README.fr.md)

Installe une commande globale `deepseek` qui lance Claude Code avec DeepSeek comme backend.

Elle utilise le point d'accès officiel compatible Anthropic fourni par DeepSeek, donc aucun proxy ni couche de traduction n'est nécessaire.

> **Référence** : la conception des variables d'environnement et la méthode de connexion s'appuient sur la documentation officielle de DeepSeek :
> [Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code).
> Ce dépôt est un mince wrapper qui transforme les étapes manuelles d'`export`
> de ce guide en un workflow `.env` + `make install` reproductible et installable globalement.

## Prérequis

- macOS / Linux avec `bash`
- `make`
- [Claude Code](https://docs.anthropic.com/claude-code) (`claude` doit être dans votre PATH)
- Une clé d'API DeepSeek (générée sur [platform.deepseek.com](https://platform.deepseek.com/))

## Installation

```bash
make install
```

Cela exécute :

1. Copie de `.env.example` vers `.env` s'il n'existe pas (avec `chmod 600`)
2. Génération de `~/.local/bin/deepseek`, avec le chemin absolu vers le `.env` de *ce dossier* gravé dedans
3. Avertissement si `~/.local/bin` n'est pas dans votre PATH

Après installation, ouvrez `.env` et renseignez votre clé d'API.

## Variables `.env`

| Variable | Requise | Défaut | Description |
|----------|---------|--------|-------------|
| `DEEPSEEK_API_KEY` | oui | — | Votre clé d'API DeepSeek (`sk-...`) |
| `ANTHROPIC_MODEL` | non | `deepseek-v4-pro` | Modèle principal |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | non | `deepseek-v4-pro` | Modèle pour le slot Opus |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | non | `deepseek-v4-pro` | Modèle pour le slot Sonnet |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | non | `deepseek-v4-flash` | Modèle pour le slot Haiku |
| `CLAUDE_CODE_SUBAGENT_MODEL` | non | `deepseek-v4-flash` | Modèle utilisé pour les sous-agents |
| `CLAUDE_CODE_EFFORT_LEVEL` | non | `max` | Niveau d'effort de Claude Code |

Le `.env` minimum viable se résume à :

```dotenv
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Utilisation

Depuis n'importe quel dossier :

```bash
deepseek
```

Cela charge le `.env` de ce dépôt et démarre Claude Code avec DeepSeek comme backend.

Les arguments passent tels quels à `claude` :

```bash
deepseek --help
deepseek -p "Revois mes définitions de types TypeScript"
```

## Fonctionnement

La commande `deepseek` est un mince script shell. À l'installation, le chemin absolu vers le `.env` de ce dossier est gravé dedans. À l'exécution, il charge ce `.env` et exporte les variables compatibles Anthropic suivantes avant d'`exec` `claude` :

- `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
- `ANTHROPIC_AUTH_TOKEN=$DEEPSEEK_API_KEY`
- Les variables de modèle listées ci-dessus

Si `ANTHROPIC_API_KEY` est défini dans votre environnement, il masquerait `AUTH_TOKEN`, donc le script l'`unset` avant de lancer `claude`.

## Désinstallation

```bash
make uninstall
```

Cela supprime `~/.local/bin/deepseek`. Le fichier `.env` est conservé — supprimez-le manuellement si vous n'en avez plus besoin.

## Dépannage

**`deepseek: command not found`**
`~/.local/bin` n'est pas dans votre PATH. Ajoutez ceci à votre shell rc (par ex. `~/.zshrc`) :
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**`deepseek: DEEPSEEK_API_KEY is empty`**
Le côté droit de `DEEPSEEK_API_KEY=` dans `.env` est vide. Renseignez-le.

**Vous avez déplacé le projet ailleurs**
Le script contient l'ancien chemin gravé. Relancez `make install` depuis le nouvel emplacement.

**Vous voulez un autre préfixe d'installation**
```bash
make install PREFIX=/opt/local
```
Cela installe dans `/opt/local/bin/deepseek`.

## Références

- [DeepSeek : Claude Code Integration Guide](https://api-docs.deepseek.com/guides/agent_integrations/claude_code) — le guide officiel sur lequel ce dépôt s'appuie
- [DeepSeek Platform](https://platform.deepseek.com/) — émission de clés d'API
- [Claude Code](https://docs.anthropic.com/claude-code) — documentation amont
