# Vibe-coding stack checklist (18 items)

The failure modes most specific to projects built quickly with AI app
builders and scaffolds — Supabase/Firebase backends, Next.js/Vite frontends,
keys wired straight into the client. Each item names a concrete thing to grep
for. Two of them are well-documented public incidents: Lovable-built apps
exposing whole databases through missing Supabase RLS (CVE-2025-48757), and
the Next.js middleware authorization bypass via `x-middleware-subrequest`
(CVE-2025-29927).

Most groups are **stack-gated**. Detect the stack from the dependency files
and imports (`@supabase/supabase-js`, `firebase`, `next`, `vite`,
`react-scripts`, `expo`) before ruling a group in or out, and give that
detection as the architectural reason when a group is N/A. A project with no
BaaS and a server-rendered backend will legitimately mark several of these
N/A.

Use prefix `V-`. At 18 items this list is small on its own, but it runs as
part of the default security audit alongside the 38 + 64 lists.

## Backend-as-a-service: Supabase (V-1 – V-3)

1. **Row-level security enabled on every exposed table.** Compare the tables
   created in migrations / `supabase/migrations/*.sql` with the
   `enable row level security` statements. Any table in an API-exposed schema
   (usually `public`) without RLS is readable and writable by anyone holding
   the anon key — which ships in the frontend by design.
2. **RLS policies actually restrict.** Look for policies with `using (true)`
   or `with check (true)` on tables holding user data, and for policies not
   scoped to `auth.uid()` or a membership check. RLS that is "on" with an
   allow-all policy protects nothing.
3. **The `service_role` key never reaches the client.** It bypasses RLS
   entirely. Grep the frontend source and every public env var for it; it may
   only appear in server-side code, edge functions, or CI secrets.

## Backend-as-a-service: Firebase (V-4)

4. **Security rules are not open.** Check `firestore.rules`, `storage.rules`,
   and `database.rules.json` for `allow read, write: if true`,
   `".read": true` / `".write": true`, and test-mode rules
   (`request.time < timestamp.date(...)`) that silently expire into
   open-or-closed states.

## Secrets in the client bundle (V-5 – V-7)

5. **No secret behind a public env prefix.** Anything prefixed
   `NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, `EXPO_PUBLIC_` or `PUBLIC_` is
   compiled into the browser bundle. Check `.env*` files, deploy config, and
   code for secret-shaped values under those names (`sk_live_`, `sk-`,
   `service_role`, provider API keys). Match on the variable name and the
   value's prefix only, with a count or `grep -l`, never by printing the
   value — see the ground rules in `SKILL.md`.
6. **Third-party paid APIs are called server-side only.** A browser calling
   OpenAI/Anthropic/Stripe-secret endpoints directly means the key is in the
   bundle, whatever the variable is named. Grep client code for provider
   hostnames and SDK constructors.
7. **Production source maps aren't published by accident.** Check
   `productionBrowserSourceMaps` (Next.js), `build.sourcemap` (Vite), and the
   deployed asset folder. Published maps hand out the full original source,
   including any comments and hardcoded values in it.

## Authorization lives on the server (V-8 – V-10)

8. **Every protected action is checked server-side.** Frontend route guards,
   hidden buttons, and disabled UI are not authorization. For each mutating
   API route, server action, or edge function, confirm it verifies the session
   itself — AI-generated apps very often protect the page and forget the
   endpoint behind it.
9. **Next.js: auth isn't enforced only in middleware, and the version is
   patched.** Server actions are public POST endpoints and route handlers are
   separate entry points; each needs its own check. If middleware is the only
   gate, confirm `next` is at a version patched for CVE-2025-29927.
10. **Identity and role come from the session, never the client.** Look for
    `role`, `isAdmin`, `user_id`, or `owner_id` read from `localStorage`, a
    cookie the client writes, query params, or the request body, then trusted
    for an authorization decision.

## Data exposure (V-11 – V-12)

11. **Responses return selected fields, not whole rows.** `select *`,
    returning raw ORM objects, or spreading a DB row into JSON leaks password
    hashes, other users' emails, internal flags, and tokens. Check list and
    detail endpoints and the BaaS client queries the frontend makes directly.
12. **User-upload storage isn't public.** Supabase Storage public buckets,
    S3 `public-read` ACLs, and Firebase Storage rules — private documents
    (IDs, invoices, attachments) must be served through signed or checked
    access, not guessable public URLs.

## Scaffolding left behind (V-13 – V-14)

13. **Seed/demo accounts and default credentials don't exist in
    production.** AI scaffolds commonly create `admin@example.com` /
    `password123` style users, seed scripts wired into deploy, or a
    "make me admin" route. Check seed files and whether they run outside
    development.
14. **Serverless/edge functions require auth where they should.** Supabase
    edge functions deployed with `--no-verify-jwt`, Vercel/Netlify functions
    with no session check, and webhook endpoints with no signature check are
    public entry points to backend authority.

## Mobile / Expo (V-15 – V-18) — only if the project ships an Expo or React Native app

Detect `expo`, `expo-updates`, `react-native` in the dependency file; with none
of them, this whole group is N/A. Items 15–18 came from a real audit of an
Expo + FastAPI project.

15. **OTA updates are code-signed, and the signing key is protected.** Check
    `app.json`/`app.config.*` for `updates.codeSigningCertificate` and
    `codeSigningMetadata`. Without signing, whoever controls the update
    server or its DNS can push code to every install. Then confirm the private
    key is encrypted, outside the repo, and unreadable by the agent (see E-93).
16. **Session expiry behaves like sign-out.** Compare the 401/expiry handler
    with the explicit sign-out handler. Both must clear the token *and* cached
    API data, profile, and offline stores (AsyncStorage, SecureStore, SQLite,
    query caches). A handler that clears only the token leaves the previous
    user's data on the device for the next person to open.
17. **Generated native directories aren't stale.** A gitignored `android/` or
    `ios/` prebuild can lag the config plugins — `allowBackup=true`,
    `usesCleartextTraffic`, no network security config or certificate pinning —
    while cloud builds regenerate it correctly, so a *local* release build ships
    without the protections. Compare the generated manifest/plist with what the
    config plugins produce, or regenerate it.
18. **The documented OTA rollback really rolls back.** An update client can
    keep a newer update over an older one, so a runbook's "rollback" step may
    do nothing for devices that already updated. Test it on an updated device,
    or document the fallback that does work — republishing the last good bundle
    as a *new* update. Don't assume the incident procedure works.

## Notes for auditing this particular list

Overlaps: V-5/V-6 with `launch-security` L-25 (secrets in git) and
`agent-security` A-1 (hardcoded secrets) — those ask whether a secret is in
the *repo*; these ask whether it reaches the *browser*, which can be true even
when the repo is clean (a secret set in the hosting dashboard under a public
prefix). V-8/V-10 overlap with A-23/A-25 (access control); reuse the citation
when both run.

Build output is evidence too. If a production build or deployed bundle is
available locally, grepping it for key prefixes settles V-5/V-6 more
reliably than reading source. Don't fetch a live deployment to do this —
see the audit ground rules in `SKILL.md`.
