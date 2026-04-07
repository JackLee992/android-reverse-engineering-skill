# Codex Installation

This repository is published as a Claude plugin, so the repository root is not directly installable as a Codex skill.

The Codex-compatible packaging lives here:

```text
codex-skills/android-reverse-engineering
```

Install it with the bundled Codex installer:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo JackLee992/android-reverse-engineering-skill \
  --ref master \
  --path codex-skills/android-reverse-engineering
```

After installing, restart Codex so the new skill is discovered.

## Why this exists

The upstream skill relies on Claude plugin conventions such as:

- plugin metadata under `.claude-plugin/`
- runtime references to `${CLAUDE_PLUGIN_ROOT}`

Codex skill installation expects a plain directory containing `SKILL.md`, so this compatibility layer copies the skill assets into a standalone Codex-friendly layout and rewrites the path references to the default Codex skill location:

```text
$HOME/.codex/skills/android-reverse-engineering
```
