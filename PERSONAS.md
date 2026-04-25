# PERSONAS.md — Caravansary

*Who we're for. Who we're not. What they're trying to ship before they give up.*

The product is a love letter to a specific archetype: **the developer who has started seven side projects this year and shipped zero, because every single one died at the API-key onboarding stage.** Everything in the product points at this person. Other personas may benefit; they are not the design target.

---

## P1 — The Saturday Builder *(primary)*

> *"I got the idea at 9am. By 1pm I'd signed up for OpenAI, Stripe, Resend, Cloudflare, Vercel, and Sentry, generated six API keys, and not written a single line of business logic. By 4pm I'd given up and watched Netflix."*

**Who they are.**
A senior engineer at a real company by day. Saturday, they have an idea. They want to build it before Sunday night. They have done this fifty times. They are very good at the "build it" part. They are very bad at the "fight twenty signup forms" part — not because they are incapable but because it kills the dopamine before the product ever gets off the ground.

**Stack reality.**
- Reaches for tools they have an account on already, even when those tools are wrong for the job.
- Refuses to add a credit card to a service they're trying for the first time.
- Has 14 abandoned `npm create next-app` projects in `~/dev`.
- Owns 2-3 unused domains they bought during similar Saturdays.
- Has hit the "Stripe wants to verify your business" wall and never come back.
- Has hit the "OpenAI wants $5 minimum to top up" wall and never come back.

**What they want from Caravansary.**
1. To click "Sign in with GitHub," see a key, copy it, and `curl` something within 90 seconds.
2. To have working email, payments, LLM, and DNS available from that key without reading any vendor's docs.
3. To be able to ship the side project without having paid us a cent or having handed any vendor a credit card.
4. To be able to *delete the whole thing* on Sunday night when the idea wasn't as good as they thought, with one click, and no $4.62 trial-charge surprise next month.

**What they don't want.**
- A "set up your team workspace" wizard.
- A "name your first project" prompt.
- An onboarding checklist with green ticks.
- A welcome email asking them what they're building.
- A "limited-time founder-tier discount" upsell on day one.

**What "won" looks like for them.**
Their crap weekend project is live on a `*.caravansary.app` subdomain by Sunday afternoon. They didn't pay anyone. They didn't paste any vendor keys. They tweet about it. Two of their friends sign up because of the screenshot.

---

## P2 — The Agent Builder *(primary)*

> *"My agent needs to send emails, look up prices, charge cards, and store memory. Each capability is one signup. By the time the plumbing is done my customer has lost interest."*

**Who they are.**
Building an AI agent or a small AI-first product. Often using Claude Code, Cursor, or similar. Their agent's value prop is doing real work in the real world — which means it touches payments, email, calendars, files. They live on the edge of MCP / function-calling / tools.

**Stack reality.**
- Currently lives in JSON config hell, pasting keys into `.env.example` files.
- Has a half-built MCP server for each capability they wanted to give the agent.
- Has been bitten by an OpenAI rate limit at exactly the wrong moment in a customer demo.
- Wants the agent to be able to "do anything" but is bottlenecked on adapter plumbing.

**What they want from Caravansary.**
1. One credential the agent carries. Every tool the agent needs (`/v1/email/send`, `/v1/payment/checkout`, `/v1/llm/chat`, `/v1/file/put`) responds to that one credential.
2. A `caravansary` MCP server published to the major MCP registries, so any MCP-aware agent can be told "sign in to Caravansary" once and instantly gain a bag of capabilities.
3. Per-action audit log so when the agent does something stupid in production, they can find it.
4. The ability to scope a key to "this agent can email, but cannot charge cards" without having to think about IAM.

**What they don't want.**
- To explain to their customer why the agent went down because OpenAI changed a tier limit.
- To rotate twenty vendor keys when an employee leaves.
- To re-implement billing for the eighth time.

**What "won" looks like for them.**
They demo their agent. The agent emails a quote, sends a payment link, files a note in storage, and pings them in Slack — all through one Caravansary key. The customer signs the contract.

---

## P3 — The "I Just Wanted to Try It" Engineer *(primary)*

> *"I had 30 minutes during lunch. I'd heard about a new open-source vector DB and wanted to see if it works. The signup wanted my company name, my use case, and my credit card. I closed the tab."*

**Who they are.**
Hands-on engineer, the kind who reads HN every morning. Skeptical of marketing. Will try a new tool only if it works in under 5 minutes; will become an evangelist if it does. Currently: closes more tabs than they open.

**Stack reality.**
- Has a personal `~/dev/scratch` where they hack throwaway things.
- Their bar for "a tool is worth my time" is "does it have a curl-able demo on the homepage."
- Burned by enough "Sign up to access the docs" walls that they now reflexively close them.

**What they want from Caravansary.**
1. To use the example `curl` from the docs without signing up at all (we'll have a public demo key with very tight limits).
2. To upgrade to a real key in under 30 seconds when they're convinced.
3. To never see a Cal.com booking link inside any of our flows.

**What "won" looks like.**
They're back the following weekend. They keep the key. They stay free for six months. Then they ship a real thing and become a $19/mo customer.

---

## P4 — The Bootstrap Founder *(secondary)*

> *"I'm building this in my garage. Every dollar matters. I have no time for sales calls, and I don't qualify for any of the startup credit programs because I'm not VC-backed."*

**Who they are.**
Solo founder or 2-person team. Self-funded. Has revenue ambitions, not unicorn ambitions. Annoyed by the entire startup-credit-perk industrial complex because it requires a YC application or $250k of VC to access most of it.

**Stack reality.**
- Already running their MVP in production for paying customers.
- Has 5-15 vendor relationships, each with its own bill arriving on a different day of the month.
- Wants to consolidate but doesn't have the time to migrate.

**What they want from Caravansary.**
1. Drop-in for at least three of their existing vendors (e.g. email, storage, payments — picked by what they happen to use, not by what we'd like to demo) so the bills consolidate.
2. BYOK at zero markup so they can keep their existing vendor pricing.
3. One place to understand spend, and eventually a single invoice where reseller or MoR agreements make that possible.
4. Eventual Merchant-of-Record support so they don't have to register for VAT in 47 places.

**What they don't want.**
- A migration project.
- Vendor lock-in.
- Marketing emails.

**What "won" looks like.**
They've consolidated 5 vendor dashboards into one operational view, kept the same vendors via BYOK, and are paying us $19/mo for the privilege of one key and one control plane. They tell other founders.

---

## P5 — The Curious Lurker *(secondary)*

> *"I'm not a developer, exactly, but I can copy-paste curl. I want to play with AI without giving Sam Altman my credit card."*

**Who they are.**
The new wave of "developer-adjacent" people: PMs who can prompt, ops people who script, founders who Cursor their way through MVPs. They are not the marketing target *yet* but they are a leading indicator. If they can use the product, the developer audience can definitely use it.

**Stack reality.**
- Comfortable with copy-paste-curl and Postman.
- Less comfortable with "set up an AWS IAM role."
- Very put off by anything that looks like "real" devops onboarding.

**What they want from Caravansary.**
A frictionless way to make HTTP requests against modern AI / payment / email infrastructure without first becoming a software engineer.

**Why they matter.**
The "Saturday Builder" of 2027 is the curious lurker of 2026. We are not optimizing for them, but if our onboarding is hostile to them, we are over-rotated on developer machismo.

---

## Anti-personas (who we are explicitly NOT for, in Phase 1)

### A1 — The Enterprise Buyer
> *"We need SOC 2 Type II, EU data residency, SAML SSO, custom DPAs, role-based access control, an MSA, and a dedicated CSM. Can we get on a call?"*

**Why we say no, in Phase 1.** Enterprise procurement cycles are 6-9 months. Caravansary's whole thesis is killing the slow path. Saying "yes" to enterprise in Phase 1 would warp the product around audit checkboxes and force us to introduce concepts (workspaces, RBAC, custom contracts) that hurt the Saturday Builder. Phase 3 may revisit. We will not pretend otherwise on the marketing site.

### A2 — The Compliance-First Industry
> *"We're in healthcare / finance / government. Are you HIPAA / PCI / FedRAMP?"*

**Why we say no, in Phase 1.** No, and we will not be soon. The free tier proxies LLM calls through our master key, which is incompatible with HIPAA's BAA chain for many vendors. The compliance roadmap exists; it does not start in Phase 1.

### A3 — The "I Want to Build This Myself" Architect
> *"I'm going to build my own gateway with LiteLLM, my own router, my own metering, and use vendor APIs directly."*

**Why we don't try to convert them.** Good for them. They're right about a lot of things. They are also not our customer — they have the time and skill to absorb the twenty-signups tax in exchange for control. We are for people who do not have, or refuse to spend, that time.

### A4 — The Crypto / Web3 Builder
> *"Can I onboard with WalletConnect? Can the API key be a token?"*

**Not now.** We sign in with Google or GitHub because the developer audience already has those identities and OAuth-with-Google is the *single most frictionless authentication pattern in the world.* WalletConnect is a worse onboarding experience for our target persona; it adds the wallet-install step that they wanted to avoid. Crypto-native sign-in becomes interesting only if a meaningful fraction of users ask for it.

---

## What unifies the primary personas

Every primary persona has the same root frustration, in slightly different language:

> *"I have done the work to know what I want to build. The thing in my way is not the work. The thing in my way is the bureaucracy that surrounds the work."*

Caravansary's job is to make that bureaucracy disappear so completely that the user never even knows it was supposed to be there. The Saturday Builder doesn't think "wow, I avoided 20 signups" — they just ship a thing on a Saturday and move on. The Agent Builder doesn't think "wow, I avoided IAM hell" — their agent just works. That invisibility is the design goal.

We will know we have built the right thing when users **forget that the alternative existed.** When they cannot remember the last time they pasted an OpenAI key into a `.env` file. When the next time they're asked to sign up for a Resend account, they're confused — *isn't email just a thing that works?*

Yes. Email is just a thing that works. So is everything else.

That's what's in the walls of the caravansary.

---

## Use cases, prioritized

| # | Use case | Persona | Phase |
|---|---|---|---|
| 1 | Send transactional email from a side project without signing up for an email provider | P1, P5 | 0 |
| 2 | Charge a card for a paid feature without building a Stripe integration first | P1, P2 | 0 (test checkout immediately; Stripe activation just-in-time for live money) |
| 3 | Call an LLM in your code without picking a provider or model | P1, P2, P3, P5 | 0 |
| 4 | Give an AI agent a single credential that grants email + payments + storage + LLM | P2 | 0 |
| 5 | Ship a Postgres-backed app without provisioning Postgres yourself | P1 | 1 (planned, hand-off to nlqdb-style abstraction) |
| 6 | Track product analytics events without signing up for PostHog/Mixpanel | P1 | 1 |
| 7 | Capture errors without signing up for Sentry | P1 | 1 |
| 8 | Consolidate 5 vendor invoices into 1 (via BYOK) | P4 | 2 |
| 9 | Region-pin all data and compute to EU | P4 | 3 |
| 10 | SAML SSO + audit trail + DPA | A1 (anti-persona) | not on roadmap |
