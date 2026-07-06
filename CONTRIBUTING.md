# Contributing

Thanks for your interest. This repo mirrors skills Ryzo uses in production.

- **Structure:** each skill is a directory under `skills/` with a single `SKILL.md`
  (YAML frontmatter `name` + `description`, then the skill body). Commands are
  single `.md` files under `commands/`.
- **No secrets, ever.** No API keys, tokens, client names, or internal URLs. All
  credentials are bring-your-own via environment variables.
- **PRs:** keep changes focused, explain the outbound rationale, and confirm no
  secrets or client-identifying data were added. CI runs a secret scan.
