# checklist-audit

A [Claude Code](https://claude.com/claude-code) skill for security and
best-practices audits. It's a `SKILL.md` file (plus reference checklists)
that you drop into Claude Code so it knows how to run this kind of audit
consistently, instead of you re-explaining it every time.

**What it does:** you ask Claude Code to audit your project — for security
gaps, or against general engineering best practices — and it checks each
item against your actual source code (grep, read the real file, find the
real enforcement point), not from general training knowledge. Every result
comes with a `file:line` citation or an explicit "couldn't verify this."

**What problem this solves:** an LLM asked "do we have rate limiting on
password reset?" from memory will confidently say yes or no either way, and
a wrong "yes" is a security hole that never gets found. This skill's rule is
simple: no citation, no "already have."

You don't need to bring your own checklist. Say "security audit this
project" and it runs its own built-in checklists (see below). You can also
paste or show your own checklist/image, which takes priority for that run.

## What's bundled

| File | Items | Source |
|---|---|---|
| `references/launch-security-checklist.md` | 38 | pre-launch web-app security meme list + additions |
| `references/agent-security-checklist.md` | 64 | Cloudflare's [security-audit-skill](https://github.com/cloudflare/security-audit-skill) (core + AI/LLM) |
| `references/extended-attack-domains.md` | 90 | the rest of Cloudflare's security-audit-skill (web/auth, client-side, cloud, supply-chain, availability, data isolation, protocols, desktop/mobile) |
| `references/community-findings-checklist.md` | 16 | real practitioner-reported patterns |
| `references/systems-checklist.md` | 111 | backend/distributed-systems meme list |
| `references/default-checklist.md` | ~100 | general engineering best practices, no security framing |

The security lists split into two tiers: a "security audit" default
(launch-security + agent-security, 102 items) and an opt-in "deep audit"
(adds extended-attack-domains + community-findings, 208 items total) — a
live test showed running everything by default was too heavy for a quick
ask.

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
