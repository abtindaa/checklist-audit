# Extended attack-domain checklist (90 items)

Source: the seven remaining domain files from Cloudflare's
[security-audit-skill](https://github.com/cloudflare/security-audit-skill)
(`WEB-PROTOCOL-AND-AUTH.md`, `CLIENT-SIDE.md`, `CLOUD-AND-DEPLOYMENT.md`,
`SUPPLY-CHAIN-AND-RELEASE.md`, `RESOURCE-EXHAUSTION-AND-AVAILABILITY.md`,
`DATA-ISOLATION-AND-LIFECYCLE.md`, `PROTOCOLS-RPC-AND-MESSAGING.md`,
`DESKTOP-MOBILE-AND-LOCAL-IPC.md`). Same character as
`agent-security-checklist.md`: these are attack *patterns* to trace through
actual code paths, not features to grep for.

`MEMORY-SAFETY-AND-BINARY.md` (the eighth file in that repo) is deliberately
excluded — it covers C/C++/Rust-unsafe/kernel/parser memory safety, which is
out of scope for the kind of application-layer projects this skill audits. If
a future audit target is a native binary or kernel module, fetch that file
directly rather than bolting it onto this one.

At 90 items this is over the ~40 threshold — parallel-subagent branch of
step 4, one agent per domain below. **Unlike `agent-security-checklist.md`,
several of these domains only apply given a specific architecture.** Check
applicability honestly before assigning an agent to a domain — don't skip a
domain because it sounds unfamiliar, and don't run a domain that plainly
doesn't fit (e.g. group 7 needs an actual message broker or RPC framework in
the dependency tree, not just any HTTP API).

## 1. Web protocol and auth (items 1–21) — applies to virtually any web app/API

Any project with sessions, JWT, OAuth/OIDC, SAML, API keys, or mTLS.

1. Front end/back end disagree on request length (duplicate `Content-Length`,
   `Transfer-Encoding`) — request smuggling
2. Web cache poisoning — a request value affects the response but isn't in the cache key
3. Cache deception — a private dynamic path served as a public cacheable asset
4. Untrusted `Host`/`Forwarded`/`X-Forwarded-*` used for URLs, tenant routing, or reset links
5. Untrusted data reaching `Location`, `Set-Cookie`, or CSP response headers
6. CSRF on a state-changing endpoint with no effective token/SameSite/Origin check
7. Session ID not rotated on login, account switch, MFA completion, or privilege change
8. Session/token still valid after logout, password change, or revocation
9. Cookie `Domain`/`Path` broader than needed, or usable over insecure transport
10. JWT: missing signature verification, algorithm confusion, or missing `exp`/`nbf`/`aud`/`iss` checks
11. OAuth/OIDC: `redirect_uri` not exactly validated, `state` missing/unbound, no PKCE
12. SAML: signature validated on a different element than the one used as identity
13. MFA: a factor can be enrolled/replaced without fresh first-factor auth; disabled factors still work
14. Step-up auth binds to the wrong session/action, or an alternate route skips the check
15. WebAuthn/passkey: missing challenge/RP-ID/origin binding at registration or auth
16. Account linking allowed without a current authenticated session + verified new-identity ownership
17. Password reset: predictable, non-expiring, or reusable token; prior tokens not invalidated
18. API key: scope not enforced server-side, or request params override the bound scope
19. API key exposed in a client bundle, URL, log, or error response
20. mTLS identity headers trusted from any peer without proxy stripping/verification
21. Expired/revoked/missing certs silently fall back to bearer-only or anonymous access

## 2. Client-side / browser (items 22–34) — applies to any SPA/frontend

22. DOM XSS via `location`/`document.referrer`/`window.name`/message data into `innerHTML` or an eval-like sink
23. DOM clobbering — attacker `id`/`name` attributes shadow a trusted global
24. Prototype pollution reaching an actual gadget, not just `JSON.parse` alone
25. `postMessage` handler missing an exact origin (and source) check
26. WebSocket upgrade accepts ambient cookies with no `Origin` check
27. CORS reflects `Origin` combined with `Allow-Credentials` on a sensitive response
28. Service worker registration/script controllable by attacker-influenceable content
29. Service worker cache serves a personalized response after account switch or logout
30. Sensitive token/data left in `localStorage`/`IndexedDB` readable by less-trusted same-origin code
31. `BroadcastChannel`/`storage` events carry identity across tabs without session binding
32. XS-Leaks — cross-origin state guessable via timing, error events, or response size
33. Clickjacking — a state-changing action lacks effective `frame-ancestors`/`X-Frame-Options`
34. Client-controlled navigation to `javascript:`/`data:` URLs, or reverse tabnabbing via `window.opener`

## 3. Cloud and deployment (items 35–47) — applies to any cloud-deployed project

35. Workload/pod/function identity can act beyond its role when untrusted input selects the target
36. Cross-account/cross-tenant role assumption not bound to the expected source
37. App trusts caller-supplied identity headers without verifying they came from the platform
38. Admin/debug/metrics/internal API reachable from a lower-trust network
39. Backend trusts forwarded identity/mTLS headers from peers outside the intended mesh/ingress
40. SSRF reaching cloud instance metadata or an internal API with workload credentials
41. Privileged container capabilities or host mounts reachable by a lower-trust workload
42. One deployment path enforces image/secret/privilege policy while another (job, restore, legacy) doesn't
43. Dev-mode config, feature flags, or overlays disable auth/transport security in a live environment
44. Secrets leak into logs, process args, or volumes shared across workload boundaries
45. Stale credentials remain active after a failed rotation/revocation
46. Object storage / signed URL not bound to principal, exact object, audience, and expiry
47. An event source (webhook, queue message) trusted as identity without signature/provenance verification

## 4. Supply chain and release (items 48–54) — applies to any project with CI/CD and dependencies

48. Dependency resolution can be redirected to an unintended registry/namespace/mirror
49. CI workflow runs untrusted PR/fork code with access to protected secrets
50. Attacker-controlled branch name/commit message/issue text reaches a shell command in CI
51. A lower-trust CI job's cache or artifact is consumed and executed by a higher-trust job
52. Release/signature verification refers to a mutable tag/name instead of an immutable digest
53. An updater verifies payload signature but not version/platform/channel/rollback binding
54. A plugin/extension gains host authority beyond its declared scope

## 5. Resource exhaustion and availability (items 55–64) — applies to any endpoint taking user input

55. Regex or parsing on user input vulnerable to catastrophic backtracking (ReDoS)
56. Decompression bomb — compressed/nested input expands far beyond the checked size
57. An unbounded DB query (no pagination/depth limit) triggerable by a small request
58. Unbounded buffering — uploads, sessions, or cache keys accumulate with no per-item/aggregate cap
59. Cancelled or timed-out requests leave DB/worker/network operations still running
60. Expensive work (crypto, parsing, external calls) happens before authentication or a rate gate
61. Quota accounting keyed on a spoofable dimension (IP, header) lets one principal escape its budget
62. Untrusted input reaches a process-killing panic/unhandled exception in a shared worker
63. Retries on failure have no jitter/ceiling — can synchronize into a retry storm
64. One malformed message blocks an entire shared queue or partition (head-of-line blocking)

## 6. Data isolation and lifecycle (items 65–74) — applies to multi-tenant/multi-user projects

65. A tenant/owner field exists but isn't enforced on every path (background jobs, admin, bulk ops)
66. Cache/search-index keys omit tenant, allowing cross-tenant key collision
67. ORM default scope bypassed by a raw query, unscoped client, join, or aggregate
68. A signed URL/object link is valid beyond its intended scope or after the underlying ACL changes
69. A search/cache/index copy isn't invalidated when the source record's ACL changes
70. Logs/analytics/traces receive private data or credentials with broader retention or access
71. Export/backup includes other tenants' or soft-deleted data without per-item authorization
72. Import/restore bypasses ACL/validation and can overwrite another tenant's data
73. A soft-deleted record is still returned via search, relation traversal, or a background job
74. Revoked membership/ACL change doesn't invalidate existing sessions, caches, or queued jobs

## 7. Protocols, RPC, and messaging (items 75–82) — only if gRPC/GraphQL/queues/brokers/webhooks are in use

75. "Internal network position" treated as authentication with no real peer-identity check
76. An auth interceptor covers unary calls but not streaming/reflection/health/gateway-transcoded routes
77. Envelope/routing metadata trusted over a conflicting tenant/subject field in the message body
78. Reused or predictable correlation IDs let one caller's response satisfy another's pending request
79. A publisher/subscriber can select another tenant's topic, queue, or consumer group
80. Dead-letter/retry/diagnostic queues expose secrets or cross-tenant payloads
81. A message body can self-declare as an admin/privileged event with no authenticated-producer check
82. Retry/redelivery repeats a non-idempotent side effect (double-charge, double-send)

## 8. Desktop, mobile, and local IPC (items 83–90) — only for desktop/mobile/native apps

83. A deep link/custom URI scheme triggers a state change or login with no session/one-time binding
84. A webview native bridge is reachable from an attacker-controlled/remote frame, not just packaged content
85. A native bridge exposes arbitrary file/command/IPC methods with no allowlist or per-call authorization
86. A local Unix socket/named pipe/D-Bus/Binder endpoint accepts any peer with no OS-level credential check
87. An exported Android component (activity/service/receiver/provider) is callable by other apps unintentionally
88. A privileged helper (sudo/polkit/UAC) trusts a caller-supplied path/command with no independent validation
89. TOCTOU on a privileged file operation — checked path replaced by a symlink before use
90. A credential/token file is readable by another app or OS user with less authority

## Notes for auditing this particular list

Domains 1–5 are broadly applicable and should usually all run together for a
typical web/cloud project. Domains 6–8 are architecture-gated — verify the
actual dependency tree and deployment target before assigning an agent, and
rule out a whole domain as "not applicable" with the specific fact (no message
broker in `package.json`/`go.mod`, no mobile/desktop build target) rather than
skipping it silently.

Heavy overlap with the other bundled lists on baseline items — CSRF (item 6
here, item 2 in `launch-security-checklist.md`, item 23 in
`agent-security-checklist.md`), JWT rotation (item 10 here, item 27 in
`launch-security-checklist.md`), SSRF (item 40 here, item 28 in
`agent-security-checklist.md`, item 48 in `launch-security-checklist.md`). When
running this list alongside the others in the same session, check each
overlapping item once and reuse the citation instead of re-deriving it.

Several items require reading actual deployment/CI configuration, not just
application source — item 43 (dev-mode config in a live environment) and
item 49 (CI running untrusted PR code) need the real workflow YAML and the
rendered config for each maintained environment, not just the base file in the
repo. Where that's not observable, these belong in "unverified" with the
specific missing fact (which environment's config, which workflow's trigger
type), not a guess in either direction.
