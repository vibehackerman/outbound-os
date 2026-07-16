```
 _____   __  __  ______  ____     _____   __  __  __  __  ____       _____   ____
/\  __`\/\ \/\ \/\__  _\/\  _`\  /\  __`\/\ \/\ \/\ \/\ \/\  _`\    /\  __`\/\  _`\
\ \ \/\ \ \ \ \ \/_/\ \/\ \ \L\ \\ \ \/\ \ \ \ \ \ \ `\\ \ \ \/\ \  \ \ \/\ \ \,\L\_\
 \ \ \ \ \ \ \ \ \ \ \ \ \ \  _ <'\ \ \ \ \ \ \ \ \ \ , ` \ \ \ \ \  \ \ \ \ \/_\__ \
  \ \ \_\ \ \ \_\ \ \ \ \ \ \ \L\ \\ \ \_\ \ \ \_\ \ \ \`\ \ \ \_\ \__\ \ \_\ \/\ \L\ \
   \ \_____\ \_____\ \ \_\ \ \____/ \ \_____\ \_____\ \_\ \_\ \____/\_\\ \_____\ `\____\
    \/_____/\/_____/  \/_/  \/___/   \/_____/\/_____/\/_/\/_/\/___/\/_/ \/_____/\/_____/
```

> **The AI-native cold-outbound pipeline [Ryzo](https://ryzo.nl) runs on** — open-sourced as a Claude Code plugin.

[![Claude Code Ready](https://img.shields.io/badge/Claude_Code-ready-5A45FF)](https://claude.com/claude-code)
[![Skills](https://img.shields.io/badge/skills-9-E0621A)](#the-pipeline)
[![Commands](https://img.shields.io/badge/commands-2-2A7DE1)](#the-pipeline)
[![License: MIT](https://img.shields.io/badge/license-MIT-3DA639)](LICENSE)

Nine skills chain from raw company context to sent campaigns, plus two commands to stand up your sending infrastructure.

## Install

**As a plugin (recommended):**
```
/plugin marketplace add initforthevibe/ryzo
/plugin install outbound-os@ryzo
```

**As plain skills (copy-in):**
```
git clone https://github.com/initforthevibe/outbound-os
cp -r outbound-os/skills/* ~/.claude/skills/
cp -r outbound-os/commands/* ~/.claude/commands/
```

## Quick start

Once installed, drive it in plain language — each skill auto-triggers on the right ask and hands its output to the next. A full run reads like a conversation:

1. *"Build company context for **my company**"* → **company-context-builder** interviews you and writes a reusable context file.
2. *"Research the problems our ICP faces"* → **market-problems-deep-research** builds a pain/problem taxonomy.
3. *"Build a target list of **&lt;segment&gt;**"* → **list-building** drafts a hypothesis-driven list.
4. *"Enrich these prospects"* → **data-points-builder** then **table-enrichment** add per-prospect datapoints.
5. *"Tier this list"* → **tiering-segmentation** scores and segments into outreach strategies.
6. *"Write emails for tier 1"* → **email-generation** assembles personalized copy; **copy-feedback** stress-tests it against prospect psychology.
7. *"Launch this in lemlist"* → **run-lemlist** builds the campaign, enrols the leads, and monitors deliverability.

One-time setup before your first send: run the **domain-infrastructure-setup** and **domain-warmup-orchestrator** commands to stand up and warm secondary sending domains.

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

## Why we open-sourced it

This is the actual pipeline we run — not a demo. We publish it because good GTM playbooks compound in the open: markdown-native (no lock-in), version-controlled, and forkable. Take it, adapt it to your ICP, and ship better outbound. If it helps, [tell us](https://ryzo.nl).

## About

Built by Ryzo — the AI-native GTM & RevOps agency. This is the real OutboundOS
pipeline we run for clients, genericized for open use. MIT licensed.
