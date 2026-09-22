# Pre-launch web-app security checklist (38 items)

Items 1–20 are a meme checklist in circulation ("20 things to have Claude do
before launching your app"). Every item here has a concrete enforcement point
to grep for, unlike the concept-shaped entries in `systems-checklist.md` (CAP
Theorem, Latency, …). Items 21–38 were added afterward to cover gaps the
original 20 leave open — same "grep for the real enforcement point" character,
not a different tier of item.

At 38 items this is over the ~15 direct-investigation threshold but well under
the ~40 subagent threshold, so step 3's answer is: group by where the answer
lives, work group by group, no subagents needed.

## Transport & headers (1, 21, 29)

1. Add HSTS
21. Set CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy headers
29. Automate TLS certificate renewal (not just require HTTPS)

## Session & auth (2, 3, 4, 5, 17, 19, 24, 27, 31, 32)

2. Add CSRF tokens
3. Reset sessions on password change
4. Expire reset links
5. Prevent user enumeration
17. Lock accounts after failed logins
19. Set secure cookie flags
24. Require MFA/2FA on admin accounts
27. Use short-lived access tokens with a separate refresh token, not a
    long-lived JWT
31. Hash passwords with a slow memory-hard KDF (bcrypt/argon2/scrypt), not a
    fast general-purpose hash (MD5, SHA1, unsalted SHA256) or plaintext
32. Check new/changed passwords against a known-breach corpus (Pwned
    Passwords range API or an offline equivalent) before accepting them

## Authorization (22, 33)

22. Check per-object authorization (IDOR) — authenticated ≠ allowed to access
    this specific resource
33. Mass assignment / over-posting — a create/update endpoint binds the whole
    request body to a model/ORM record without an explicit allow-list,
    letting a client set `role`, `isAdmin`, `balance`, etc. just by including
    the field in the payload

## Input handling & storage (6, 13, 23)

6. Whitelist upload types
13. Sanitize before storing
23. Prevent path traversal on file upload/download paths

## Payments & pricing (7, 8)

7. Verify payment webhooks
8. Set prices server-side

## AI/LLM surface (9, 10, 36, 37)

9. Block prompt injection
10. Cap AI usage
36. If an AI coding agent has shell/filesystem access to this repo, confirm it
    can't reach production credentials, production data, or push/deploy
    unreviewed — not the app's own runtime AI surface, but the development
    tooling around it
37. If the app has a code-interpreter-style feature (executes model-generated
    code as a product feature), confirm it runs sandboxed/isolated with no
    more privilege than the host process, not just "the coding agent is safe"

## Request/response hardening (11, 14, 15, 16, 20, 34)

11. Limit request size
14. Lock down CORS
15. Disable directory listing
16. Remove default admin routes
20. Restrict database permissions
34. If the API is GraphQL: introspection disabled in production, and a query
    depth/complexity limit exists (a single request can't construct an
    unbounded nested/fan-out query)

## Abuse prevention (12, 30)

12. Rate limit password resets
30. Rate limit all auth endpoints (login, signup, token refresh), not just
    password reset

## Secrets & supply chain (25, 26, 35)

25. No secrets committed to git history (.env, API keys, tokens)
26. Automated dependency vulnerability scanning (npm audit / pip-audit /
    Dependabot or equivalent)
35. Dependency slopsquatting — a package name was hallucinated by an LLM
    (dependency selection, a copied tutorial, a coding agent) and installed as
    written; check for lockfile entries that are unusually obscure, added
    recently with no clear reason in commit history, or don't match what a
    human would plausibly have searched for

## Logging & monitoring (18, 28)

18. Log security events
28. Redact PII and secrets from logs — a logged event stream that itself
    leaks what it's supposed to be monitoring is a second breach on top of
    the first

## Process & disclosure (38)

38. A responsible-disclosure path exists — `/.well-known/security.txt`, a
    `SECURITY.md`, or a published security contact — so an outside researcher
    has a documented way to report a vulnerability

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
- "No secrets committed to git history" (25) and "dependency slopsquatting"
  (35) are both supply-chain items but need different checks — one is history
  scanning, the other is eyeballing the lockfile for a package that
  shouldn't exist. Don't let the shared group heading collapse them into one
  investigation.
- Item 32 (breached-password check) is about what password gets *accepted*;
  item 31 (hashing algorithm) is about how the accepted password gets
  *stored*. A project can do one without the other.

Items 6, 9, 10, 14, 16, 20, 22, 23, 24, 26, 34, 36, 37 are only applicable if
the corresponding surface exists at all (file uploads, an LLM integration,
cross-origin clients, an admin panel, per-object resources, path-based file
access, admin accounts, a dependency manifest, GraphQL, an AI coding agent
with repo access, a code-interpreter-style feature). Absence of the surface is
a legitimate "not applicable," not a reason to skip checking that it's absent.
