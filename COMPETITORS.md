# COMPETITORS.md — Caravansary

*The landscape, what each player gets right, and the white space we're aiming at.*

April 2026. Compiled from public pricing pages, vendor docs, HN discussion, and analyst writeups. Citations inline.

The thesis the rest of this document defends: **no existing product unifies the developer's runtime infrastructure across categories with a one-signup-one-key-zero-config experience.** The cousins exist. None of them have done it.

---

## The thirty-second summary

There are five adjacent categories, each with a $1B-or-pretending-to-be leader:

1. **LLM gateways** — OpenRouter is the king here. Solves "20 LLM keys" but not "20 dev-tool keys." Single-category.
2. **Unified API for end-customer integrations** — Pipedream Connect, Composio, Nango, Merge.dev. Solve "your customer's apps integrate with X." Wrong direction; we're trying to solve "your developer's stack integrates with X."
3. **Multi-vendor-hiding app platforms** — Vercel, Railway, Render, Supabase. Solve "one console for hosting + DB + auth," but they each lock you into their console. They are the dashboards we are routing around.
4. **Identity-as-a-Service** — Clerk, WorkOS, Auth0. Federate identity. We use that pattern (Google/GitHub OAuth) but for one category only; we apply the same federation principle across all dev categories.
5. **Founder credit programs** — Stripe Atlas, AWS Activate, Google for Startups. The OG "one signup → many vendor relationships," but they activate at *incorporation time*, not *runtime*, and they hand you raw credentials, which puts you back in dashboard hell.

Caravansary is the runtime version of Stripe Atlas crossed with the seamlessness of OpenRouter, applied across every dev category, with an onboarding that fits in two screens.

---

## Comparison table

| Product | Category | What they unify | Pricing | Free tier | MoR? | BYOK? | Onboarding screens (estimated) |
|---|---|---|---|---|---|---|---|
| **OpenRouter** | LLM gateway | 300+ LLMs | ~5.5% margin on top-up | Some free models (low limits) | Yes | Yes (1M req free, then surcharge) | 3–4 |
| **Vercel AI Gateway** | LLM gateway | All major LLMs | 0% markup; $5 monthly included credit until paid top-up | $5 monthly included credit | Yes | Yes (0% markup) | 4–5 (gated by Vercel signup) |
| **Cloudflare AI Gateway** | LLM gateway | BYOK only | Core gateway free; persistent log storage capped by Workers plan | Yes | No (BYOK only) | Yes | 5+ (gated by Cloudflare signup) |
| **Portkey** | LLM gateway + obs | 200+ LLMs | Per-log; free tier + paid | Yes | No (BYOK middleware) | Yes | 3–4 |
| **LiteLLM (proxy)** | LLM gateway (OSS) | 100+ providers | OSS + self-host infra (~$200–500/mo) | Free (MIT) | No | Yes | Self-host = many |
| **Helicone** | LLM gateway + obs | 100+ models | Free 10k req/mo, $79/mo+ | 10k req/mo | No | Yes | 3 |
| **Together / Fireworks / Replicate / Groq Cloud** | Hosted inference | OSS / fine-tuned LLMs | Per-token + GPU/hr | Trial credits | Yes | No (single provider each) | 3 |
| **Pipedream Connect** | Unified API (embed) | 3,000+ APIs / 10,000+ tools | $29 / $79 / custom | 100 credits/mo | No | n/a (managed OAuth) | n/a (you embed in your app) |
| **Composio** | Unified API for agents | 500+ apps | ~$99 base + per-user | Free dev | No | n/a | n/a |
| **Nango** | Unified API | 700+ APIs | ~$250/mo cloud; OSS self-host | OSS | No | n/a | n/a |
| **Merge.dev** | Unified API | HRIS / ATS / CRM / accounting | Per-account | Limited | No | n/a | n/a (B2B vertical) |
| **Apideck** | Unified API | 200+ connectors | Per-active-consumer; from $599 | Trial | No | n/a | n/a |
| **Paragon** | Embedded iPaaS | 130+ apps | ~$500–3,000/mo | Demo | No | n/a | n/a |
| **Vercel** | App platform | Hosting + edge + KV + AI | $20/seat + usage | Hobby tier | Yes (within Vercel) | n/a | 4+ |
| **Railway** | App platform | Compute + DB | Usage; $5 trial | $5 trial | Yes | n/a | 4 |
| **Render** | App platform | Compute + DB | Per-service | Free static | Yes | n/a | 4 |
| **Fly.io** | App platform | VMs + Postgres | Resource-priced | Trial | Yes | n/a | 4 |
| **Supabase** | BaaS | Postgres + Auth + Storage | $0 / $25 / $599 | 500MB / 50k MAU | Yes | n/a | 4 |
| **Encore** | App framework | Deploy to user's AWS/GCP | OSS + paid Cloud | Yes | No | n/a | 5+ |
| **Convex** | BaaS | DB + functions + sync | Tiered | Hobby | Yes | n/a | 4 |
| **Clerk** | IDaaS | SSO providers | $0.02/MAU after 10k free | 50k MRUs (raised 2026) | No | n/a | 3 |
| **WorkOS** | IDaaS (enterprise) | SSO + SCIM + AuthKit | AuthKit free to 1M MAU; SSO $125/conn | 1M MAU on AuthKit | No | n/a | 3 |
| **Auth0** | IDaaS | OIDC/SAML | Often $30k+/yr enterprise | Free dev | No | n/a | 4+ |
| **Frontegg** | IDaaS (B2B) | SSO + multi-tenant | $0.02/MAU after 10k | 7,500 MAU | No | n/a | 4 |
| **Stytch** | IDaaS | Passwordless | $0 to 5k MAU; $249/mo+ | 5k MAU | No | n/a | 3 |
| **Stripe Atlas** | Founder ops | Incorp + EIN + bank + perks | $500 + $100/yr | n/a | Yes (perks) | n/a | Many; one-shot |
| **AWS Activate** | Credit program | AWS only | Self-serve $1k; up to $300k | Tiered | n/a | n/a | One application |
| **Google for Startups** | Credit program | GCP + AI credits | Up to $350k for AI | Tiered | n/a | n/a | One application |
| **Cloudflare for Startups** | Credit program | Cloudflare + Workers AI | Up to $250k | Tiered | n/a | n/a | One application |
| **Anthropic startup credits** | Credit program | Anthropic API | $25k, 12-mo expiry | n/a | n/a | n/a | Application |
| **Zapier** | iPaaS | 7,000+ apps | Task-based; $20+/mo | 100 tasks/mo | No | n/a | n/a (non-dev audience) |
| **Make** | iPaaS | 1,000+ apps | Op-based; $10.59/mo+ | 1,000 ops/mo | No | n/a | n/a |
| **n8n** | iPaaS (OSS) | 400+ apps | OSS + cloud €24+ | OSS | No | n/a | Self-host = many |

The "Onboarding screens" column is the column we care most about. Caravansary's target is **2.**

---

## Top-10 positioning, with citations

### OpenRouter
The largest LLM aggregator and the closest cultural cousin to "one signup, many vendors" — for the inference category alone. They are the merchant of record: developers top up credits, OpenRouter pays the upstream provider, takes ~5.5% on credit-card top-ups plus a surcharge on BYOK after 1M monthly requests. Strong on model breadth and discovery; weak on support and reliability per HN ([Going Rogue thread](https://news.ycombinator.com/item?id=47563884), [outage](https://news.ycombinator.com/item?id=45050428)). Recently raised $120M at a $1.3B valuation. Their monetization model is the template — be the MoR, take a small margin, throw a generous BYOK ramp at the top of the funnel ([pricing](https://openrouter.ai/pricing), [BYOK announcement](https://openrouter.ai/announcements/1-million-free-byok-requests-per-month)).

**What we learn from them.** The MoR-with-BYOK-graduation model works at scale. It also doesn't extend to non-LLM categories on its own — OpenRouter has shown no interest in routing email, payments, or storage. The wedge is open.

### Vercel AI Gateway
The most aggressive pricing in LLM gateways: 0% markup on tokens *including with BYOK* on the paid tier ([Vercel pricing](https://vercel.com/docs/ai-gateway/pricing), [BYOK docs](https://vercel.com/docs/ai-gateway/authentication-and-byok/byok)). Their leverage is that they're already the developer's hosting bill. We cannot beat 0% markup on raw tokens; we must compete on the breadth of categories Vercel does not touch (email, payments, monitoring, DNS) and on the seamlessness of the onboarding (Vercel still requires you to set up a Vercel team and project before you can use the gateway, which is several screens).

### Cloudflare AI Gateway
Free core gateway, but BYOK-only — they don't resell inference here. Logging is capped at 100k events/month free, gateway runs on Workers, where high-volume usage triggers normal Workers billing ([CF pricing](https://developers.cloudflare.com/ai-gateway/reference/pricing/)). Strategically interesting because Cloudflare is quietly assembling the same one-vendor-many-categories play we are — they're moving into transactional email ([source](https://pbxscience.com/cloudflare-takes-aim-at-transactional-email/)) and have first-class storage, DNS, edge compute, AI, and now mail. The difference: Cloudflare is bundling *under their own roof*. We're bundling *across roofs*. Cloudflare is also our infrastructure, not our competitor — until the day they decide it is.

### Portkey
Best-in-class on observability and guardrails for LLMs. Bills per recorded log; not a reseller — you pay OpenAI/Anthropic directly ([Portkey pricing](https://portkey.ai/pricing)). They are a *dev-tool layer*, not a "outsource your vendor relationships" play. Useful as a pattern for the *observability* and *fallback routing* features any aggregator should ship; not a direct competitor.

### Pipedream Connect
The most credible existing answer to "one onboarding for many APIs" outside the LLM category — 3,000+ APIs and 10,000+ tools, with managed OAuth, approved client IDs, and end-user account management ([connect docs](https://pipedream.com/connect)). $29/$79/custom plans.

**The orientation gap, and why it matters.** Pipedream is positioned for *embedding integrations into your customers' apps* — i.e., the developer using Pipedream is building a product whose end-users connect their Salesforce / Slack / HubSpot. We're inverted. The developer is the end user. They are the one who needs Stripe + OpenAI + Resend + Cloudflare to ship their thing. Pipedream's billing model (per active end-user) doesn't fit "the developer themselves." Their ToS doesn't fit. Their UI doesn't fit.

This is the largest semantic gap in the competitive landscape. Pipedream is excellent at "you embed integrations into your SaaS." Nobody is excellent at "you, the developer, never have to sign up for a vendor again."

### Composio
Closed-source agent SDK with 500+ connectors and managed auth, priced at roughly $99 base + per-external-user ([Composio overview](https://composio.dev/content/ai-agent-integration-platforms)). Designed for AI-agent use cases. Strong execution in a narrow lane; not a fit for non-agent developer flows like "I just want a Postmark account."

### Nango
Open-source under Elastic License, 700+ APIs, ~$250/mo cloud and free self-host ([Nango blog](https://nango.dev/blog/best-agentic-api-integrations-platform/)). The OSS posture is appealing to senior devs; the trade-off is that you get unified data sync and OAuth scaffolding, not unified billing or vendor-management. Closest "we'll handle the OAuth dance for you" company today, but again oriented at end-customer integrations.

### Merge.dev / Apideck / Unified.to
Category leaders for unified APIs in vertical B2B — HRIS, ATS, CRM, accounting, ticketing. Per-linked-account billing ([Merge vs Apideck](https://www.merge.dev/vs/apideck)). Excellent in their lane, irrelevant to our lane. We share zero customer overlap.

### Clerk / WorkOS
The two reference points for "one signup federates many systems" in identity. Clerk pushed its free tier to 50k MRUs in 2026, charging $0.02/MAU above 10k ([Clerk pricing analysis on WorkOS](https://workos.com/blog/clerk-pricing)). WorkOS made AuthKit free to 1M MAU and bills per enterprise SSO connection ($125 first connection, dropping with volume) ([Stytch vs Auth0](https://supertokens.com/blog/stytch-vs-auth0)). Both are *the* model for "one developer onboards once, many federated identities" — but only inside the auth domain.

**What we learn.** Clerk's onboarding (sign up → installable React component → working login) is roughly three screens, which is the closest existing product to our two-screen target. That UX should be the one to study most carefully. They don't extend across categories; we will.

### Stripe Atlas
Closest analog to a "founder operating system" — $500 one-time, you get incorporation, EIN, registered agent, $2,500 in Stripe credits, and a perks portal worth ~$50k+ across AWS, Mercury, Xero, and others ([Stripe Atlas overview](https://stripe.com/atlas)). It is *the* template: one signup, many vendor relationships *brokered* on the founder's behalf via partner-perks. But Atlas is incorporation-flavored, not runtime-flavored — once your company exists, Atlas stops mattering. Caravansary is the runtime version of Atlas — what Atlas would be if it kept providing value through the build-and-ship phase, not just the legal-formation phase.

---

## What about iPaaS?

Zapier, Make, n8n, Pipedream Workflows. Adjacent, not directly competitive.

- **They serve a different audience.** iPaaS is for non-developers ("connect Gmail to Notion when X happens"). Caravansary is for developers ("I want to call OpenAI from a `curl`").
- **Their abstraction is too high-level for us.** "Trigger → action" workflows are slower and less flexible than direct API access. Our users want a `curl` and a JSON response, not a flowchart.
- **They're worth watching for the OAuth-app-management patterns** — Pipedream Workflows in particular has made it cheap-and-easy to register OAuth apps against hundreds of vendors, which is a practical capability we need.

We do not, and probably never will, ship a workflow builder. If we ever do, that's a Phase-4 conversation.

---

## What about credit programs?

AWS Activate (~$1k self-serve, up to $300k for AI), Azure for Startups (~$150k), Google for Startups (~$350k for AI), Cloudflare for Startups (~$250k), Anthropic ($25k, 12-mo), OpenAI startup credits via partners. Stripe Atlas perks include $2.5k Stripe credits + $50k partner deals.

These are not competitors. They are **resources we will integrate.** A user who signs in to Caravansary and tells us "I'm pre-Series-A, less than three years old, sub-$1M ARR" should be able to one-click apply for the credit programs they qualify for, *through us.* We surface eligibility, we pre-fill applications, we track expirations.

This is a **white-space opportunity** in itself: nobody curates the startup-credit landscape with a "click here to apply" flow. Closest existing is [orbitmoney.io](https://orbitmoney.io/deals/blog/startup-credits) which lists them but doesn't broker.

---

## White space — what nobody is doing well

After surveying ~40 products, the honest answer:

> **No one occupies the full intersection of (a) one signup, (b) cross-category — LLMs *and* email *and* DNS *and* payments *and* monitoring, (c) one place to understand spend and, eventually, reseller/MoR-backed vendor consolidation, (d) BYOK opt-out so they can graduate, (e) developer-grade DX with SDKs and provisioning APIs, (f) two-screen onboarding.**

The closest things:
- **OpenRouter** — does (a)(b within LLMs)(c)(d) but is single-category.
- **Vercel / Cloudflare** — do (a)(b)(c)(e) but only within their own platform's surface area, and lock you in.
- **Pipedream Connect** — does (a)(b)(e) but is positioned for *your customers' integrations*, not *your dev stack*, and is not the MoR.
- **Stripe Atlas** — does (a)(b — via perks)(e) at incorporation time, but does not provision runtime services.
- **Manifold (RIP)** — historically tried multi-cloud-multi-SaaS provisioning but never crossed the chasm. The fact that they tried and failed is *informative* — the unit-economics shape (they were trying to be the MoR for AWS/GCP/Heroku, all of which have margins close to zero) is exactly what we are not doing.

**Specifically, here is what is genuinely open:**

1. **Cross-category MoR for runtime dev services.** Today a developer who wants OpenAI + Stripe + Postmark + Cloudflare DNS + Vercel hosting + Sentry has six contracts, six payment methods, six tax forms. OpenRouter solved this *only for inference*. Nobody has solved it across categories.

2. **Provisioning-as-API for the long tail.** Pipedream/Composio/Nango handle OAuth for *existing* accounts. None of them *create* accounts on the developer's behalf. A platform that calls Resend's signup API, Stripe Connect's onboarding, Cloudflare's account provisioning, etc. — that's a different layer, and it's the layer Caravansary aims at.

3. **Credit-pool unification.** A developer can stack AWS Activate + Azure for Startups + Cloudflare + OpenAI + Anthropic + Stripe Atlas perks and get $500k+ in credits ([source](https://orbitmoney.io/deals/blog/startup-credits)) — but each requires a separate application, separate dashboard, separate expiry tracking. A platform that aggregates eligibility and applies credits intelligently across spend is a real wedge — it's effectively a cashback program for dev tooling.

4. **Graceful BYOK off-ramp.** Vercel AI Gateway nailed this for LLMs (0% markup with BYOK so you stay even after you graduate). The same pattern across non-LLM vendors is unbuilt. The promise: "we are never cheaper than going direct, but we are never more expensive either — we charge a flat platform fee, not a margin on every vendor."

5. **Unified observability & invoice across vendors.** Helicone for LLMs, Datadog for infra, Sentry for errors, separate Stripe dashboards for revenue. A unified spend/usage/error dashboard across all dev vendors a project uses is genuinely missing.

6. **The "embedded for developers themselves" niche.** Embedded iPaaS (Paragon, Workato Embedded) and unified API (Merge, Composio) are positioned for *your customers using your product*. Nobody is positioned for *your developers using their own infra*. That semantic flip is the wedge.

7. **The two-screen onboarding floor.** Even the best product in this space (OpenRouter, Clerk) takes 3+ screens to working code. The literal "sign in → key page → done" doesn't exist anywhere we can find. This is not a small UX win; it is the entire product.

---

## How we win, concretely

The risks are real: being MoR for many vendors creates compliance and tax surface area; vendors (OpenAI, Stripe) may not love a reseller layer; lock-in fears mean BYOK has to be first-class. The opportunity is that **every other category has a $1B+ leader** — OpenRouter for LLMs, Clerk for auth, Vercel for hosting, Merge for unified API, Stripe Atlas for incorporation. The cross-category, runtime-MoR slot is conspicuously empty.

The plan to occupy it:

1. **Win on onboarding velocity, before anything else.** If we can demo "sign in → working `curl`" in 30 seconds and no competitor can demo it in under three minutes, we win the indie-dev mindshare game by inches per week.
2. **Stack free tiers, not VC money, to fund the free tier.** Other gateways (OpenRouter, Portkey) burn margin on free traffic; we burn other vendors' free quotas. Different cost structure, sustainable longer.
3. **Make the BYOK graduation invisible.** When a free user graduates to paid + BYOK, nothing in their code changes. Same key, same endpoint, same SDK. They go from "we're paying for them" to "they're paying us a platform fee" without friction. This is also the lock-in: every additional vendor they add to BYOK is one more reason not to leave.
4. **Treat partner programs as a Phase-3 unlock, not a launch requirement.** Cloudflare Partner Network, AWS APN, Twilio ISV, Vercel Marketplace all take months of approval. Apply early. Ship without them.
5. **Resist enterprise gravity.** Every dollar we earn from a side-project hobbyist is worth more than a dollar we earn from an enterprise pilot, because the side-project hobbyist tells fifteen people on Bluesky and the enterprise tells nobody.

---

## Sources

- [OpenRouter Pricing](https://openrouter.ai/pricing) · [OpenRouter BYOK guide](https://openrouter.ai/docs/guides/overview/auth/byok) · [1M free BYOK announcement](https://openrouter.ai/announcements/1-million-free-byok-requests-per-month)
- [OpenRouter HN — Going Rogue](https://news.ycombinator.com/item?id=47563884) · [OpenRouter outage HN](https://news.ycombinator.com/item?id=45050428) · [OpenRouter $120M raise](https://news.ycombinator.com/item?id=47643347)
- [Portkey pricing](https://portkey.ai/pricing) · [Portkey AI Gateway](https://portkey.ai/features/ai-gateway)
- [LiteLLM Pricing Calculator](https://docs.litellm.ai/docs/proxy/pricing_calculator) · [LiteLLM review (TrueFoundry)](https://www.truefoundry.com/blog/a-detailed-litellm-review-features-pricing-pros-and-cons-2026)
- [Helicone](https://www.helicone.ai/) · [Helicone — observability comparison](https://www.helicone.ai/blog/the-complete-guide-to-LLM-observability-platforms)
- [Vercel AI Gateway pricing](https://vercel.com/docs/ai-gateway/pricing) · [Vercel BYOK](https://vercel.com/docs/ai-gateway/authentication-and-byok/byok)
- [Cloudflare AI Gateway pricing](https://developers.cloudflare.com/ai-gateway/reference/pricing/) · [Cloudflare email send (PBX Science)](https://pbxscience.com/cloudflare-takes-aim-at-transactional-email/)
- [Together / Fireworks / Replicate comparison (Infrabase)](https://infrabase.ai/blog/ai-inference-api-providers-compared) · [Fireworks pricing](https://fireworks.ai/pricing)
- [Groq pricing](https://groq.com/pricing) · [Groq pricing 2026 (TokenMix)](https://tokenmix.ai/blog/groq-api-pricing)
- [Pipedream Connect](https://pipedream.com/connect) · [Pipedream pricing](https://pipedream.com/pricing)
- [Composio AI agent platforms 2026](https://composio.dev/content/ai-agent-integration-platforms) · [Composio vs Pipedream](https://composio.dev/composio-vs-pipedream)
- [Nango — best agentic API integrations 2026](https://nango.dev/blog/best-agentic-api-integrations-platform/) · [Nango / Composio toolkit](https://docs.composio.dev/toolkits/nango)
- [Merge vs Apideck](https://www.merge.dev/vs/apideck) · [Apideck pricing breakdown](https://www.apideck.com/blog/breaking-down-unified-api-pricing-why-api-call-pricing-stands-out) · [Top unified API platforms (Ampersand)](https://www.withampersand.com/blog/the-10-best-unified-api-platforms-in-2026)
- [Paragon vs Prismatic](https://albato.com/blog/publications/embedded-paragon-vs-prismatic-albato-embedded) · [Paragon pricing analysis (Merge)](https://www.merge.dev/blog/paragon-pricing) · [Paragon vs Workato Embedded](https://www.useparagon.com/competitor/paragon-vs-workato-embedded)
- [Vercel vs Netlify vs Railway 2026](https://getathenic.com/blog/vercel-vs-netlify-vs-railway-deployment) · [Railway vs Render 2026](https://thesoftwarescout.com/railway-vs-render-2026-best-platform-for-deploying-apps/) · [Railway vs Vercel](https://docs.railway.com/platform/compare-to-vercel)
- [Supabase pricing](https://supabase.com/pricing) · [Fly.io pricing](https://fly.io/docs/about/pricing/) · [Supabase alternatives (Encore)](https://encore.dev/articles/supabase-alternatives) · [Northflank pricing](https://northflank.com/pricing)
- [Clerk pricing analysis (WorkOS blog)](https://workos.com/blog/clerk-pricing) · [Clerk vs WorkOS](https://workos.com/compare/clerk) · [Frontegg alternatives 2026](https://www.scalekit.com/blog/frontegg-alternatives-b2b-ai-auth)
- [Stytch vs Auth0 (SuperTokens)](https://supertokens.com/blog/stytch-vs-auth0) · [Stytch pricing guide](https://supertokens.com/blog/stytch-pricing) · [Cheapest Auth0 alternatives 2026](https://supertokens.com/blog/cheapest-auth-alternatives)
- [AWS Activate credits](https://aws.amazon.com/startups/credits) · [Startup credits guide 2026 (Orbit)](https://orbitmoney.io/deals/blog/startup-credits) · [AWS Activate eligibility (SquareOps)](https://squareops.com/knowledge/who-is-eligible-for-aws-startup-credits-and-how-to-apply-2026-guide/) · [Free cloud credits 2026 (CloudKompas)](https://cloudkompas.com/blog/free-cloud-credits-for-startups-AWS-azure-google-cloud-oci)
- [Anthropic startup program 2026](https://aicreditmart.com/ai-credits-providers/anthropic-startup-program-how-to-get-25k-in-credits-2026/)
- [Stripe Atlas](https://stripe.com/atlas) · [Stripe Atlas pricing 2026](https://sparklaun.ch/compare/stripe-atlas) · [Stripe Atlas review 2026](https://startupsavant.com/service-reviews/stripe-atlas)
- [Zapier vs Make vs n8n 2026](https://devtoolpicks.com/blog/zapier-vs-make-vs-n8n-2026-solo-developers) · [n8n pricing 2026 (Miniloop)](https://www.miniloop.ai/blog/n8n-pricing-2026)
- [Resend pricing](https://nuntly.com/resend-pricing) · [Postmark vs Resend 2026](https://forwardemail.net/en/blog/postmark-vs-resend-email-service-comparison)
- [Best MoR providers 2026](https://grow.cleverbridge.com/blog/top-merchant-of-record-providers-2026) · [Best MoR for SaaS (Creem)](https://www.creem.io/blog/best-merchant-of-record-saas-2026)
- [Best AI gateways 2026 (LLM Gateway)](https://llmgateway.io/blog/best-ai-gateways) · [What is OpenRouter actually charging for](https://www.datastudios.org/post/openrouter-pricing-byok-routing-costs-and-cost-optimization-strategies-how-openrouter-actually-c)
