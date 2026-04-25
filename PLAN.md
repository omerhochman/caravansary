# PLAN.md — Caravansary

*One signup. Every vendor. The walled compound for your dev stack.*

April 2026. Pre-alpha planning doc. Subject to change as Phase 0 lands.

---

## Reading note — what this product is and isn't

This is **not** an LLM gateway. It is **not** an AI product. It is **one API key replacing twenty** across every category of developer infrastructure a side project needs: domain, hosting, file storage, transactional email, error tracking, product analytics, payments, *and* LLMs. LLMs are one row in the endpoint table. Nothing in the value proposition depends on the LLM endpoint existing.

If, while reading this document, LLM examples start to feel disproportionate, that is a documentation bug, not a product fact. The reason LLM examples recur is that the LLM ecosystem currently has the largest number of distinct vendors per category (six in `LEGAL.md`), so per-vendor scoping naturally produces more LLM paragraphs. Discount accordingly. The product comparison tests, success metrics, and architectural decisions in this document apply equally to email, storage, DNS, payments, and the rest.

If you ever find yourself describing Caravansary by leading with "an LLM…", stop and reread this section.

---

## 0. The thesis (read this first, then read it again)

The product is **the absence of signup forms.**

A developer arrives. They click "Sign in with Google" or "Sign in with GitHub." They land on a page that already has a working API key on it. Copy. Or delete. That is the entire onboarding.

Behind that single key is **every developer-infrastructure category they will ever need for a side project, ready to call:** transactional email, file storage, DNS and a public subdomain, error tracking, product analytics, payments, container hosting, queues, feature flags, LLMs and embeddings. The user does not pick a provider. The user does not pick a region. The user does not name the key. The user does not set rate limits. The user does not configure billing. The user does not verify email. The user does not see a wizard.

The user gets a key and an endpoint and ships.

This document repeats that thesis many times because every architectural choice that follows is downstream of it. The seamlessness *is* the product. Everything else — the pricing, the gateway, the legal posture — is a consequence of refusing to ship anything that breaks the seamlessness.

The product comparison test is simple: **count the screens between the homepage and a working `curl`.** Two — the OAuth round-trip and the key page. Anything that adds a third screen is wrong.

---

## 1. The shape of the product

### 1.1 The onboarding (the entire thing)

```
caravansary.dev
   │
   ▼
[Sign in with Google]  [Sign in with GitHub]
   │
   ▼ (OAuth round-trip)
   │
   ▼

   Your key:   csk_live_a4f7b3c2…   [copy] [delete]
   Endpoint:   https://api.caravansary.dev/v1
   Free tier:  $5.00 of compute and inference per day.

   Try it:
   ▸ curl https://api.caravansary.dev/v1/llm/chat …
   ▸ curl https://api.caravansary.dev/v1/email/send …
   ▸ curl https://api.caravansary.dev/v1/payment/checkout …
```

That is the whole UI for a free-tier user. There is no settings page. There is no project list. There is no "team" tab. There is no model selector. The five example `curl`s are the documentation. Click any of them, get a working snippet in your language of choice.

The user can:
- Copy the key.
- Delete the key (which immediately mints a new one).
- Click any example to see the docs for that endpoint.

That's it.

### 1.2 The endpoints (Phase 1)

One key. One base URL. Categories, not vendors.

| Path | What it does | Provider(s) we route to |
|---|---|---|
| `POST /v1/dns/record` | Create/edit a DNS record (and auto-issue a `<key>.caravansary.app` subdomain on first call) | Cloudflare DNS on our zone |
| `POST /v1/email/send` | Transactional email | AWS SES (per-tenant verified domain or Caravansary subdomain) → Resend / Postmark on BYOK |
| `POST /v1/file/put` | Object storage | Cloudflare R2 |
| `POST /v1/error/track` | Capture an error | Self-hosted GlitchTip → Sentry |
| `POST /v1/event/track` | Product analytics event | Self-hosted PostHog OSS |
| `POST /v1/payment/checkout` | Hosted checkout link | Immediate test checkout; live money requires just-in-time Stripe Connect activation |
| `POST /v1/host/deploy` | Deploy a container | Fly.io → Cloudflare Workers (auto-pick by image type) |
| `POST /v1/llm/chat` | Chat completion, OpenAI-compatible | Gemini Flash → Groq → DeepSeek → OpenRouter free → (fallback) Anthropic |
| `POST /v1/llm/embed` | Text embeddings | Cloudflare Workers AI → Voyage → OpenAI |
| `caravansary` MCP server | Agent tool surface over the same categories | Same Caravansary router; no vendor keys exposed |

The user calls a category. Caravansary picks the vendor. The user never knows which one. The output is normalized — same schema regardless of which vendor served the request.

The MCP server is not a separate product with separate credentials. It is the agent-native face of the same Caravansary account: one Caravansary key gives an agent the same nine categories the SDK has, without handing the agent any vendor credentials.

### 1.3 The graduation path (Phase 2+)

When a user grows out of the free tier, they hit one button. There are no plans to compare. The button does three things:
1. Opens a Stripe-hosted upgrade flow.
2. Reveals a small "Connected Providers" vault — *now* they can pin "always Anthropic for `/llm/chat`" or "always Postmark for `/email/send`" or connect their own OpenAI account/key (BYOK at zero markup).
3. Lifts daily rate limits and removes Caravansary's per-request margin entirely (paying customers pay platform-flat, not per-call).

The graduation is itself an act of seamlessness — they keep the same Caravansary key, the same endpoint, the same SDK. Nothing in their project receives vendor keys and nothing in their code changes.

### 1.4 What is *not* on the free tier homepage (deliberately)

These are conscious omissions. Each one would be reasonable in a different product. None ship in Phase 1.

- No "create a new project" button. (There is one project: yours.)
- No "team workspace" tab. (Phase 2.)
- No "API key name" field. (The key is named for you: `csk_live_<8 chars>`.)
- No multiple keys per account in free tier. (One account, one key. Rotate by deleting.)
- No provider picker for any category. (We pick — for every endpoint, not just LLMs.)
- No region picker. (We pick — closest to user's egress.)
- No spend cap slider. (The free tier is a hard cap; paid tier is unlimited until your card declines.)
- No webhook configuration page. (Phase 2.)
- No retention slider for logs. (We pick: 7 days free, 30 paid.)
- No theme picker. (System.)
- No billing tab when there is no billing. (Empty state is dishonest — we just don't show it.)

If a feature does not survive the test "would Steve Jobs leave this in?", it does not ship in Phase 1.

---

## 2. Why this works (the architectural insight)

The "twenty signups" tax exists because every dev-infra vendor wants to be a dashboard relationship with you. Stripe wants you in their account console. OpenAI wants you in `platform.openai.com`. Cloudflare wants you in their edge UI. Each one has correctly identified that the dashboard is where retention lives.

This is fine for the vendor. It is hostile to the developer.

The gap is that **most developers, most of the time, want exactly one thing from each vendor: a working endpoint.** They do not want to learn Stripe's data model on day one. They do not want to think about OpenAI's tier limits on day one. They want to send an email, charge a card, summarize a PDF, and ship.

Caravansary is the bet that **a uniform "send an email / store a file / register a domain / charge a card / summarize a PDF" API, fronting the actual vendors invisibly, is what 80% of indie projects need 100% of the time.** When that bet is right, we win because:

1. **One signup beats twenty.** The dev-experience win is so large that we do not need to be cheaper than the underlying vendors. We need to be roughly even.
2. **Free-tier composition is a real arbitrage, but not a quota-circumvention strategy.** Many infrastructure vendors publish meaningful free quotas — Cloudflare Workers, R2, and DNS for edge and storage; AWS SES for transactional email; self-hosted GlitchTip and PostHog OSS for monitoring; multiple LLM providers for inference. By composing one honest account per provider with hard per-user caps, we offer a useful aggregate free tier without pretending public quotas are infinitely shardable. Our cost to provide this is dominated by routing engineering and abuse controls, not API spend.
3. **Aggregation creates a graduation moat.** When the user does grow out of free, Connected Providers keep them on us — because they can bring existing vendor accounts, credits, or keys into the Caravansary control plane while keeping the same code path and metrics dashboard. We charge a platform fee, not a margin on usage.

The "one signup" experience is the wedge. The "free-tier composition" is what makes the wedge profitable to give away. The Connected Providers graduation is what keeps users when they win.

---

## 3. Phasing

### Phase 0 — The core endpoints, no UI (~6 weeks, solo)

Ship the gateway. Ship the OAuth flow. Ship the key page. Ship the smallest coherent set of `/v1/*` endpoints. Ship the SDKs (TS first, then Python, then Go).

Goals:
- Sign in with Google → key page in <10 seconds.
- `curl` works in <30 seconds from the moment the page loads.
- Free tier holds: a real side project ships without paying us, and without us paying meaningful $$ for them.

Success criteria:
- 100 sign-ups via word of mouth (HN "Show HN" + r/SaaS + Bluesky).
- 10 of them make a real call within 60 seconds of signing in.
- 5 of them ship a side project that uses **three or more** Caravansary categories. (This is the "one key replaces twenty" proof, not "they used our LLM endpoint.")
- Blended free-tier vendor spend across all categories < $50 across all 100 users in the first month. (If this is wrong, the architecture is wrong.)

Stack:
- Edge gateway: Cloudflare Workers (free tier: 100k req/day per script).
- Control plane: Cloudflare D1 (users, keys, usage counters).
- Long-term storage: Cloudflare R2 (logs, audit).
- Auth: Better Auth (OSS) on Workers, with Google + GitHub OAuth providers.
- Metering: in-memory rate limit + Upstash Redis token bucket (free tier: 10k commands/day) + nightly Lago batch into Stripe.
- Provider router: in-house, generic across categories — the same router serves email, storage, DNS, error tracking, and LLM endpoints. Health-check each provider every 60s, route around 5xx and 429.

### Phase 1 — Polish, more vendors, the graduation path (~3 months)

Add:
- The "upgrade" button → Stripe Checkout. One paid tier. $19/mo flat. Connected-provider passthrough. Higher limits.
- Connected Providers vault — paid users can connect existing OpenAI / Anthropic / Stripe / cloud accounts or store vendor keys as control-plane credentials. Their project still uses only `CARAVANSARY_API_KEY`.
- Hosted `caravansary` MCP server — the same one-key abstraction exposed as agent tools, with scopes and audit logs.
- Webhook fan-out (`/v1/event/subscribe`).
- Two more categories: queues (`/v1/queue/push`), feature flags (`/v1/flag/check`).
- A status page that's actually accurate.

Success criteria:
- 1,000 free users.
- 50 paid users.
- $1,000 MRR.
- Free-tier blended cost-per-user < $0.10/mo. (If this fails, identify the most-expensive endpoint of the month — could be LLM, could be email, could be storage egress — and either reprice it, gate it harder, or move it BYOK-only.)

### Phase 2 — Team workspaces, audit log, partner provisioning (~6 months)

- Multi-key (named keys, per-environment, scoped permissions).
- Team workspaces (one billing relationship, many keys).
- Just-in-time downstream activation: when a user's usage on `/v1/payment/*` crosses from simulation into real money, we return a Stripe-hosted activation link and then resume the same API flow. Same pattern for Cloudflare zones, Vercel projects, etc. The user sees the vendor only at the moment the real world requires it.
- Audit log surfaced in UI.
- Status page with real SLO numbers.

### Phase 3 — Reseller agreements, MoR, EU residency (~12 months)

- Apply to Cloudflare Partner Network. Apply to AWS APN Select. Apply to Twilio ISV.
- Once approved, switch eligible categories from BYOK-or-master-key to true multi-tenant provisioning under partner programs.
- Onboard Paddle or Polar as Merchant of Record so we can charge end-customers globally without the VAT registration nightmare.
- Region-pin the gateway: `eu.api.caravansary.dev` → European data path only, via Vertex AI / Bedrock EU regions.

---

## 4. The seamless experience expectation, restated as design rules

(Repeating because the user instructed it, and because every PR review should reference this list.)

**Rule 1 — Two screens, never three.** From caravansary.dev to a copyable key, there are exactly two screens (homepage, key page) plus the OAuth round-trip the IdP owns. If anyone proposes a feature that requires a third screen on the onboarding path, the feature does not ship until it can be demoted off that path.

**Rule 2 — Default everything.** The free tier exposes zero choices. Provider, model, region, retention, rate limit, key name, project name — all chosen by us. The user can override later, after they have shipped. They cannot override before.

**Rule 3 — One key, one endpoint, one spend surface.** The user sees one credential, one base URL, one Caravansary platform invoice, and one usage ledger. True vendor-consolidated billing arrives only where reseller/MoR agreements make it real; every routing decision remains invisible either way.

**Rule 4 — No card on free tier, ever.** Not "to verify identity." Not "for spam protection." Not "it'll only charge if you go over." No.

**Rule 5 — The graduation is itself seamless.** Upgrading does not change the endpoint, the SDK, or the key. The same code keeps working. New capabilities appear; nothing existing breaks.

**Rule 6 — Configuration is a graduation feature, not an onboarding feature.** "Choose your provider" is a power-user setting that appears *after* the user has shipped, not before. Every config knob's default position is what 95% of users would have picked anyway.

**Rule 6b — Connected providers never become project credentials.** Later, Caravansary may become the control plane where users connect third-party integrations, OAuth apps, cloud accounts, and existing vendor keys. Those credentials stay inside Caravansary's vault. The user's app still has one environment variable: `CARAVANSARY_API_KEY`.

**Rule 7 — Empty states are dishonest. Hide them.** No "billing" tab when there is no billing. No "team" tab when there is no team. No "webhooks" tab when no webhooks are configured. The UI grows as the user does.

**Rule 8 — Failure modes are also seamless.** If a downstream vendor 5xx's, the user gets a 200 from a different vendor — not a "we're routing to a fallback, please retry" toast. The router eats the failure.

**Rule 9 — The marketing is the experience.** The landing page does not have a hero animation explaining "what we do." It has a "Sign in with Google" button. The product is the demo.

**Rule 10 — Cut, don't add.** Every quarter, we kill at least one knob, one toggle, or one screen. If we cannot find one to kill, we are not shipping enough.

---

## 5. Business model — free for us, free for the user (Tier 1)

### 5.1 Why our cost to serve a free-tier user is near zero

The unit of cost on this platform is the underlying vendor's bill, summed across all categories the user touches. We minimize ours by:

1. **Routing free-tier traffic to honest public quotas across every category.** Cloudflare Workers includes 100k req/day, R2 includes 10GB-month stored with no egress, D1 includes 5M rows read/day, AWS SES from EC2 includes 62,000 emails/month, self-hosted GlitchTip and PostHog OSS run on a single Fly machine, and multiple LLM providers expose meaningful daily free quotas (Gemini, Groq, Cloudflare Workers AI). These quotas are per account/org/project, not per user. We do not shard users across duplicate master accounts to evade limits; when public quotas stop being enough, we add paid capacity, partner programs, or BYOK.
2. **Caching aggressively inside tenant boundaries.** Every endpoint has a deterministic tenant-scoped cache key. Identical requests within the same tenant — repeated DNS lookups, idempotent storage writes, repeated prompts, repeated event payloads — can be served from edge cache before calling the vendor. Cross-tenant caching is allowed only for public demos, static examples, or artifacts explicitly marked shareable.
3. **Normalizing schemas at the gateway.** No SDK overhead, no per-vendor adapter bloat — the gateway is small enough to fit in a Cloudflare Worker.

Target: blended cost-per-active-free-user **under $0.10/mo across all nine categories combined**. This is achievable because most of the free quota is large and most users will not approach it on any single category.

### 5.2 Why our cost to *build* the free tier is near zero

Same logic applied to our own infra:
- Gateway: Cloudflare Workers (free).
- DB: Cloudflare D1 (free) + Neon (free) for relational user data.
- Cache: Upstash Redis (free).
- LLM: stack of free LLM tiers above.
- Object storage: Cloudflare R2 (free, no egress).
- Auth: Better Auth OSS, self-hosted on Workers.
- Metering: Lago (OSS) on a single Fly machine ($0 free trial; later ~$5/mo).
- Email: Resend free tier for our own auth emails (3,000/mo, vastly more than we'll send).
- Monitoring: Sentry free + Grafana Cloud free.
- DNS / domain: Cloudflare DNS (free) + a $13/yr `.dev` registration. The only recurring out-of-pocket cost.

Total monthly burn for Phase 0: **<$5/mo**, dominated by the domain prorate.

### 5.3 The paid tier — one plan, one button

**$19/mo, flat.**

What you get:
- 10x the free-tier rate limits.
- BYOK passthrough at 0% markup via Connected Providers. Connect your OpenAI / Anthropic / Stripe accounts or keys; we route through them; you pay them directly; your project still only sees the Caravansary key.
- Provider preferences. "Always Anthropic for chat. Always Postmark for email."
- 30-day log retention.
- Webhooks.
- An email address that gets answered by a human.

What you do *not* get:
- A pricing page with five tiers.
- A sales call.
- A contract.
- Custom rate limits.
- An SSO enterprise tier upcharge.

Phase 3+ may introduce a Team plan ($X per seat) and an Enterprise plan (custom, EU residency, SOC 2). We will not introduce them earlier, and they will not be on the marketing site until they exist.

### 5.4 Long-term unit economics

Cost per active free user is the sum of nine category costs, not one. We expect a long-tail shape: most users will only meaningfully exercise two or three categories (typically auth-email + storage + one of LLM / events / DNS), and most usage within those categories will fit inside vendor free quotas. The blended-vendor cost to serve a typical free user is therefore dominated by whichever single category that user happens to lean on hardest — for some users that's email, for some that's storage egress, for some that's LLM tokens. Paid users pay $19/mo flat and we project a margin north of 60% on them at typical usage; connected-provider customers still pay us the $19/mo platform fee while their vendor spend flows through their own accounts.

The contingency for any single category's free economics inverting (a vendor shrinks free tier, a price rises) is the same regardless of which category it is: kill switch on the master-key route, fall back to a constrained managed demo path, and graduate serious usage into Connected Providers at zero markup. The free path still demonstrates "one key, working endpoints"; it does not become a vendor-key setup wizard. This contingency is category-agnostic — the same playbook applies whether the wobbly category is LLM, email, hosting, or DNS.

---

## 6. Architecture (Phase 0)

```
                           ┌───────────────────────┐
                           │  caravansary.dev      │
                           │  (Cloudflare Pages)   │
                           └───────────────────────┘
                                     │
                                     ▼
            ┌─────────────────────────────────────────────────┐
            │   Edge Gateway  (Cloudflare Workers, global)    │
            │                                                  │
            │   /v1/llm/* /v1/email/* /v1/payment/*           │
            │   /v1/host/* /v1/dns/* /v1/file/* /v1/error/*   │
            │                                                  │
            │   - JWT verify                                   │
            │   - Rate limit (Upstash token bucket)           │
            │   - Provider router (health-checked round-robin)│
            │   - Schema normalizer                            │
            │   - Usage event emitter → Lago                   │
            └─────────────────────────────────────────────────┘
                │            │            │            │            │            │
                ▼            ▼            ▼            ▼            ▼            ▼
            ┌───────┐    ┌───────┐    ┌───────┐    ┌───────┐    ┌───────┐    ┌───────┐
            │ DNS / │    │ Email │    │Storage│    │ Error │    │Payment│    │  LLM  │
            │subdom.│    │       │    │       │    │ /Evts │    │       │    │       │
            │  (CF) │    │ (SES, │    │ (R2)  │    │(Glitch│    │(Stripe│    │(Gemini│
            │       │    │Resend)│    │       │    │Tip,PH)│    │Connect│    │ Groq, │
            └───────┘    └───────┘    └───────┘    └───────┘    └───────┘    │ etc.) │
                                                                              └───────┘

                            ┌───────────┐
                            │   Auth    │
                            │ (Better   │
                            │  Auth OSS │
                            │  on Wkrs) │
                            │           │
                            │  Google   │
                            │  GitHub   │
                            └───────────┘

                            ┌───────────┐
                            │ Control   │
                            │ Plane:    │
                            │  D1 +     │
                            │  Lago +   │
                            │  Stripe   │
                            └───────────┘
```

**Deliberately not in the diagram:**
- A user-facing dashboard with more than one screen.
- A vendor-management UI.
- A multi-region replicated control plane.
- A microservices mesh.
- A queue between the gateway and the vendors.

These are all reasonable Phase-2 additions. They are wrong for Phase 0.

---

## 7. Risks & mitigations

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| A free-tier vendor in *any* category (LLM, email, storage, DNS, etc.) revokes / shrinks their free quota | Medium | High | Multi-vendor router with N≥2 free providers per category where the market supports it; OSS self-host (GlitchTip, PostHog, etc.) where it doesn't. If one dies, we still have N−1 or the self-hosted fallback. |
| Vendor ToS interpretation flips against us (esp. OpenAI/Anthropic on master-key resale, or any single-source category vendor we depend on) | Medium | High | Master-key path used only for endpoints whose ToS is unambiguous (per-vendor scoping in [LEGAL.md](./LEGAL.md)). Anything ambiguous defaults to Connected Providers after graduation. The risk applies equally to email providers, hosting providers, and storage providers, not just LLMs. |
| Abuse: spammers using `/v1/email/send` to send spam under our accounts | High | High | Per-user domain verification before email send; outbound abuse heuristics; per-tenant subaccount on Resend; killswitch. |
| Abuse: prompt-injection / jailbreak content getting routed via our master LLM key, attributing the abuse to us with the vendor | Medium | Medium | Run a safety classifier on master-key inputs and outputs — OpenAI Moderation API where permitted, or provider/local guardrails where data routing requires it. Log abuse minimally. Cooperate with vendor. |
| Cost runaway because someone discovered our master key gives them free Anthropic credits | Low | Catastrophic | Per-user daily $ ceiling, hard. Per-key second-level rate limit at 100x typical use. Synthetic abuse canary. |
| The free-tier-stacking arbitrage gets killed (vendors shrink or remove free capacity) | Medium | Medium | Preserve a constrained managed demo path, then graduate serious usage into Connected Providers at zero markup. The product survives because the user's project still calls Caravansary, not vendors directly. |
| Big LLM gateway (OpenRouter, Vercel) ships an "all dev categories" version of this | Medium | High | We're a wedge product (DX, $0, no signups) not a gateway product (latency, model breadth). If we lose the wedge race, we lose. Move fast on the seamless onboarding lock-in. |
| Legal liability for content the platform mediates (LLM output, email content) | Medium | High | Connected-provider routing as the default for any path where vendor ToS is unclear. Insurance (E&O / cyber) by paid-tier launch. Clear AUP that flows down vendor policies. See [LEGAL.md](./LEGAL.md). |

---

## 8. What this is *not*

- **Not an LLM gateway.** LLMs are one of nine categories. The product proposition does not depend on the LLM endpoint at all — a user who only ever calls `/v1/dns/*`, `/v1/email/*`, and `/v1/file/*` gets the full "one key replaces twenty" value. If you are scoping work and you find yourself prioritizing the LLM endpoint because "that's the most exciting one," reread the reading note at the top of this file.
- Not Zapier — we don't connect SaaS apps to each other for non-developers. We are a developer's API key.
- Not OpenRouter — they are an LLM-only gateway. We are not.
- Not Vercel — we don't host the user's code. We host the API call to the host.
- Not Pipedream — we don't help you build integrations into other people's apps. We replace the keys you'd use to build *your* app.
- Not Stripe Atlas — we don't incorporate companies. We replace the dev-runtime side of what Atlas covers via perks (and we're free).
- Not "BYOK only" like Helicone — free tier is master-key proxied, intentionally. BYOK is graduation.
- Not an enterprise sales motion. There is no "contact sales." There is sign in.

---

## 9. The one-line sell

> *Caravansary turns "twenty signups before line one of code" into one signup. Sign in with Google. Copy your key or run `caravansary init`. Ship.*

If we ever cannot honestly say that line about the live product, we have shipped the wrong thing.
