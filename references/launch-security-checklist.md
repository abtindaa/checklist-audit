# Pre-launch web-app security checklist (30 items)

Items 1–20 are a meme checklist in circulation ("20 things to have Claude do
before launching your app"). Every item here has a concrete enforcement point
to grep for, unlike the concept-shaped entries in `systems-checklist.md` (CAP
Theorem, Latency, …). Items 21–30 were added afterward to cover gaps the
original 20 leave open — same "grep for the real enforcement point" character,
not a different tier of item.

At 30 items this is over the ~15 direct-investigation threshold but well under
the ~40 subagent threshold, so step 3's answer is: group by where the answer
lives, work group by group, no subagents needed.

## Transport & headers (1, 21, 29)

1. Add HSTS
21. Set CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy headers
29. Automate TLS certificate renewal (not just require HTTPS)

## Session & auth (2, 3, 4, 5, 17, 19, 24, 27)

2. Add CSRF tokens
3. Reset sessions on password change
4. Expire reset links
5. Prevent user enumeration
17. Lock accounts after failed logins
19. Set secure cookie flags
24. Require MFA/2FA on admin accounts
27. Use short-lived access tokens with a separate refresh token, not a
    long-lived JWT

## Authorization (22)

22. Check per-object authorization (IDOR) — authenticated ≠ allowed to access
    this specific resource

## Input handling & storage (6, 13, 23)

6. Whitelist upload types
13. Sanitize before storing
23. Prevent path traversal on file upload/download paths

## Payments & pricing (7, 8)

7. Verify payment webhooks
8. Set prices server-side

## AI/LLM surface (9, 10)

9. Block prompt injection
10. Cap AI usage

## Request/response hardening (11, 14, 15, 16, 20)

11. Limit request size
14. Lock down CORS
15. Disable directory listing
16. Remove default admin routes
20. Restrict database permissions

## Abuse prevention (12, 30)

12. Rate limit password resets
30. Rate limit all auth endpoints (login, signup, token refresh), not just
    password reset

## Secrets & supply chain (25, 26)

25. No secrets committed to git history (.env, API keys, tokens)
26. Automated dependency vulnerability scanning (npm audit / pip-audit /
    Dependabot or equivalent)

## Logging & monitoring (18, 28)

18. Log security events
28. Redact PII and secrets from logs — a logged event stream that itself
    leaks what it's supposed to be monitoring is a second breach on top of
    the first

## Notes for auditing this particular list

Overlaps with `systems-checklist.md` on a few items (CSRF, rate limiting,
CORS, webhooks) — if both lists get audited for the same project in the same
session, don't re-investigate an item twice; reuse the citation.

Several items are two-part and easy to half-verify:
- "Add CSRF tokens" (2) vs "Reset sessions on password change" (3) — a CSRF
  middleware existing doesn't mean session rotation on password change also
  exists; check both independently.
- "Verify payment webhooks" (7) vs "Set prices server-side" (8) — a project can
  verify webhook signatures correctly while still trusting a client-supplied
  price elsewhere (e.g. a checkout endpoint that accepts an `amount` field).
  Grep for where the price/amount actually originates, not just the webhook
  handler.
- "Prevent user enumeration" (5) is commonly half-done: login returns a
  generic error but the signup or password-reset endpoint still leaks via
  timing or a distinct "email already exists" message. Check all three
  endpoints, not just login.
- "No secrets committed to git history" (25) means the full history, not just
  the current tree — a key removed in a later commit is still present in
  earlier ones and in any fork/clone. `git log -p` or a secrets scanner
  (gitleaks / trufflehog) on full history, not `grep` on HEAD.
- "Automate TLS renewal" (29) and "Add HSTS" (1) are easy to conflate — a
  project can force HTTPS correctly while still renewing certs by hand
  (cron/manual), which is a real gap even though "TLS is on."

Items 6, 9, 10, 14, 16, 20, 22, 23, 24, 26 are only applicable if the
corresponding surface exists at all (file uploads, an LLM integration,
cross-origin clients, an admin panel, per-object resources, path-based file
access, admin accounts, a dependency manifest). Absence of the surface is a
legitimate "not applicable," not a reason to skip checking that it's absent.
