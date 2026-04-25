# RISKS.md - Caravansary

*The things that can kill the product, and the way we route around them without compromising the developer experience.*

April 2026. Pre-alpha planning doc. Not legal advice.

---

## 0. The anchor

The product promise does not move:

> **One signup. One key. Working infrastructure. No dashboard pilgrimage.**

Risk mitigation is not allowed to become onboarding tax. If a mitigation asks the user to read vendor docs, create a vendor account before they can try us, compare plans, configure IAM, or paste five secrets into a wizard, it is not a mitigation. It is product failure wearing a safety vest.

The right pattern is:

1. Keep the first experience seamless.
2. Move complexity into the gateway, policy engine, partner layer, or delayed just-in-time flow.
3. Only surface friction at the exact moment the real world requires it.
4. Explain the friction in the user's language, not the vendor's language.

The user should never learn our legal architecture. They should feel that infrastructure just works.

---

## 1. Risk register

| # | Risk | Severity | Solution that preserves the developer experience |
|---|---|---|---|
| 1 | Free-tier quota pooling is treated as circumvention | Critical | Do not depend on multiple master accounts or quota sharding. Use one honest vendor account per provider, enforce hard per-user caps, route across truly distinct providers, and pursue partner/reseller programs before scaling beyond public quotas. |
| 2 | A vendor bans master-key resale | Critical | Keep BYOK and partner provisioning behind the same Caravansary API shape. If a provider flips red, disable only that backend route; the user keeps the same key, endpoint, SDK, and response schema. |
| 3 | Payments cannot be fully silent because Stripe requires onboarding/KYC | Critical | Let `/v1/payment/checkout` work in simulation mode immediately. At first real-money charge, return a hosted "activate payouts" link that completes Stripe onboarding and then resumes the same API call. |
| 4 | "One invoice" is impossible while users BYOK vendors | High | Promise one Caravansary invoice for the platform fee and one unified usage ledger. Defer true consolidated vendor billing until reseller/MoR agreements exist. In the UI, call this "one place to understand spend" until it is legally one invoice. |
| 5 | No-card free tier attracts abuse | Critical | Keep no-card onboarding, but add invisible abuse controls: OAuth reputation, per-key velocity limits, device/IP risk scoring, endpoint-specific caps, content scanning where appropriate, and instant kill switches. |
| 6 | Email endpoint gets abused for spam | Critical | Require domain verification for general outbound email. Before verification, allow only tightly limited system-style messages from a Caravansary-controlled subdomain. Maintain suppression lists, bounce handling, complaint handling, and per-tenant reputation. |
| 7 | Hosting endpoint runs malicious or expensive user code | Critical | Do not launch arbitrary container hosting as the first public free endpoint. Start with constrained deploy targets: static assets, Workers-style request handlers, or template-generated services with CPU/memory/network caps. Add full container deploy only after sandboxing and abuse automation are real. |
| 8 | LLM master-key traffic creates safety and liability exposure | High | Put moderation, rate limits, logging minimization, and vendor-policy flowdown in the gateway. Route risky use cases to BYOK or deny them. Keep the user API unchanged; change only the backend route or error reason. |
| 9 | Cross-tenant caching leaks private prompts or behavior | High | Scope caches per tenant by default. Allow cross-tenant caching only for public demo prompts, static docs examples, or explicitly marked shared artifacts. Never let the cache optimization become a privacy surprise. |
| 10 | BYOK custody creates secret-management obligations | High | Treat BYOK as a paid graduation feature backed by envelope encryption, scoped decrypt paths, audit logs, rotation, redaction, and emergency purge. The user pastes one key once; the operational burden stays with us. |
| 11 | Downstream providers have incompatible schemas and failure modes | Medium | Normalize responses aggressively at the gateway. Maintain contract tests per endpoint so providers can be swapped without user code changes. If fidelity cannot be preserved, expose a Caravansary capability flag, not a vendor flag. |
| 12 | Fallback routing changes output quality unexpectedly | Medium | Define quality classes per endpoint. Fallback only within the same class unless the user explicitly opts into "best effort." The user should get reliability without mysterious degradation. |
| 13 | Region, privacy, and DPA obligations arrive before the product is ready | High | Publish a subprocessor list, retention policy, deletion policy, and DPA before paid launch. For free tier, keep data retention short and transparent. Add EU region pinning as a paid graduation feature without changing the API path. |
| 14 | Partner programs take longer than expected | Medium | Ship Phase 0 on modes that do not require contracts: self-hosted OSS, permitted master-key use, and BYOK. Design provider adapters so partner provisioning can replace master-key routes later without changing the public API. |
| 15 | Credit programs cannot be one-click applied through us | Medium | Start as a curated eligibility and expiry tracker. Pre-fill what we can, deep-link where we must, and pursue referral/partner flows later. The user still gets one place to discover and manage credits. |
| 16 | Competitors copy the landing-page promise | High | The moat is not the slogan; it is the adapter network, abuse controls, legal routing matrix, normalized schemas, and accumulated user trust. Keep the UX simple while making the backend hard to replicate. |
| 17 | The product drifts into enterprise workflow software | Medium | Keep enterprise features behind graduation: teams, audit, SSO, custom DPA, region pinning. None of them enters the first-run path. The first-run path remains sign in, copy key, call endpoint. |
| 18 | The support burden overwhelms a low-price product | Medium | Build support into the product: precise error messages, request IDs, replayable traces, status page, and self-serve diagnostics. Paid users get human support; free users get excellent tooling. |
| 19 | "Every vendor" creates too much surface area | High | Launch with fewer categories but make them complete. Prefer five endpoints that feel magical over nine endpoints that leak vendor complexity. Add categories only when they pass the two-screen test and the abuse/legal review. |
| 20 | Legal caution erodes the core value | Critical | Legal constraints decide backend mode, not user experience. If master-key is unsafe, use BYOK or partner provisioning behind the same Caravansary endpoint. If a vendor requires a user-facing ceremony, delay it until first use and make it resumable. |

---

## 2. Non-negotiable mitigation patterns

These are the patterns we should reuse whenever a new risk appears.

### 2.1 Just-in-time friction

Friction appears only when the user has asked for the thing that requires it.

Bad:
- "Before you get a Caravansary key, connect Stripe, Cloudflare, Resend, and OpenAI."

Good:
- "Here is your key."
- Later, when the first real payment is created: "Activate payouts to accept live money."
- After activation, the original `/v1/payment/checkout` call works with the same request shape.

### 2.2 Simulation before activation

Endpoints that require legal or financial activation should still be callable on day one.

For example:
- `/v1/payment/checkout` returns a test checkout link until payouts are activated.
- `/v1/email/send` can send limited test/dev mail before domain verification.
- `/v1/host/deploy` can deploy a constrained preview before full custom hosting is enabled.

The user experiences momentum first and paperwork later.

### 2.3 Backend-mode substitution

The user should not care whether a call is served by:

- master-key route,
- BYOK route,
- partner-provisioned subaccount,
- self-hosted OSS,
- or a fallback provider.

The gateway owns that decision. The public API stays stable.

### 2.4 Hard caps over surprise bills

Free tier failure mode is "you hit today's limit," never "you owe us money."

Paid tier failure mode is "your configured billing method or vendor key needs attention," not "we silently ran up a vendor bill."

No-card free tier remains non-negotiable.

### 2.5 Tenant isolation by default

Every optimization starts tenant-scoped:

- cache,
- logs,
- rate limits,
- abuse scores,
- provider route history,
- email reputation,
- storage paths.

Cross-tenant sharing requires an explicit product reason and a privacy review.

### 2.6 Kill switches that do not kill the product

Every provider route and endpoint category needs an independent kill switch.

If Groq changes terms, disable Groq routing.
If email abuse spikes, freeze outbound email for risky tenants.
If hosting is attacked, pause deploys.

The Caravansary key, account, docs, and other endpoints remain alive.

---

## 3. Phase guidance

### Phase 0: prove magic, not breadth

Ship the smallest set of endpoints that can honestly deliver the emotional promise:

1. LLM chat through safe free/provider routes.
2. Transactional email with strict verification and dev-mode fallback.
3. File/object storage.
4. Error or event capture through self-hosted infrastructure.
5. Payment checkout in test/simulation mode, with live activation just-in-time.

Do not launch arbitrary container hosting, broad DNS mutation, or unconstrained email until abuse controls are already boring.

### Phase 1: make graduation invisible

Add BYOK, provider preferences, paid limits, and activation flows without changing the SDK or endpoint shape.

The paid user should feel like the same product got stronger, not like they migrated to a different product.

### Phase 2+: earn true consolidation

Only after partner agreements, MoR coverage, and subaccount provisioning exist should we claim:

- one consolidated vendor invoice,
- managed downstream accounts,
- pooled startup credits,
- region guarantees,
- enterprise procurement readiness.

Until then, the honest promise is:

> One key, one API, one place to understand your stack.

That is still valuable. It is also defensible.

---

## 4. Review checklist for every new endpoint

Before adding a provider or endpoint, answer:

1. Can a new user call it with the existing Caravansary key?
2. If activation is required, can we delay it until first meaningful use?
3. Does the fallback route preserve the same response contract?
4. What is the abuse mode?
5. What is the hard free-tier cap?
6. What data do we log, and for how long?
7. Can we disable this route without disabling the whole account?
8. Does this create a new vendor relationship for the user?
9. If yes, can we hide it until it is legally required?
10. Does this strengthen or weaken "sign in, copy key, ship"?

If the answer to #10 is "weaken," the endpoint is not ready.
