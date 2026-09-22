# Community findings checklist (16 items)

**Provenance:** distinct from every other bundled list — not from a vetted
single source like Cloudflare's `security-audit-skill`. These items came from
the user's own review of real practitioner discussions (~150 posts across
security-related community tags). Each item below reflects an actual
reported incident or pattern, not a document anyone here re-read directly, so
treat them the way you'd treat a tip from a colleague: plausible and worth
checking, but verify the specific mechanism against this project's actual
code before trusting it — the exact framework/library named in an item may
not match this project's stack at all, in which case it's legitimately "not
applicable," not a failed check.

Under the ~40 threshold but over ~15 — group by domain and investigate group
by group, per step 4's normal rule; subagents optional.

## Agents and AI tooling (items 1–6)

Applies only when the project has a persistent agent (memory/index files that
auto-load), MCP integration, or an agent that reads external content or
publishes autonomously.

1. **Secret leaked into an agent's own persistent memory** — a secret gets
   written into an index/memory file that auto-loads on every future session,
   and may even sync to a vector store. Distinct from
   `agent-security-checklist.md` item 53 (attacker-poisoned memory) — here the
   leak is accidental and the "attacker" is just normal accumulated state.
2. **Credential visible in an MCP tool's schema/description** — the model can
   see the actual credential value instead of the call being brokered via
   stdio env-injection or an HTTP two-hop pattern that keeps the real secret
   out of the model's context entirely.
3. **Content/output guard placed upstream of the actual consuming sink** — a
   filter runs before a step that later streams raw, unfiltered tool output
   directly to the user, while the model itself only ever saw the redacted
   version. The guard being *somewhere* in the pipeline isn't the same as
   being at the node that actually emits to the untrusted audience.
4. **Human-approval gate placed after the side effect, not before it** — e.g.
   a terminal command runs and *then* asks for confirmation, or confirmation
   arrives after the mutation is already committed. The gate has to block
   execution, not just report on it.
5. **An autonomous "publish" agent lacking a pre-send leak-scan gate** — an
   agent that posts directly to a public knowledge base, wiki, or docs site
   with no check for internal-only context (credentials, other users' data,
   draft content) leaking into the published result.
6. **Third-party agent skills/plugins installed with no vulnerability or
   permission review** — most agent tooling installs a skill/plugin on
   request with no scan step at all. Check whether this project (or its CI)
   has any gate before a new skill/plugin/MCP server gets tool access.

## Authentication and tokens (items 7–9)

7. **Refresh token minted with a short TTL at session start, with no
   mid-task refresh** — a long-running agent/background task can outlive the
   token's TTL and fail partway through, or worse, silently continue on a
   stale credential depending on how the failure is handled.
8. **API keys not treated like passwords** — stored and compared in
   plaintext instead of hashed at rest, verified on every request, and gated
   by the calling principal's actual privilege level (not just "is this key
   valid at all").
9. **A WAF/access-proxy (e.g. Cloudflare Access) protects the domain, but the
   origin server is still directly reachable** — an attacker who finds the
   origin IP bypasses the WAF/access layer entirely. Check whether the origin
   accepts direct connections or only from the proxy's IP range.

## CI and supply chain (item 10)

10. **A SHA-pinned GitHub Action is still mutable** — the Action's own
    internal downloads (a script it fetches, a binary it installs) aren't
    pinned by the same digest, so pinning the Action's SHA doesn't actually
    freeze everything it executes.

## Database (items 11–12)

Applies to projects using row-level security or raw SQL functions.

11. **A self-referential RLS (row-level security) policy causes infinite
    recursion** — a policy's own subquery hits the same table the policy
    protects, looping. Check policy definitions for a subquery against their
    own table without a bounding condition.
12. **A parameterized query is safe at the call site but reconcatenated
    downstream** — application code parameterizes correctly, but a database
    function it calls (e.g. via `dblink` or dynamic SQL inside a stored
    procedure) reassembles the value into a raw string before executing it.
    "We use parameterized queries" doesn't cover this path.

## Infrastructure (items 13–14)

13. **A rate limiter or connection cap enforced per-process behind a
    multi-replica load balancer** — the effective limit is the configured
    limit multiplied by the replica count, not the configured value itself,
    because each process/listener enforces its own independent counter.
14. **Client IP trusted directly from a header instead of configured from a
    fixed, known-trusted proxy source** — vulnerable to `X-Forwarded-For`
    spoofing when the app trusts whatever the header says rather than only
    accepting the IP handed off by the actual reverse proxy/CDN at a known
    hop.

## Secret leakage (items 15–16)

15. **A debug/logging library configured to dump local variables into
    tracebacks or logs** (e.g. `rich(show_locals=True)`, verbose `structlog`
    configs) — prints the contents of every local variable in a crashing
    frame, including ones that only ever held a secret in memory and were
    never meant to be logged.
16. **Unattended artifact/release signing silently skips signing when the
    signing credential is missing**, instead of failing the build. Produces
    an unsigned artifact that looks like a normal release output.

## Notes for auditing this particular list

Items 13 and 14 describe exactly the kind of infrastructure pattern this
project's own deployment (behind a CDN/reverse proxy, potentially
multi-replica) should be checked against directly — don't treat these as
generic/theoretical without actually reading the deployment config.

Two items from the original research were left out as too narrow to bundle
generically: an AWS-WAF-specific 8KB body-size default, and a
Supabase/PostgREST-specific JWT-signing-key migration that silently falls
back RLS to anonymous. Ask the user whether either applies before assuming
either belongs in this project's stack — add them as one-off checks only if
the stack actually matches.
