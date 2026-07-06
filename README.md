# OutboundOS

The AI-native cold-outbound pipeline [Ryzo](https://ryzo.nl) runs on — open-sourced
as a Claude Code plugin. Nine skills chain from raw company context to sent campaigns,
plus two commands to stand up your sending infrastructure.

## Install

**As a plugin (recommended):**
```
/plugin marketplace add ryzo-revops/outbound-os
/plugin install outbound-os@ryzo
```

**As plain skills (copy-in):**
```
git clone https://github.com/ryzo-revops/outbound-os
cp -r outbound-os/skills/* ~/.claude/skills/
cp -r outbound-os/commands/* ~/.claude/commands/
```

## The pipeline

Each skill consumes the previous step's output:

1. **company-context-builder** — living context file: product, ICP, lingo, win cases
2. **market-problems-deep-research** — pain/problem taxonomy for your ICP
3. **list-building** — hypothesis-driven target list
4. **data-points-builder** — per-prospect research datapoints
5. **table-enrichment** — fill and augment the prospect table
6. **tiering-segmentation** — score and tier into outreach strategies
7. **email-generation** — instruction-following email assembly from datapoints
8. **copy-feedback** — multi-layer prospect-psychology review before send
9. **run-lemlist** — configure, launch, and monitor campaigns in lemlist

**Infrastructure commands:** the **domain-infrastructure-setup** and
**domain-warmup-orchestrator** commands stand up and warm secondary sending domains
before step 9.

## Prerequisites

- A [lemlist](https://lemlist.com) account (with lemwarm) for sending.
- Your own API keys, supplied via environment variables (e.g. `LEMLIST_API_KEY`).
  No keys ship in this repo; none are stored in any skill file.
- Secondary sending domains (the two infrastructure commands help you set these up).

## About

Built by Ryzo — the AI-native GTM & RevOps agency. This is the real OutboundOS
pipeline we run for clients, genericized for open use. MIT licensed.
