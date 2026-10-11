# checklist-audit

**A security audit skill for vibe-coded projects.** A
[Claude Code](https://claude.com/claude-code) skill that checks your project
against more than 200 built-in security items, focused on the issues that most
often affect AI-assisted codebases. It checks the actual code, not general
knowledge, and every covered item comes with a `file:line` citation.

## Why

Ask an LLM "do we have rate limiting on password reset?" and it will answer
confidently either way. A wrong "yes" is a security hole that never gets
found. This skill has one rule: **no citation, no "already have."** Anything
it can't prove from the code is reported as unverified, not assumed safe.

## What it checks

You don't need to bring a checklist — the skill ships its own:

| File | Items | Source |
|---|---|---|
| `references/launch-security-checklist.md` | 38 | Widely shared pre-launch web-app security checklist, plus additions |
| `references/agent-security-checklist.md` | 64 | Adapted from Cloudflare's [security-audit-skill](https://github.com/cloudflare/security-audit-skill) (core attack patterns + AI/LLM) |
| `references/extended-attack-domains.md` | 90 | Adapted from the rest of Cloudflare's security-audit-skill (web/auth, client-side, cloud, supply chain, availability, data isolation, protocols, desktop/mobile) |
| `references/community-findings-checklist.md` | 16 | Real incidents reported by practitioners (agent memory leaks, MCP credentials, WAF bypass, logging leaks, …) |
| `references/systems-checklist.md` | 111 | Backend and distributed-systems engineering concepts |
| `references/default-checklist.md` | ~100 | General engineering best practices |

Security audits run in two tiers:

- **Security audit** (default): launch + agent-security lists, 102 items.
- **Deep security audit** (opt-in): adds extended domains + community
  findings, 208 items.

You can also paste or show your own checklist, as text or an image, and it is
used instead.

## How it works

1. **Asks which checklist to run**, unless you already named one or provided
   your own.
2. **Reads the project first**: stack, architecture, `CLAUDE.md`/`README`, and
   any earlier security reports and recorded decisions, so known issues aren't
   re-reported as new.
3. **Investigates each item in the real code**, grouped by area. Large lists
   are split across parallel subagents that all return a fixed verdict format.
4. **Re-verifies every gap before reporting it**: the whole code path,
   including callers, middleware and proxy config, not just the cited line. It
   also checks any proposed fix against real data.
5. **Sorts every item into four buckets**: covered, worth fixing, unverified,
   or not applicable (with the architectural reason). Risks the owner
   deliberately accepted aren't reported as bugs.
6. **Writes the findings** where the project already keeps its audit reports
   (for example `docs/security-*.md`), so the next audit can build on them.

## Example output

Illustrative:

```
## Already covered
L-1  HSTS — max-age=63072000; includeSubDomains; preload [deploy/nginx/nginx.conf:47]
L-5  User enumeration — constant-time dummy hash, identical error message [auth_service.py:25-33]

## Worth fixing
A-38 Race in winner selection — check-then-act with no row lock [quote_service.py:513]
C-15 loguru diagnose=True — local variables (incl. an API key) can reach logs [core/log.py:69]

## Unverified
C-9  Origin reachable without the CDN? — firewall rules live only on the host; run `iptables -S DOCKER-USER`

## Not applicable
L-7  Payment webhooks — no payment gateway in this project
```

Start with: A-38 — directly exploitable by two concurrent requests.

## Install

For all projects:

```bash
cd ~/.claude/skills
git clone https://github.com/abtindaa/checklist-audit.git
```

For one project only:

```bash
cd your-project
git clone https://github.com/abtindaa/checklist-audit.git .claude/skills/checklist-audit
```

## Usage

In Claude Code, inside your project:

```
security audit this project
```

```
پروژه رو از نظر امنیتی چک کن
```

```
what best practices are we missing
```

Or paste/show your own checklist and ask "do we have this?" See
[SKILL.md](SKILL.md) for the full workflow and decision rules.

## Requirements

- Claude Code with access to your project's source.
- Subagent support (the Agent tool) is recommended. Lists over ~40 items, such
  as the deep audit, are split across parallel subagents.
- Works with any stack: Python, JS/TS, Go, and others.

## Limitations

- This is a **source-level audit**, not a penetration test. It doesn't send
  traffic to your services or exploit anything.
- Facts outside the repository can't be confirmed from code, so they are
  reported as **unverified** with the exact check to run. Examples: host
  firewall rules, CDN settings, production environment variables.
- Results depend on what the checklists cover. A clean report means these
  items were checked, not that the project is secure in every respect.

## Contributing

Issues and pull requests are welcome, especially:

- New checklist items grounded in a real incident.
- Reports of false positives or false negatives from real runs.

## License

MIT — see [LICENSE](LICENSE). The two lists adapted from Cloudflare's
security-audit-skill keep Cloudflare's MIT copyright notice; see
[NOTICE](NOTICE).
