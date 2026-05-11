# Sendry Agent Skills

Skill files for AI coding assistants (Claude, Cursor, GitHub Copilot, etc.) working with Sendry. Each `SKILL.md` provides authoritative, opinionated guidance that agents can read before generating code or answering questions in that domain.

## Skills

| Skill | Description |
|---|---|
| [`email-best-practices/`](./email-best-practices/SKILL.md) | Deliverability, HTML structure, subject lines, list hygiene, legal requirements |
| [`react-email/`](./react-email/SKILL.md) | Building cross-client email templates with `@react-email/components` |
| [`sendry-api/`](./sendry-api/SKILL.md) | Sending emails, managing contacts & campaigns, webhooks, error handling |

## Usage

Point your AI assistant at the relevant skill file before starting a task:

```
Read packages/skills/sendry-api/SKILL.md before helping me integrate Sendry.
```

Or reference all three for full-stack email work:

```
Read all SKILL.md files under packages/skills/ before starting.
```
