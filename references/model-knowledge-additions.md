# Model-knowledge additions (8 items)

**Provenance warning:** unlike the other five bundled lists, these items are
not sourced from a document anyone read — they came from the model's own
training knowledge when asked "what else is missing," the same kind of
unsourced claim this whole skill exists to distrust. Kept separate, small, and
explicitly labeled rather than folded into `agent-security-checklist.md` or
`launch-security-checklist.md`, so a future audit doesn't inherit false
confidence from an item that merely *sounds* plausible.

Treat these with more skepticism than the other lists during step 4: they are
starting hypotheses to investigate, not confirmed attack classes from a
vetted source. If investigation shows an item doesn't really apply to how this
project's stack works, that's a legitimate and expected outcome, not a
verification failure.

Under the ~15-item direct-investigation threshold — no grouping or subagents
needed.

1. **Dependency slopsquatting** — a package name was hallucinated by an LLM
   (during dependency selection, in a tutorial the team copied from, or by a
   coding agent) and someone installed it as written. Check whether any
   dependency in the lockfile is unusually obscure, was added recently without
   a clear reason in commit history, or doesn't match what a human would
   plausibly have searched for. This is a supply-chain risk distinct from
   `extended-attack-domains.md` item 48 (registry/namespace redirection) —
   here the package name itself is fake, not the resolution path.

2. **Coding agent given unsandboxed write access** — an AI coding agent (this
   one included) running with shell/filesystem access that reaches production
   credentials, production data, or unreviewed deploy/push capability. Check
   for CI secrets or production `.env` values reachable from a dev sandbox or
   agent-accessible environment, and whether agent-initiated commits/deploys
   require human review before taking effect.

3. **Mass assignment / over-posting** — a create/update endpoint binds the
   entire request body to a model or ORM record without an explicit
   allow-list, letting a client set fields it shouldn't (`role`, `isAdmin`,
   `balance`, `verified`) by simply including them in the payload. Related to
   `agent-security-checklist.md` item 24 (a body field overriding a
   permission check) but broader — this applies even where no permission
   check exists to override, because the field was never meant to be
   client-settable at all.

4. **GraphQL introspection and query cost** — introspection left enabled in
   production (lets an attacker enumerate the entire schema, including
   fields never exposed in the client), or no query depth/complexity limit,
   letting a single request construct deeply nested or fan-out queries.
   Related to `extended-attack-domains.md` item 57 (unbounded DB query) but
   specific enough to GraphQL's shape that it's easy to miss when auditing
   REST-style rate limits and pagination instead.

5. **Weak or missing password hashing algorithm** — passwords hashed with a
   fast general-purpose hash (MD5, SHA1, SHA256 with no work factor) or, worse,
   stored reversibly-encrypted or in plaintext, instead of a slow
   memory-hard KDF (bcrypt, argon2, scrypt) with a real cost parameter. Distinct
   from `agent-security-checklist.md`'s crypto group (items 31–36), which
   covers primitive misuse generically but never names password storage
   specifically — easy to assume "we have crypto covered" from that group
   without ever checking this exact line.

6. **No responsible-disclosure path** — no `security.txt` (`/.well-known/security.txt`),
   no published security contact/email, and no documented policy for how an
   outside researcher reports a vulnerability. Not a code defect, but a real
   and commonly-missing operational control — check for the file, a
   `SECURITY.md`, or a mention in the README before concluding it's absent.

7. **Model-generated code executed without sandboxing or review** — a
   code-interpreter-style feature (or any tool that takes LLM output and runs
   it as code) executes that code with the same privileges as the host
   process, no container/VM isolation, and no human review gate. Distinct from
   `model-knowledge-additions.md` item 2 above (an AI *coding agent* like this
   one reaching production) — this is about a *product feature* that lets an
   end user's prompt result in server-side code execution.

8. **No breached-password check at signup/password-change** — a new or
   changed password isn't checked against a known-breach corpus (e.g. the
   Pwned Passwords range API or an equivalent offline list), so users can set
   credentials already public in a prior breach. Distinct from item 5 above
   (which is about how the password is stored) — this is about what password
   is accepted in the first place.
