# x402 — Codex Skill for BSV Micropayments

Pay AI agents with Bitcoin (BSV) from inside [Codex CLI](https://github.com/openai/codex). Generate images, videos, transcribe audio, search Twitter — all with micropayments.

## Install

```bash
# 1. Clone to the user-level skills directory
mkdir -p ~/.agents/skills
git clone https://github.com/Calgooon/codex-x402.git ~/.agents/skills/x402

# If your setup uses CODEX_HOME, clone there instead:
# mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
# git clone https://github.com/Calgooon/codex-x402.git "${CODEX_HOME:-$HOME/.codex}/skills/x402"

# 2. Install Python dependency
pip3 install requests

# 3. Install MetaNet Client wallet (required for signing & payments)
# Download from https://getmetanet.com
# Launch it — it runs at localhost:3321
```

> **How Codex finds skills:** Codex auto-discovers any directory containing a `SKILL.md` under `~/.agents/skills/`. Some environments use `$CODEX_HOME/skills/` instead, so clone there if your setup is configured that way. You can also put skills in `.agents/skills/` inside any git repo for project-scoped use. See [Codex Skills docs](https://developers.openai.com/codex/skills).

## Use

Launch Codex with full-auto mode (network access required for auth + payments):

```bash
codex --full-auto
```

Then just ask:

```
> generate an image of a mountain sunset
> search twitter for bitcoin scaling debates
> what agents are available?
> transcribe this audio file
```

Codex picks up the x402 skill automatically and handles BRC-31 authentication + BRC-29 micropayments behind the scenes.

## Manual test

Verify the scripts work outside Codex:

```bash
HELPER="$HOME/.agents/skills/x402/scripts/brc31_helpers.py"
if [ ! -f "$HELPER" ] && [ -n "${CODEX_HOME:-}" ]; then
  HELPER="$CODEX_HOME/skills/x402/scripts/brc31_helpers.py"
fi

# List available agents
python3 "$HELPER" list

# Check wallet is running
python3 "$HELPER" identity

# Discover banana agent pricing
python3 "$HELPER" discover banana

# Check installed helper version
python3 "$HELPER" --version
```

## What's included

| Agent | What it does | Cost |
|:------|:-------------|:-----|
| **banana** | AI image generation | ~$0.19/image |
| **veo** | AI video generation with audio | ~$0.75-$1.50/clip |
| **whisper** | Speech-to-text | ~$0.0006/min |
| **x-research** | Twitter/X search | ~$0.005-$0.06/req |
| **nanostore** | File hosting | ~$0.0004/MB/yr |

## Requirements

- macOS (MetaNet Client)
- Python 3.8+
- [Codex CLI](https://github.com/openai/codex) (`npm i -g @openai/codex`)
- [MetaNet Client](https://getmetanet.com) running at localhost:3321
