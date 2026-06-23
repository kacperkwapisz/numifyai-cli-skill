# Numify AI CLI Skill

An AI agent skill file that teaches coding agents (Claude Code, Cursor, Windsurf, etc.) how to operate the [Numify AI CLI](https://numifyai.com) — bookkeeping for Polish sp. z o.o. companies.

## Install

### skills-cli

```bash
npx skills-cli add kacperkwapisz/numifyai-cli-skill
```

### Claude Code

```bash
mkdir -p .claude/skills/numifyai-cli
curl -fsSL https://numifyai.com/skills/numifyai-cli/SKILL.md \
  -o .claude/skills/numifyai-cli/SKILL.md
```

### Cursor

```bash
mkdir -p .cursor/rules
curl -fsSL https://numifyai.com/skills/numifyai-cli/SKILL.md \
  -o .cursor/rules/numifyai-cli.md
```

### Any other agent

Download the file and place it wherever your agent reads rules or system prompts:

```bash
curl -fsSL https://numifyai.com/skills/numifyai-cli/SKILL.md
```

## Prerequisites

1. Install the CLI: `npm i -g numifyai`
2. Create an API key at **Dashboard → CLI → API keys**
3. Set `NUMIFY_TOKEN` in your environment

## Links

- [Numify AI](https://numifyai.com)
- [Raw skill file](https://numifyai.com/skills/numifyai-cli/SKILL.md)
