---
name: checklist-audit
description: Audit this project for real, using built-in checklists — a 38-item pre-launch security list, a 64-item Cloudflare attack-pattern list, a 90-item extended attack-domain list, a 111-item backend/distributed-systems list, and a general engineering list — checked against the actual code, not general knowledge. No list from the user is required; a pasted or shown checklist/image is also accepted and takes priority when given. Produces a grounded triage: already implemented (file:line proof), a genuine gap, unverified, or not applicable given this project's real architecture. Use whenever the user asks for a security audit or review ("security audit this project", "پروژه رو از نظر امنیتی چک کن", "چی از نظر امنیتی کم داریم"), a general best-practices audit ("چک کن چی کم داریم", "which of these do we need"), or pastes/shows their own list ("do we have this") — even without naming this skill. Do NOT use for a specific diff or PR (use code-review) — this audits the whole project.
---

# Checklist audit

The point of this skill is refusing to answer from memory. A list like "add HSTS,
rate-limit password resets, cap AI usage" is trivial to answer generically — every
item sounds plausible, half of them are already standard practice somewhere, and
a confident-sounding answer built from training data is worse than useless here:
it tells the user nothing about *this* codebase, and a wrong "yes we have that"
is a security hole that never gets found.

Every verdict below must be backed by something you actually read this run —
a grep hit, a file you opened, a config line. If you can't point to it, the
item goes in the "unverified" bucket, never in "already have."

Works for any stack — Python, JS/TS, Go, whatever the current project turns
out to be.

The user does not need to supply anything. "Security audit this project" is a
complete request on its own — the checklists live in `references/`, not in the
user's head. A pasted or shown list is a second, equally valid way to invoke
this skill, and when given, it replaces the bundled list for that run (see
step 1).

## Bundled lists

Six lists are pre-loaded under `references/`, all pre-grouped by where the
answer lives so step 4's grouping is already done. These are the skill's own
checklists, not a fallback for when the user forgot to bring one — running
against them *is* the default way this skill operates.

The four security-flavored lists split into two tiers (see step 1's ask):
`launch-security-checklist.md` (38) and `agent-security-checklist.md` (64)
are the default "security audit" — 102 items, tested end-to-end on a real
project without becoming a marathon. `extended-attack-domains.md` (90) and
`community-findings-checklist.md` (16) are the "deep audit" add-on — 208
items total, meaningfully heavier and opt-in rather than the default.

- `references/launch-security-checklist.md` — the 38-item pre-launch web-app
  security list: the original 20-item meme list (HSTS, CSRF tokens, prompt
  injection, payment webhooks, …) plus 18 added items covering common gaps
  (IDOR/authorization, path traversal, MFA, secrets in git history, dependency
  scanning, log redaction, password hashing, mass assignment, GraphQL
  introspection, dependency slopsquatting, unsandboxed AI coding agents,
  responsible-disclosure path, …). Narrower and more concrete than the
  systems list; every item has a real enforcement point to grep for. Under
  the subagent threshold — direct group-by-group investigation.
- `references/agent-security-checklist.md` — the 64-item attack-pattern
  checklist sourced from Cloudflare's `security-audit-skill`, not a meme list.
  Most items are patterns to trace through actual data flow ("follow untrusted
  input to its sink"), not features to grep for — a materially different
  investigation style from the other three lists. Its last 14 items (AI/LLM/
  agent-specific: prompt injection, MCP trust, tool-argument injection, memory
  poisoning, …) apply only when the project has a model in a trust-sensitive
  loop — rule the whole group "not applicable" together when it has none,
  after actually checking for an LLM SDK/API key/prompt-template, not on
  assumption. Over the ~40 threshold — parallel-subagent branch of step 4, one
  agent per group in the file.
- `references/extended-attack-domains.md` — the 90-item domain checklist
  covering the rest of Cloudflare's `security-audit-skill`: web protocol/auth,
  client-side/browser, cloud/deployment, supply-chain/CI, resource
  exhaustion/availability, data isolation (multi-tenant), protocols/RPC/
  messaging, and desktop/mobile/local-IPC. The first five domains (items 1–74)
  are broadly applicable to most web/cloud projects; the last two (protocols/
  messaging, items 75–82, and desktop/mobile/IPC, items 83–90) apply only if
  that architecture is actually present — check the dependency tree and build
  target before ruling a domain in or out. Over the ~40 threshold —
  parallel-subagent branch of step 4, one agent per domain.
- `references/systems-checklist.md` — the 111-item backend/distributed-systems
  list ("who's gonna tell vibe coders about… Rate Limiting, Caching,
  Kubernetes, CAP Theorem…"). Mostly infra/architecture concepts, several
  concept-shaped rather than feature-shaped (see its own notes on turning
  those into concrete questions). Over the ~40 threshold — intended for the
  parallel-subagent branch of step 4.
- `references/default-checklist.md` — for a general (non-security) engineering
  audit only: "چک کن چی کم داریم", "audit us against general best practices"
  with no security framing and no list of the user's own. Themed groups exist
  only to make investigation efficient; the output is still the skill's normal
  flat four-bucket format, not grouped by theme.
- `references/community-findings-checklist.md` — 16 items from the user's own
  review of real practitioner discussions (agent-memory secret leaks, MCP
  credential exposure, misplaced content guards, approval gates after the
  side effect, API keys not hashed like passwords, WAF bypass via reachable
  origin, mutable Action internals despite SHA-pinning, RLS recursion,
  reconcatenated SQL inside a DB function, per-process rate limits behind a
  multi-replica LB, spoofable client-IP trust, debug libraries dumping locals
  into logs, silent-skip artifact signing, unreviewed agent-skill installs).
  Not a single vetted document like Cloudflare's skill — verify the specific
  mechanism against this project's actual stack, since an item naming a
  specific framework may simply not apply here.

All six lists supply *items*, never verdicts: every item still goes through
step 5, and on a small single-server project a large fraction of the systems
list in particular is correctly "not applicable." When running more than one
list in the same session, check overlapping items once and reuse the citation
(each file's own notes section flags its overlaps with the others).

## Workflow

1. **If the user already supplied a list, skip straight to parsing it** —
   pasted text, a shown image, or an explicit name ("run the 111 things",
   "audit against Cloudflare's list"). That list wins outright; don't ask
   anything else, don't touch the bundled lists.

   **Otherwise, before doing anything else, explain the bundled lists and ask
   which to run.** Briefly state in the chat that this skill ships six
   built-in checklists rather than needing one from the user, name them
   (38-item pre-launch security, 64-item Cloudflare attack-pattern list,
   90-item extended attack-domain list, 16-item community findings list,
   111-item backend/systems, general engineering), and use `AskUserQuestion`
   (allow multiple selection) to let the user pick. Pre-select nothing —
   recommend in the option description, don't decide for them:
   - "Security audit (recommended for most asks)" — just the 38 + 64 lists
     (102 items). This is the default weight: enough to catch real gaps
     without the run turning into a marathon. A live test on a real 30-item
     run alone took a meaningful amount of investigation; running all the
     security lists by default made the "just audit my project" case too
     heavy, so the wider lists moved to their own tier below.
   - "Deep security audit" — adds `extended-attack-domains.md` (90) and
     `community-findings-checklist.md` (16) on top of the 102 above (208
     items total). Note in the option description that this is significantly
     heavier and better suited to a dedicated security review than a quick
     check — the 90-item list's architecture-gated domains still get ruled
     in/out individually during investigation, not skipped wholesale.
   - "General engineering audit" — the default-checklist
   - "Full backend/systems audit" — the 111-item list, note it's mostly
     infra concepts and often heavily "not applicable" on small projects
   - "I have my own list" — if picked, prompt for it and go parse that instead

   Skip this ask only when the user's phrasing already names which one they
   want ("run the security checklist", "do the 111-item audit") — then load
   that directly. The ask exists for the ambiguous case ("audit this project",
   "چک کن چی کم داریم" with no further detail), not to add a confirmation step
   to an already-specific request.

2. **Parse whichever list is now selected.** If the source is an image, read
   it fully, don't paraphrase from a glance. Number the items if they aren't
   already.

3. **Load project context first.** Before checking anything, find out what this
   project actually is: its stack, its architecture, its binding constraints,
   and anything already known to be a gap. Look for whatever this project uses
   as its own source of truth — `CLAUDE.md`, `README.md`, `docs/architecture*`,
   `AGENTS.md`, an onboarding doc, or failing all of that, `package.json` /
   `pyproject.toml` / `go.mod` / etc. to at least identify the stack — and grep
   it for the topic area too, since a prior session may have already recorded a
   finding. Getting this step wrong wastes the whole audit: a Kubernetes item is
   only "not applicable" *because* the deploy story is one docker-compose VPS
   (or whatever it actually is here) — not because Kubernetes items are
   generically skippable everywhere.

4. **Plan the investigation against the list's size.** Count the items first.
   - **Up to ~15 items:** investigate each one directly, in whatever order is
     cheapest.
   - **More than ~15:** do not fire one grep per item. Group the items by where
     the answer lives (auth, request middleware, data layer, deploy/infra, AI
     surface, …), then work group by group — one pass of reads over the auth
     code answers every auth item at once. Report progress per group so a long
     audit stays legible.
   - **More than ~40, or groups that need deep independent digging:** dispatch
     the groups to parallel `Explore` subagents, one group per agent, each with
     the project-context summary from step 3 and its own item numbers. Require
     each agent to return `file:line` citations; an agent that reports a verdict
     with no citation gets its items placed in "unverified," not "already have."

5. **Investigate every item for real.** For each item, form a concrete question
   ("is there a rate limit on X route", "does the User model have a lockout
   column", "is CSP set in the reverse-proxy config") and go answer it with
   Grep/Read/Bash, the same way you'd verify any other claim about the code.
   Prefer reading the actual enforcement point (the middleware, the model, the
   config file) over a docstring or comment that merely claims something is
   handled — comments drift from code.

   Common places the answer lives, whatever the stack turns out to be:
   - Request/response mechanics, headers, rate limits → middleware/interceptor
     files, reverse-proxy config, route decorators or guards
   - Auth/session behavior → the login/token service, the user/session model
   - Data handling → the service + data-access layer, and separately the
     response schema/serializer/DTO (a field the service computes but the
     response type doesn't declare gets silently dropped — check both, not
     just the service function)
   - Infra items (TLS, CORS, backups, permissions) → deploy configs, the
     project's own deployment docs, docker-compose / CI / IaC files

6. **Sort into exactly four buckets.** An item can only end up in "not
   applicable" if you can name the specific architectural fact that makes it
   so (e.g. "no self-service password reset flow exists — admin resets
   directly" is a real reason; "we're small" is not, by itself, unless the
   project's own docs treat scale as a settled decision). Anything you ran out
   of budget on, couldn't locate, or only half-confirmed goes in "unverified"
   with the specific open question — that bucket existing is what keeps the
   other three honest, so leaving it empty is only correct when it's true.

7. **Record the confirmed findings.** Append the audit's durable conclusions —
   the confirmed gaps, and the non-obvious "we already have this, here's where"
   facts — to whatever the project uses as its source of truth (`CLAUDE.md`,
   the architecture doc, an issue tracker if the user prefers). Date them and
   cite the files. Without this, the next session re-derives the same answers
   from scratch and step 3's grep for prior findings has nothing to find. Skip
   only if the user asked for a throwaway answer.

## Output format

Four sections, in this order. Translate the headings into whatever language the
user is working in, and drop the "unverified" section entirely if it's empty:

1. **Already covered** — no action needed
2. **Genuinely worth fixing** — real gaps for this project
3. **Unverified** — couldn't confirm this run, with the open question
4. **Not applicable** — with the architectural reason

Each item is **one line**: verdict-relevant fact + terse reason, with a
`file:line` citation for anything you found in code. No preamble, no per-item
paragraph, no restating the item's name at length — the user already has the
list.

Example output, from a Persian-language project — match the user's language,
not this one:

```
## داریم (نگران نباش)
۲. CSRF — میدلور اختصاصی داره (double-submit token) [middlewares/csrf.py]
۵. Prevent user enumeration — verify_password روی هش ساختگی، پیام خطای یکسان [auth_service.py:26-33]

## واقعاً بدرد ما می‌خوره
۹. Block prompt injection — یادداشت تأمین‌کننده مستقیم توی پرامپت میره، بدون علامت‌گذاری «این متن کاربره» [ai_summary_service.py:298]
۱۷. Lock accounts after failed logins — فقط rate-limit روی IP هست، نه قفل حساب

## تأیید نشد
۲۳. Backup restore drill — اسکریپت بکاپ هست ولی جایی پیدا نشد که تست بازیابی انجام شده باشه؛ باید از تیم دیپلوی پرسید

## موضوعیت نداره
۷. Verify payment webhooks — این پروژه gateway پرداخت نداره
```

Close with one line naming which "worth fixing" item to start with and why —
not a restated summary of everything, just the actual next action.

Keep it in normal prose register, not compressed slang — this is content the
user has to act on precisely, same as a code review or security finding.

## Things that go wrong if you skip the investigation step

- Claiming a feature is "standard so probably fine" when the actual code has a
  specific, documented exception (e.g. a temporary permission widening, a
  known-broken alerting channel, a rate limit that's per-IP when the item asks
  about per-account).
- Missing that a backend service computes a field but the schema/serializer/DTO
  never declares it — the field is invisibly dropped from every response, and
  grepping only the service function makes it look shipped when it isn't.
- Calling something "not applicable" because it sounds enterprise-scale, when
  the project's own docs show the identical risk already happened at small
  scale (a real un-alerted outage, a real near-miss) — scale is not a reason to
  skip, only architecture is.
- Padding the "already have" bucket to look thorough. A short, confident, well-
  cited list beats a long one where half the citations are guesses — and
  "unverified" is always available for the ones you couldn't nail down.
