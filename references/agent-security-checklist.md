# AI-agent-introduced security checklist (64 items)

Source: Cloudflare's [security-audit-skill](https://github.com/cloudflare/security-audit-skill)
(`ATTACK-CLASSES.md` and `AI-AND-LLM.md`), a real multi-agent security-audit
tool, not a meme list. Distinct in character from `systems-checklist.md` and
`launch-security-checklist.md`: those are individually-named features/controls
("add HSTS"); most items here are attack *patterns* that only make sense
against actual code paths — "trace untrusted input to a dangerous sink" isn't
something you grep for, it's something you go read the data flow for.

At 64 items this is over the ~40 threshold — the parallel-subagent branch of
step 3 applies, one agent per group below. Groups 1–9 come from
`ATTACK-CLASSES.md` and apply to any codebase. Group 10 comes from
`AI-AND-LLM.md` and applies only when a language model participates in a
trust-sensitive decision (chatbot, RAG, tool-calling agent, MCP server/client,
code that builds prompts from untrusted input or acts on model output) — skip
it entirely for a codebase with no LLM integration, and say so as the
architectural reason.

## 1. Baseline exposures (items 1–14)

The things everyone assumes someone else already checked — cheap to verify,
easy to miss.

1. Hardcoded passwords, API keys, tokens, or secrets in source
2. Security-relevant TODO/FIXME/HACK comments never resolved (`TODO: add auth`)
3. Debug/dev mode enablable in production via env var, query param, or header
4. Test/example/seed credentials that still work in production
5. Unprotected `/debug`, `/admin`, `/status`, `/.env`, `/config`, `/metrics` endpoint
6. `.env`, `*.pem`, `*.key`, `credentials.json` committed to the repo
7. `.gitignore` that doesn't actually cover secrets/uploads/local config
8. Unpinned dependencies or known CVEs in the lockfile
9. `eval()`, `exec()`, `child_process`, `Function()`, dynamic `import()` with untrusted input
10. CORS set to `*` combined with `Access-Control-Allow-Credentials`
11. Cookies missing `HttpOnly`, `Secure`, or `SameSite`
12. Open redirects via `redirect`/`return`/`next`/`url`/`goto` parameters
13. TLS not enforced anywhere; HTTP-only endpoints exist
14. Production error responses leaking stack traces, internal paths, or SQL errors

## 2. Injection (items 15–22)

15. Untrusted input reaching a SQL query
16. Untrusted output reaching HTML without encoding (XSS)
17. Untrusted input reaching a shell command
18. Untrusted input reaching a template engine
19. Unsafe deserialization
20. Indirect injection — data stored safely, then used in a dangerous context later
21. Injection via field names, keys, headers, or metadata, not just values
22. Injection into secondary systems: logs, caches, search indexes, analytics

## 3. Access control (items 23–26)

23. A second path to the same state change checks a weaker permission
24. A request-body field overrides what the permission system meant to restrict
25. An endpoint checks authentication but forgets authorization
26. Bulk/batch/export/import operations skip per-item permission checks

## 4. Resource and file handling (items 27–30)

27. Path traversal via symlinks, encoded sequences, or null bytes
28. SSRF via redirects or DNS rebinding
29. Zip slip during archive extraction
30. TOCTOU race conditions on file check-then-use

## 5. Cryptography and secrets (items 31–36)

31. Weak randomness for tokens, keys, or nonces
32. Secrets leaking into logs, error messages, or URLs
33. Missing HMAC verification or nonce reuse
34. Timing side-channel on secret comparison
35. Crypto primitive misuse (ECB mode, static IVs, unauthenticated encryption)
36. Crypto failure silently falling back to no-crypto

## 6. Business logic (items 37–42)

37. State machine allows skipping steps or reaching an invalid state
38. Race condition with financial/business impact (double-spend, double-approve)
39. Negative, zero, or overflowed quantities accepted
40. One operation bypasses a restriction enforced on a different operation for the same effect
41. Data from storage/config trusted as "already validated" without re-checking
42. Security posture undefined when config is missing or a feature flag is off

## 7. Feature abuse and data leakage (items 43–48)

43. Export/backup exposing data above the requesting user's access level
44. Import/restore bypassing normal write validation
45. Search/filter acting as an oracle for content the user can't directly access
46. Different error messages/timing for "doesn't exist" vs "no access" (enumeration)
47. Preview tokens scoped wider than the single item they're meant for
48. Server-fetched webhook/notification URL usable for SSRF

## 8. Chained vulnerabilities and trust boundaries (items 49–50)

49. Component A validates input; component B assumes a different guarantee (truncation, type coercion)
50. Token/API-key/capability gains broader authority after delegation or refresh

## 9. The wildcard move

Not a numbered item — a method for finding what the other 63 miss. Ask of the
codebase: what's the strangest code here and why does it exist; what looks
half-finished or bolted on; what API calls are possible but the frontend never
makes; is there anything reverted or commented-out in git history around auth;
what happens when two unrelated features get combined (OAuth + impersonation +
API keys, import + plugins + webhooks).

## 10. AI/LLM/agent-specific (items 51–64) — only when a model is in the loop

Applies only when a language model participates in a trust-sensitive decision:
chatbot, RAG pipeline, persistent agent memory, tool-calling loop, MCP
server/client, prompt built from untrusted input, or code that acts on model
output. If the project has none of this, all 14 items here are "not
applicable — no LLM/agent surface in this codebase," not "unverified."

Core rule for every item in this group: prompt injection *by itself* is not a
finding. There must be a code-level boundary failure — content reaching
another principal's context, invoking authority the requester lacks,
disclosing data they can't read, or driving a sink they can't reach directly.
A guardrail prompt is not a security boundary; only deterministic checks,
scoped authorization, isolation, and constrained credentials count.

51. Indirect injection via RAG document/file/email/issue that enters another principal's model context
52. Cross-tenant context bleed in conversation history, embeddings, or prompt caches
53. Persistent memory poisoned by attacker content, later shaping another user/task
54. Prompt role/provenance confusion — untrusted text impersonates a system message or tool result
55. Model-produced tool arguments reaching SQL/shell/file/URL sinks without handler-side validation
56. Excessive agency — agent uses a shared/service credential without re-checking the actual requester's permission
57. Action-confirmation binding failure — user approves one action, execution uses different arguments/resource/principal
58. Tool schema and dispatcher disagree on aliases, duplicate keys, or coercions
59. Unbounded delegated action loop — no per-request budget, authorization, or idempotency control
60. Sub-agent/MCP call inherits full session credentials instead of least authority
61. MCP server/tool identity confusion — calls routed by attacker-influenceable names rather than the authenticated connection
62. MCP-supplied metadata/schema trusted as policy instead of just guidance
63. Model output reaching an HTML/Markdown/command sink without the sink's required encoding
64. Assembled context containing credentials or another user's data, exposed via user-influenced output

## Notes for auditing this particular list

Unlike the other three bundled lists, most items here are *patterns to trace*,
not *features to grep for*. "SQL injection" isn't verified by finding a
parameterized-query call somewhere — it's verified by picking an actual
user-controlled input and following it to its actual sink, then checking every
sink of that kind, not just the first one found.

Overlaps heavily with `launch-security-checklist.md` (its items 1, 10, 11, 19,
25 — HSTS, CORS, cookies, TLS, secrets) and with the Security group in
`systems-checklist.md`/`default-checklist.md` (CORS, CSRF, XSS, SSRF, TLS). If
more than one of these lists gets audited in the same session, check baseline
exposures (group 1) and injection (group 2) once and reuse the citation across
lists instead of re-deriving it.

Group 10 is the one most likely to get a lazy "not applicable" — check for an
LLM integration honestly (an SDK import, an API key for a model provider, a
prompt-template file) before ruling it out, since "we don't have a chatbot" can
still mean an agent framework or MCP integration hiding a few files deep.

Items 43–48 (feature abuse) require thinking about the codebase's actual
features, not a generic scan — read the note in `ATTACK-CLASSES.md`'s wildcard
section: "look for bugs in the design, not only in the code."
