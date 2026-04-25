# LEGAL.md — Caravansary

*Are we OK with reselling other services? Which ones, in which mode, and what would require a partnership?*

April 2026. **Not legal advice.** This is a developer's working scoping document. Every URL below should be re-fetched and diffed against the version cited here before any commercial launch — vendor ToS in this space have shifted multiple times since 2024 and will keep shifting. Nothing in this document substitutes for engagement with a real lawyer before we collect a paying customer.

**On reading weight.** This file devotes a disproportionate amount of text to LLM vendors (six of them, in §2.1–§2.6) because there happen to be many distinct LLM vendors with non-trivial and frequently-changing ToS to scope. The page-weight should not be read as a signal that LLMs are the central category. Caravansary is one API key replacing many across nine categories; the legal complexity per category is uneven, and LLM happens to be the heaviest. Email (§2.13), payments (§2.7), and the cloud platforms (§2.8–§2.10) carry equal product weight even though they take fewer paragraphs to scope.

The honest summary: **the seamless-onboarding thesis pushes us toward "platform reselling," which is the legally fraught mode. The legal scoping pushes us back toward connected-provider modes like BYOK, which are less seamless if exposed during onboarding. The product design has to thread that needle: projects still use only the Caravansary key, while the control plane may hold provider credentials later.**

---

## 0. The four provisioning modes

Every third-party service we want to abstract can be integrated in one of four ways. The choice has direct legal consequences.

| Mode | Description | Who is the vendor's customer? | Who bills the end-user? | Compliance burden |
|---|---|---|---|---|
| **(A) Master-key resell** | We hold one master account with the vendor; all our users' calls use it; we re-bill our users. | Caravansary. | Caravansary. | High — we own AUP enforcement, abuse, KYC, MoR. |
| **(B) Multi-tenant provisioning** | Via OAuth or partner API, we create a vendor-side subaccount for each user. | The user (with Caravansary as broker). | Vendor (or us under partner agreement). | Medium — gated by partner approval. |
| **(C) BYOK** | User signs up with vendor directly, connects or stores their vendor credential in Caravansary's control plane. Runtime calls still use the Caravansary key. | The user, directly. | The user, directly. | Low — we are a tool. |
| **(D) Formal reseller / partner** | Signed contract with vendor; revenue share; KYC; vendor reviews our use. | Mixed. | Either side. | Medium — gated by formal agreement. |

The seamless thesis (sign in → key → done, no vendor-by-vendor signups) **wants Mode A or B everywhere.** The legal landscape **only allows that for some services.** This document is the per-service scoping that tells us which mode each one gets in Phase 0, and which becomes possible later via partner programs.

---

## 1. Executive verdict per service

| # | Service | A Resell | B Multi-tenant | C BYOK | D Partner | Verdict for $0 free tier |
|---|---|---|---|---|---|---|
| 1 | OpenAI | Red | Red without contract | Green | Yellow (enterprise only) | **C only** (until Phase 3 partner discussion) |
| 2 | Anthropic | Red | Red without contract | Green | Yellow (limited) | **C only** |
| 3 | Google Gemini API (AI Studio) | Yellow | Yellow | Green | Yellow | **A allowed for free-tier inference** ([see §2.3](#3-google-gemini-api)) — verify language re: "substitutes for" |
| 4 | Groq API | **Likely A-friendly** | n/a | Green | Yellow | **A on free tier** (verify ToS language) |
| 5 | DeepSeek API | Yellow | n/a | Green | Yellow | **C; A only with explicit ToS read** |
| 6 | OpenRouter (free models) | n/a (already an aggregator) | n/a | Green | n/a | **C only** — chaining aggregators violates their AUP |
| 7 | Stripe | n/a | **Green via Connect onboarding** | Green | Green | **B (Connect)** — just-in-time user onboarding before live money moves |
| 8 | AWS | Yellow | Green via Organizations + APN | Green | Green (APN/Marketplace) | **C now, D later** |
| 9 | Google Cloud | Yellow | Green via Partner Sales Console | Green | Green | **C now, D later** |
| 10 | Cloudflare | Yellow | **Green via Tenant API (partner-only)** | Green | Green (Partner Network) | **A on our own zones for free tier; B after partner approval** |
| 11 | Fly.io | Yellow | Green (orgs API) | Green | Yellow (informal) | **A on our own org for free; B per-user later** |
| 12 | Vercel | Red (multi-tenant under our team) | Green via Vercel Integration | Green | Yellow | **B (Integration) or C** |
| 13 | Resend | Yellow | Yellow (domains-as-tenant) | Green | Yellow | **A on free tier with per-tenant subdomain** (verify); else C |
| 14 | Postmark | Yellow | Yellow | Green | Yellow | **C** (their AUP is strict on multi-tenant) |
| 15 | SendGrid (Twilio) | Red without ISV | **Green via Subusers + ISV** | Green | Green (ISV / Twilio Build) | **C now, B after ISV approval** |
| 16 | AWS SES | Yellow | Green | Green | Green | **A from our AWS account for free-tier email; per-tenant verified domains required** |
| 17 | GitHub | n/a | **Green via GitHub App** | Green (PAT) | n/a | **B (GitHub App)** |
| 18 | Sentry / GlitchTip (OSS) | Self-hosted = Green | n/a | Green | n/a | **A self-hosted GlitchTip** |
| 19 | Plausible / PostHog (OSS) | Self-hosted = Green | n/a | Green | n/a | **A self-hosted** |

**Big-picture takeaway:** for the $0 free tier shipping in 2026, the cleanest viable architecture is a *hybrid* — Mode A on services whose ToS explicitly permits it (free-tier LLMs with no resale prohibition; OSS we self-host; transactional email under per-tenant domain verification), Mode B on services with formal partner-friendly OAuth flows (Stripe Connect, GitHub Apps, Vercel Integrations), and Mode C as the fallback for everything else (OpenAI, Anthropic, AWS, GCP, Postmark). Mode C is a control-plane integration, not a project credential: the user's app still sees only `CARAVANSARY_API_KEY`.

The product UI does not show the difference. The user signs in once, gets one key, calls one endpoint. The hybrid happens in our gateway code.

---

## 2. Per-service legal scoping

### 2.1 OpenAI

**Canonical URLs to verify:**
- Business Terms — https://openai.com/policies/business-terms/
- Usage Policies — https://openai.com/policies/usage-policies/
- Sharing & Publication Policy — https://openai.com/policies/sharing-publication/
- Enterprise Privacy — https://openai.com/enterprise-privacy/

**What's allowed:**
- (A) Resell — **Not allowed** under standard Business Terms. Usage Policies require us to (i) enforce OpenAI's policies on our users, (ii) not transfer access in a way that obscures the end user from OpenAI's safety stack, and (iii) take responsibility for user usage. Pure resell where OpenAI sees only "Caravansary" and we re-bill anonymous users is a violation.
- (B) Multi-tenant — Not allowed without a custom contract. No public OAuth/subaccount API.
- (C) BYOK — Green. The user holds their own OpenAI account; Caravansary stores and uses the connected credential server-side. This is the path used by Cursor, Continue, Cline, Helicone (paid), Portkey, except our user's project still only stores the Caravansary key.
- (D) Partner — No public reseller program with revenue share. Case-by-case enterprise deals; the [OpenAI Startup Program](https://openai.com/form/startup-program/) gives credits but not resale rights.

**Mode for default $0 free tier: not exposed.** If a user explicitly needs OpenAI before we have a partner path, the only clean mode is C through the connected-provider vault. Optional Phase-2: a small "managed OpenAI" tier where users explicitly opt in, with strict per-user moderation logging and a per-call disclosure that calls flow through our master key — but only after counsel signs off.

### 2.2 Anthropic

**Canonical URLs:**
- Commercial Terms of Service — https://www.anthropic.com/legal/commercial-terms
- Usage Policy (AUP) — https://www.anthropic.com/legal/aup
- Privacy Policy — https://www.anthropic.com/legal/privacy

**What's allowed:**
- (A) Resell — Not allowed under standard Commercial Terms. AUP flow-down obligations make pure resell incompatible with the standard agreement.
- (B) Multi-tenant — Not allowed without contract.
- (C) BYOK — Green.
- (D) Partner — Limited. Anthropic ships through cloud partners (AWS Bedrock, GCP Vertex). Selective enterprise partnerships. No self-serve reseller tier.

**Mode for free tier: C.** *Bedrock / Vertex caveat:* if a user has an AWS or GCP account, we can route Anthropic calls via Bedrock under their cloud, which makes their cloud the contracting party. This shifts liability cleanly but requires a configured AWS/GCP account on the user side, which breaks our seamlessness — so it's not Phase-1.

### 2.3 Google Gemini API

**Canonical URLs:**
- Generative AI Additional Terms of Service — https://ai.google.dev/gemini-api/terms
- Generative AI Prohibited Use Policy — https://policies.google.com/terms/generative-ai/use-policy
- Google APIs Terms of Service — https://developers.google.com/terms

**What's allowed:**
- (A) Resell — **Yellow.** Google APIs ToS forbids using the APIs in a way that "substitutes for" the Google service to your users without permission. Routing free-tier Gemini calls through our master key, with the user calling our generic `/v1/llm/chat`, is at the edge of this clause. Counsel review required, but the literal reading: as long as we're not branding it as "Gemini powered by Caravansary" or hiding from users that the call may be served by Gemini, the routing is closer to "use of API to provide a service" than "substitute for Gemini." Many AI gateways operate this way without enforcement, but absence of enforcement is not permission.
- (B) Multi-tenant — Yellow. Gemini-via-Vertex on a customer's GCP project is real (see Google Cloud below); Gemini-via-AI-Studio is not multi-tenant.
- (C) BYOK — Green.
- (D) Partner — Available via [Google Cloud Partner Advantage](https://cloud.google.com/partners), GCP-level relationship.

**Mode for free tier:** **A with caveats** — the master-key path is operationally usable for free-tier inference; user-facing language should disclose that the request *may* be routed to Gemini (alongside Groq, DeepSeek, etc.). Counsel should review the "substitutes for" language before commercial launch.

### 2.4 Groq

**Canonical URLs:**
- Terms of Service — https://groq.com/terms-of-use/
- Acceptable Use Policy — https://groq.com/aup/
- Pricing — https://groq.com/pricing/

**What's allowed:**
- Groq's documented base limits include 14,400 RPD for Llama 3.1 8B Instant, but limits apply at the organization level and exact current limits should be verified in the account limits page before launch. Their ToS as of recent revisions has been broadly developer-friendly; resale-style usage is not explicitly prohibited. **Verify current text before launch.**
- (C) BYOK — Green.
- (D) Partner — Informal as of 2025; no public reseller program.

**Mode for free tier: A.** Groq is one of the foundational providers in our free-tier router. Treat as the primary throughput backbone for OSS-model traffic.

### 2.5 DeepSeek

DeepSeek pricing (as of late 2025) is among the cheapest API-LLMs available. ToS is in active flux. Treat as **Yellow**: BYOK in Phase 0. Re-evaluate routing through master key only after a careful read of the latest ToS. There are also export-control considerations (US users routing through a Chinese-hosted API) that warrant counsel review for paid-tier US customers.

### 2.6 OpenRouter (as a downstream)

We could trivially use OpenRouter as one of our LLM providers (their free plan currently exposes free models behind a tight daily request limit). **Don't, except as last-resort fallback.** OpenRouter's AUP prohibits "reselling or rebroadcasting OpenRouter services" — chaining aggregators is a clear violation. Use OpenRouter only on the BYOK path (paid tier) when the user explicitly chooses it.

### 2.7 Stripe

**Canonical URLs:**
- Stripe Connect overview — https://stripe.com/connect
- Connect account types — https://docs.stripe.com/connect/accounts
- Connected Account Agreement — https://stripe.com/legal/connect-account
- Stripe Services Agreement — https://stripe.com/legal/ssa

**What's allowed (this is the textbook B case):**
- **Standard accounts / Connect onboarding** — user has a Stripe relationship linked to our platform. Stripe handles merchant onboarding and identity collection; we are the platform. Best for the seamless flow: `/v1/payment/checkout` works in test mode immediately, and the first live-money call returns a hosted activation link that resumes the same API flow after onboarding. Cleanest legal posture.
- **Express accounts** — Stripe-hosted onboarding; we do more lifting; we can take a cut.
- **Custom accounts** — full embedding; more compliance obligations.
- **Stripe is NOT a Merchant of Record by default.** Connect is a platform/marketplace pattern, not MoR. Don't conflate.

**Mode: B via Connect.** The end user is the legal merchant for their charges; we orchestrate. Platform fees via Connect application fees where applicable; we are a platform, not a payment processor.

**The seamlessness compromise:** the *first live-money* Stripe call by a user *will* surface a Stripe-hosted onboarding flow because Stripe's KYC requires the merchant to provide their own legal info — there is no way around this. We hide it from the rest of the experience: until they make their first live charge, the payment endpoint can operate in test/simulation mode.

### 2.8 AWS

**Canonical URLs:**
- AWS Customer Agreement — https://aws.amazon.com/agreement/
- AWS Partner Network — https://aws.amazon.com/partners/
- AWS Solution Provider Program — https://aws.amazon.com/partners/programs/solution-provider/
- AWS Marketplace — https://aws.amazon.com/marketplace/
- AWS Activate — https://aws.amazon.com/activate/
- Acceptable Use Policy — https://aws.amazon.com/aup/

**What's allowed:**
- (A) Resell — Yellow without APN. The AWS Customer Agreement allows us to use AWS to provide services; explicit *resale* of AWS as AWS requires the **Solution Provider Program**.
- (B) Multi-tenant — Green. AWS Organizations + IAM Identity Center is the canonical pattern. We can also create AWS accounts on behalf of customers via the Organizations API once they're under our management; this needs MSA-level paperwork to be clean.
- (C) BYOK / cross-account roles — Green and conventional.
- (D) APN tiers (Select / Advanced / Premier). Marketplace SaaS is an alternate path with Amazon collecting payment.

**Mode for free tier: C (cross-account role) for the rare AWS-touching path.** We minimize AWS dependency in Phase 0 (favoring Cloudflare for hot path). Move to D when monetizing or expanding.

### 2.9 Google Cloud

**Canonical URLs:**
- GCP Terms of Service — https://cloud.google.com/terms/
- Google Cloud Partner Advantage — https://cloud.google.com/partners
- Google Cloud for Startups — https://cloud.google.com/startup
- Acceptable Use Policy — https://cloud.google.com/terms/aup

Mirrors AWS structurally. Partner Sales Console + Reseller agreement enables formal resale. Without that, customer-owned GCP project + service account or Workload Identity Federation is the BYOK pattern.

**Mode: C now, D when monetizing.**

### 2.10 Cloudflare

**Canonical URLs:**
- Self-Serve Subscription Agreement — https://www.cloudflare.com/terms/
- Cloudflare for Startups — https://www.cloudflare.com/forstartups/
- Cloudflare Partner Network — https://www.cloudflare.com/partners/
- Tenant API (partner-only) — https://developers.cloudflare.com/tenant/

**What's allowed:**
- (A) Resell — Yellow without partner agreement.
- (B) Multi-tenant via **Tenant API** — Green, but partner-only. Once approved, programmatic creation of child accounts owned by the customer.
- (C) BYOK via API tokens scoped to user's zone — Green.
- Workers ToS specifically: multi-tenant Workers (running other people's code on our account) is **Yellow** — Cloudflare permits SaaS to run user-supplied code, but we must enforce abuse policies and sustained abuse will result in account suspension.

**Mode: A on our own zones for the free-tier `/v1/dns/*` endpoint (we own the apex; users get subdomains under our zone). C on user-owned zones for paid users.** Apply to the Cloudflare Partner Network early; the Tenant API unlocks proper B behavior in Phase 3.

### 2.11 Fly.io

**Canonical URLs:**
- Terms of Service — https://fly.io/legal/terms-of-service/
- Acceptable Use — https://fly.io/legal/acceptable-use/
- Organizations / Machines API — https://fly.io/docs/

**What's allowed:**
- Fly's "organizations" model is multi-tenant-friendly. We can be a member of a customer's org with a deploy token, or vice versa.
- No formal reseller program as of early 2026; partner discussions are case-by-case.
- Hosting end-user code on our own org is technically allowed but we absorb all abuse responsibility.

**Mode for free tier: A on our own org with strict per-user resource isolation; B (deploy tokens into user's org) for paid.** The free tier's `/v1/host/deploy` runs the user's container under our org with a sandboxed network and a per-user CPU/memory cap.

### 2.12 Vercel

**Canonical URLs:**
- Terms of Service — https://vercel.com/legal/terms
- Acceptable Use — https://vercel.com/legal/aup
- Vercel Integrations / Marketplace — https://vercel.com/integrations
- Vercel for Startups — https://vercel.com/startups

**What's allowed:**
- Hosting end-customer apps under our single Vercel team — **Red in spirit.** Vercel's ToS treats our team as the responsible party; an unbounded number of strangers' Next.js apps under one team is hosting reseller without contract.
- **Vercel Integrations (OAuth)** is the supported pattern — our app deploys to the user's own Vercel team. Green.
- Vercel Marketplace listing is the formal partner path.

**Mode: B via Integration, or C through the connected-provider vault.** Free-tier users targeting Vercel-style hosting should use our own infra (Cloudflare Workers, Fly) rather than Vercel-via-Caravansary.

### 2.13 Resend / Postmark / SendGrid / AWS SES

**Canonical URLs:**
- Resend Terms — https://resend.com/legal/terms-of-service · AUP — https://resend.com/legal/acceptable-use
- Postmark Terms — https://postmarkapp.com/eula · Sending Policy — https://postmarkapp.com/support/article/1078-what-is-postmark-s-sending-policy
- Twilio MSA / SendGrid — https://www.twilio.com/legal/tos · Twilio AUP — https://www.twilio.com/legal/aup · Twilio ISV — https://www.twilio.com/partners/isv
- AWS SES — https://aws.amazon.com/ses/ · AWS AUP — https://aws.amazon.com/aup/

**The shared rule across all four:** the sender must own the domain and pass DMARC/SPF/DKIM. Sending on behalf of users without their domain auth is broadly prohibited, *and will get any shared IP blocked anyway* — so this is operational, not just legal.

- **Twilio (incl. SendGrid)** has a real subaccount API + ISV program. Best-in-class for B once we're approved.
- **Resend's** domain-per-tenant model approximates B but isn't a formal partner program; verify "send on behalf of customers" language.
- **Postmark** explicitly requires us to enforce their sending policy; bulk SaaS sending via one Postmark account is discouraged.
- **AWS SES** allows multi-tenant sending so long as we maintain reputation; very generous free tier (62,000/mo from EC2).

**Mode for free tier:** **A via AWS SES from our account, with per-tenant verified domains** — when a user calls `/v1/email/send`, we either send from `noreply@<their-verified-domain>` (preferred) or, if no domain configured, `noreply@<their-key>.send.caravansary.dev` (limited to magic-link / OTP-style mail; no marketing). Postmark/Resend remain BYOK options for paid users who want better deliverability. SendGrid via subaccounts becomes available after Twilio ISV approval.

### 2.14 GitHub

**Canonical URLs:**
- GitHub Terms of Service — https://docs.github.com/en/site-policy/github-terms/github-terms-of-service
- GitHub Apps — https://docs.github.com/en/apps
- OAuth Apps vs GitHub Apps — https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/deciding-when-to-build-a-github-app

**Mode: B via GitHub App.** Unambiguously green. User installs our App on their repos; we get scoped, revocable tokens. This is also the pattern for our login flow ("Sign in with GitHub").

---

## 3. Recommended legal architecture for Phase 0

The seamless thesis demands one signup; the legal scoping demands per-service variation. We resolve the tension as follows:

**Architecture:** *Hybrid with the user blissfully unaware.* The user signs in to Caravansary once. We mint a Caravansary key. From their perspective, that key works against every `/v1/*` endpoint. Internally, depending on the endpoint:

- **Mode A (master-key resell):** we serve from our own account on Gemini, Groq, AWS SES, our own Cloudflare zone, our own Fly org, self-hosted GlitchTip / Plausible. The free tier is largely powered by Mode A traffic, which is what makes the unit economics work.
- **Mode B (just-in-time provisioning):** we create or link a downstream account only when the user first needs the real-world capability. Stripe Connect onboarding appears only when the user makes their first live `/v1/payment/*` call. GitHub App installation appears only when the user calls `/v1/git/*`. The user sees the vendor ceremony at the moment it is legally required; before that, nothing.
- **Mode C (connected-provider vault):** for OpenAI, Anthropic, Postmark, and any vendor whose ToS we cannot navigate cleanly, BYOK is the only path. In Phase 0, those endpoints don't exist on the free tier — the master-key-friendly providers (Gemini, Groq) cover the LLM use cases. In Phase 1, we add a provider vault so paid users can connect OpenAI or Anthropic while their projects keep using the same Caravansary key.

**This is the only architecture I can find that ships in 2026 without contracts** while preserving the two-screen onboarding for the 95% case.

**The 5% case** — users who specifically need OpenAI or Anthropic from day one — sees provider connection as a graduation feature in settings, not as an onboarding requirement. They are the same persona who will pay $19/mo, so the friction is acceptable as long as it never enters project setup.

**When to graduate to Mode D (formal partner agreements):** after Series A and a real general counsel, pursue:
- AWS Solution Provider (Phase 3)
- Cloudflare Partner Network → Tenant API (Phase 2; multi-month review cycle, apply early)
- Twilio ISV (Phase 2; for proper email subaccounts)
- Vercel Marketplace (Phase 3)

For LLMs specifically, the Phase 3 unlock is **routing paid tiers via AWS Bedrock or GCP Vertex under the user's own cloud account**, which converts "resell OpenAI" (illegal) into "use cloud partner credits" (already-solved).

---

## 4. Risks and mitigations

### 4.1 Merchant of Record (MoR)

- **Stripe Connect is NOT MoR.** The user is the merchant; Stripe is the processor. We're responsible for tax registration in jurisdictions where we have nexus. At Caravansary's scale (all our own subscription revenue, no marketplace passthrough), our tax footprint is small.
- **For our own subscription revenue ($19/mo paid tier),** we will use **Stripe directly** initially and migrate to **Polar** ([polar.sh](https://polar.sh/)) as Merchant of Record before EU customer volume forces VAT registration. Polar absorbs sales tax/VAT, chargebacks, refund processing, and is OSS-friendly. Cost: ~5% + $0.50, vs Stripe's ~2.9% + $0.30. The 2% premium is the price of not registering for VAT in 50 jurisdictions ourselves.
- **Alternatives:** Paddle (mature, B2B-focused), Lemon Squeezy (Stripe-acquired, ecosystem-aligned). All three are reasonable; Polar is the OSS-friendly current pick.

### 4.2 LLM abuse / content moderation liability

- **OpenAI / Anthropic position:** developer (us, on the master-key path; user, on BYOK) is responsible for end-user behavior. If we proxy, we answer for abuse on the master-key path.
- **EU AI Act (in effect 2025–2026 in phases):** if we're deploying a "general purpose AI system" in the EU, we're a deployer with obligations: transparency, logging, no prohibited use cases. BYOK + thin orchestration probably keeps us out of "provider" classification, but "deployer" still applies if we offer prebuilt prompts/agents.
- **US Section 230:** ambiguous for LLM output. Recent district-court signals suggest 230 may not shield generative output. **Treat LLM output as our speech, not user-generated content, for liability purposes** — the cautious posture.
- **Mitigations:**
  - Master-key path runs OpenAI moderation (free) on inputs and outputs.
  - Per-user daily $ ceiling on master-key spend; per-key second-level rate limit at 100x typical use; synthetic abuse canary.
  - Publish our own AUP that flows down vendor AUPs.
  - Carry tech E&O / cyber insurance by paid-tier launch.
  - Log enough to respond to abuse complaints; don't log raw prompts beyond need.

### 4.3 KYC / AML / money transmission

- We become a money transmitter only if we **receive funds and pay them out** to a third party. Stripe Connect Standard avoids this — Stripe pays the merchant directly.
- Marketplaces (we take payment from end-buyers and pay sellers) need either Stripe Connect's marketplace flow or state-level money-transmitter licenses. **We will not build the marketplace pattern.**
- **Free tier:** no money flows through us. **Paid tier:** Stripe Connect Standard → user's Stripe → user's bank for their `/v1/payment/*` activity. For our own subscription revenue: Stripe / Polar.

### 4.4 Data residency / GDPR / DPA chains

- BYOK helps: the user's prompt goes through our infra, but the vendor sees the request as the user's. We're a **processor** of the prompt en route; the user is the **controller**; the LLM vendor is a **sub-processor**.
- Master-key paths require us to stand in as both processor and controller, which is heavier.
- Required publications:
  - Sub-processor list naming Gemini/Groq/AWS/Cloudflare/etc.
  - DPA template for users.
  - Region-pinning for EU traffic by Phase 3.
- DPAs to verify:
  - Google Cloud DPA — https://cloud.google.com/terms/data-processing-addendum
  - OpenAI DPA — https://openai.com/policies/data-processing-addendum/
  - Anthropic DPA — https://www.anthropic.com/legal/dpa
  - AWS DPA — https://aws.amazon.com/agreement/

### 4.5 Sender reputation (email)

- BYOK email or per-tenant subaccounts. Never one shared IP for many tenants on the master-key path.
- AWS SES from our account is fine for low-volume transactional mail (auth emails) so long as we enforce per-user verified domains for any sending beyond magic-link / OTP volume.

### 4.6 Open-source license compatibility

- **MIT / Apache 2.0** (Postgres, Better Auth, LiteLLM core, Lago) — safe to embed, redistribute, modify privately.
- **Redis** (post-RSALv2/SSPL switch in 2024, partial reversion in 2025) — confirm current license of the version we ship; if SSPL, we cannot offer Redis-as-a-service as part of our hosted product without releasing our management layer source. **Mitigation:** prefer **Valkey** (BSD), the Redis fork.
- **AGPLv3** (some Ollama dependencies, MongoDB Community) — if we offer the software as a network service, we must offer corresponding source. **Mitigation:** avoid embedding AGPL multi-tenant; isolate behind a clean network boundary.
- **Plausible / GlitchTip / PostHog** — confirm individual licenses (Plausible is AGPL; GlitchTip MIT; PostHog MIT). If we self-host Plausible as a backend for our `/v1/event/*` endpoint, we trigger the AGPL network-service obligation. **Decision:** for Plausible-style analytics, self-host **PostHog OSS** (MIT) instead. Update the SBOM accordingly.

---

## 5. Open questions for a real lawyer

Before we commercialize (paid tier launch), we need an attorney engagement that resolves:

1. **OpenAI/Anthropic BYOK custody:** can we hold the user's API key without becoming a "sub-processor" subject to their DPA? (Industry practice says yes; we want explicit confirmation.)
2. **EU AI Act deployer-vs-provider classification** for our master-key path on Gemini/Groq when we offer prebuilt prompts or agent templates.
3. **Stripe Connect Standard fee structure:** is `application_fee_amount` the right vehicle for our paid-tier upsell, or do we need a Connected Account Agreement amendment?
4. **Cloudflare Tenant API approval gating** — what's the realistic timeline? Worth applying at $0 stage to enable Mode B sooner.
5. **State-level money-transmitter exposure** if we ever take payment for vendor passthrough. Confirm Stripe Connect Standard fully avoids this in NY (BitLicense), CA, and other strict-license states.
6. **Section 230 / LLM-output liability:** what does our cyber/E&O policy cover, and what's the carve-out for generative output?
7. **Sub-processor disclosure obligations** to enterprise customers (most enterprise procurement now requires 30-day notice for sub-processor changes). Operationally feasible at our cadence?
8. **AI transparency disclosure laws** (California, Colorado, EU) — do we need to disclose "this output was AI-generated" at the API layer, the UI layer, or both?
9. **Data-deletion guarantee chain** — OpenAI honors deletion within 30 days for API data; Anthropic similar. We need to mirror this in our user-facing terms; confirm the chain holds across all our master-key vendors.
10. **The "substitutes for" clause in Google APIs ToS** as it applies to Mode A on Gemini.

---

## 6. The seamless-onboarding compromise, stated honestly

The seamless thesis says the user signs in once and never sees a vendor. The legal scoping says some endpoints (Stripe, GitHub, OpenAI BYOK) require a user-facing relationship with the vendor.

**The compromise:** we hide that relationship until the *exact moment* it's required.

- The user signs in. Sees one key. Free tier works against most endpoints with Mode A — they never see Gemini, Groq, AWS SES, our Cloudflare zone, our Fly org. To them, the LLM is "Caravansary's LLM."
- The first time the user calls `/v1/payment/checkout`, a Stripe Connect dialog appears in the response: "Confirm your payout details to take payments." That's two clicks; nothing else changes. The Caravansary key still works. The endpoint still has the same shape.
- The first time the user calls `/v1/git/*`, the GitHub App install link appears. Same pattern.
- Connected providers exist as power-user settings after onboarding. The user discovers them when they want to use their own vendor account, credits, or compliance posture.

**This compromise is the entire reason this document is long.** We're not hiding the legal complexity from ourselves — we're hiding it from the user. The product is not legally simpler than its competitors; it is *experientially* simpler. That is the value.

---

## Sources

- [OpenAI Business Terms](https://openai.com/policies/business-terms/) · [Usage Policies](https://openai.com/policies/usage-policies/) · [Startup Program](https://openai.com/form/startup-program/)
- [Anthropic Commercial Terms](https://www.anthropic.com/legal/commercial-terms) · [AUP](https://www.anthropic.com/legal/aup)
- [Google Gemini API ToS](https://ai.google.dev/gemini-api/terms) · [Google APIs ToS](https://developers.google.com/terms) · [GenAI Prohibited Use Policy](https://policies.google.com/terms/generative-ai/use-policy)
- [Stripe Connect](https://stripe.com/connect) · [Connect account types](https://docs.stripe.com/connect/accounts) · [Connected Account Agreement](https://stripe.com/legal/connect-account)
- [AWS Customer Agreement](https://aws.amazon.com/agreement/) · [AWS APN](https://aws.amazon.com/partners/) · [Solution Provider Program](https://aws.amazon.com/partners/programs/solution-provider/) · [Activate](https://aws.amazon.com/activate/)
- [GCP ToS](https://cloud.google.com/terms/) · [Partner Advantage](https://cloud.google.com/partners) · [GCP for Startups](https://cloud.google.com/startup)
- [Cloudflare Subscription Agreement](https://www.cloudflare.com/terms/) · [For Startups](https://www.cloudflare.com/forstartups/) · [Tenant API](https://developers.cloudflare.com/tenant/)
- [Fly.io ToS](https://fly.io/legal/terms-of-service/) · [Fly.io AUP](https://fly.io/legal/acceptable-use/)
- [Vercel ToS](https://vercel.com/legal/terms) · [Integrations](https://vercel.com/integrations)
- [Twilio MSA](https://www.twilio.com/legal/tos) · [Twilio AUP](https://www.twilio.com/legal/aup) · [ISV program](https://www.twilio.com/partners/isv)
- [Resend Terms](https://resend.com/legal/terms-of-service) · [Resend AUP](https://resend.com/legal/acceptable-use)
- [Postmark EULA](https://postmarkapp.com/eula) · [Postmark Sending Policy](https://postmarkapp.com/support/article/1078-what-is-postmark-s-sending-policy)
- [GitHub ToS](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service) · [GitHub Apps](https://docs.github.com/en/apps)
- [Polar.sh](https://polar.sh/) · [Paddle](https://www.paddle.com/) · [Lemon Squeezy](https://www.lemonsqueezy.com/) · [Best MoR providers 2026](https://grow.cleverbridge.com/blog/top-merchant-of-record-providers-2026)
- [OpenAI DPA](https://openai.com/policies/data-processing-addendum/) · [Anthropic DPA](https://www.anthropic.com/legal/dpa) · [GCP DPA](https://cloud.google.com/terms/data-processing-addendum)
