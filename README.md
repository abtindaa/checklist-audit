# checklist-audit

A [Claude Code](https://claude.com/claude-code) skill that audits a project
against real checklists — checked against the actual code, not answered from
general knowledge.

No list from you is required. Say something like "security audit this
project" and the skill runs its own built-in checklists. You can also paste
or show your own checklist/image and it takes priority for that run.

## Why

Asking an LLM "do we have CSRF protection" from memory is worse than useless
— every answer sounds plausible, and a wrong "yes we have that" is a security
hole that never gets found. This skill's rule: every verdict must be backed by
something actually read during that run (a grep hit, a file, a config line).
Anything it can't point to goes in "unverified," never in "already have."

## What's bundled

| File | Items | Source |
|---|---|---|
| `references/launch-security-checklist.md` | 30 | pre-launch web-app security meme list + additions |
| `references/agent-security-checklist.md` | 64 | Cloudflare's [security-audit-skill](https://github.com/cloudflare/security-audit-skill) (core + AI/LLM) |
| `references/extended-attack-domains.md` | 90 | the rest of Cloudflare's security-audit-skill (web/auth, client-side, cloud, supply-chain, availability, data isolation, protocols, desktop/mobile) |
| `references/model-knowledge-additions.md` | 8 | model's own training knowledge — flagged with lower confidence |
| `references/community-findings-checklist.md` | 16 | real practitioner-reported patterns |
| `references/systems-checklist.md` | 111 | backend/distributed-systems meme list |
| `references/default-checklist.md` | ~100 | general engineering best practices, no security framing |

Output is always four buckets: already covered (with `file:line` proof),
genuinely worth fixing, unverified, or not applicable (with the specific
architectural reason).

## Install

```bash
cd ~/.claude/skills
git clone https://github.com/abtindaa/checklist-audit.git
```

Or scope it to one project instead of all of them:

```bash
cd your-project
git clone https://github.com/abtindaa/checklist-audit.git .claude/skills/checklist-audit
```

## Usage

```
security audit this project
```

```
پروژه رو از نظر امنیتی چک کن
```

```
what best practices are we missing
```

Or paste/show your own checklist and ask "do we have this."

See [SKILL.md](SKILL.md) for the full workflow and decision logic.
