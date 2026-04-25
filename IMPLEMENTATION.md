# IMPLEMENTATION.md - Caravansary

*How we build toward the seamless product without letting implementation details leak into the first-run experience.*

April 2026. Pre-alpha planning doc.

---

## 0. Implementation anchor

The implementation exists to protect the product promise:

> **Sign in once. Ship.**

The key page is the proof that the account exists. The CLI is allowed to make the promise shorter by turning that account into a working repository without another dashboard, another vendor login, or another pasted secret.

Every phase should reduce the distance between idea and first working call. If a technical milestone makes the backend more correct but makes the user's first five minutes worse, it is not ready for the main path.

---

## 1. Phase guidance

### Phase 0: prove magic, not breadth

Ship the smallest set of endpoints that can honestly deliver the emotional promise:

1. LLM chat through safe free/provider routes.
2. Transactional email with strict verification and dev-mode fallback.
3. File/object storage.
4. Error or event capture through self-hosted infrastructure.
5. Payment checkout in test/simulation mode, with live activation just-in-time.

Do not launch arbitrary container hosting, broad DNS mutation, or unconstrained email until abuse controls are already boring.

### Phase 1: make setup one-shot

Add the developer tooling that turns a Caravansary account into a working project:

- CLI install.
- `caravansary init`.
- Browser/device-code auth that reuses the web session.
- Framework-aware examples.
- Official `@caravansary/mcp` server.
- GitHub Actions workflow generation.
- Environment variable wiring.
- Local smoke test that proves the key works.

The CLI can be an onboarding path if it improves the promise to "sign in once, ship." It must onboard the repository, not interrogate the user about infrastructure.

### Phase 2: make graduation invisible

Add connected providers, BYOK, provider preferences, paid limits, and activation flows without changing the SDK or endpoint shape.

The paid user should feel like the same product got stronger, not like they migrated to a different product. Their project still has only one runtime secret: `CARAVANSARY_API_KEY`.

### Phase 3+: earn true consolidation

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

## 2. CLI doctrine

The web onboarding is the product's front door. The CLI is the fast lane after the key exists.

The command should be:

```sh
caravansary init
```

Use the full name. `crvs` or `carvs` can exist as aliases later, but the primary command should teach the brand and avoid ambiguity while the product is young.

Doctrine:

> **Web onboards the person. CLI activates the project.**

`caravansary init` should:

1. Detect the project type.
2. Open the browser or device-code flow to confirm the existing Caravansary session.
3. Retrieve or mint the project key through a short-lived bootstrap token.
4. Write the smallest safe environment file change.
5. Install the SDK only when the project stack has an obvious package manager.
6. Add one runnable example route/script.
7. Offer to add GitHub Actions.
8. Run a smoke test.

It should not ask the user to choose vendors, regions, models, log retention, billing plans, or rate limits. It may ask what local files it is allowed to touch.

Manual key entry is only an escape hatch:

```sh
caravansary init --key csk_...
```

Use it for CI, headless machines, browser-auth failure, or users who explicitly want copy/paste. It is not the main path.

---

## 3. One-shot setup target

The target flow:

```sh
npm create next-app my-thing
cd my-thing
caravansary init
npm run dev
```

During `init`, the CLI opens the browser, confirms the already signed-in user, retrieves the Caravansary project key, writes it locally, and proves it works.

After that, the project has:

- `CARAVANSARY_API_KEY` in the local env file.
- A tiny server-side example that calls `/v1/llm/chat`.
- Optional examples for email, file storage, events, and payment test checkout.
- A generated smoke test.
- Optional GitHub Actions workflow with the secret name documented.

The user can delete the example files after they learn the shape. The important thing is that the first successful call happens while momentum is still alive.

---

## 4. GitHub Actions workflow

The generated workflow should be boring and safe:

- run tests,
- run a Caravansary smoke test when `CARAVANSARY_API_KEY` is present,
- never print the key,
- never call live-money endpoints,
- never send unrestricted email,
- fail with a clear message when the secret is missing.

The workflow should prove the integration, not become a deployment platform in disguise.

---

## 5. MCP server

MCP is a first-class product surface, not an afterthought.

The official MCP server should be:

```sh
npx @caravansary/mcp
```

or installed by:

```sh
caravansary init --mcp
```

It exposes Caravansary categories as agent tools:

- `llm.chat`
- `email.send`
- `file.put`
- `event.track`
- `payment.checkout.test`
- later: `queue.push`, `flag.check`, `dns.record`, `host.deploy`

The MCP server uses the same `CARAVANSARY_API_KEY` as the SDK and REST API. It does not ask the agent developer for OpenAI, Stripe, Resend, GitHub, AWS, or Cloudflare secrets.

The agent builder experience should be:

1. Sign in to Caravansary.
2. Run `caravansary init --mcp` or add the hosted MCP endpoint.
3. Tell the agent "use Caravansary."
4. The agent can email, store files, create test checkouts, call LLMs, and track events through one credential.

MCP tool permissions must be scoped by default. A project key can allow `llm.chat` and `file.put` while denying `payment.checkout` or `email.send`. The permission model belongs to Caravansary; it should not leak vendor IAM to the user.

---

## 6. Connected providers and integrations vault

In later phases, Caravansary becomes the place where users connect third parties:

- OpenAI / Anthropic / Gemini.
- Stripe.
- Resend / Postmark.
- GitHub.
- Vercel / Cloudflare / Fly.
- AWS / GCP.
- Slack / Notion / Linear.
- MCP tools.

These are control-plane credentials, not project credentials. The user's app does not receive `OPENAI_API_KEY`, `STRIPE_SECRET_KEY`, `RESEND_API_KEY`, `GITHUB_TOKEN`, or cloud access keys. It receives only:

```env
CARAVANSARY_API_KEY=...
```

The integrations vault exists to raise limits, use the user's credits, enable live modes, satisfy compliance, and reduce lock-in anxiety. It must never become first-run setup.

---

## 7. Skeleton examples

The skeleton should cover common intent, not every provider:

- "Ask the model" -> `/v1/llm/chat`
- "Send a dev email" -> `/v1/email/send`
- "Store a file" -> `/v1/file/put`
- "Track an event" -> `/v1/event/track`
- "Create a test checkout" -> `/v1/payment/checkout`

Each example uses the same key, the same base URL, and the same SDK client. The code teaches the category abstraction. It should never mention Gemini, Groq, Resend, Stripe, or Cloudflare unless the user asks what happened under the hood.

---

## 8. Review checklist for every implementation milestone

Before shipping a milestone, answer:

1. Does this reduce time-to-first-working-call?
2. Does it preserve sign in -> ship?
3. Does it avoid provider choice on the main path?
4. Does it keep the user's code stable across backend mode changes?
5. Does it fail safely without surprise bills or live side effects?
6. Does it make abuse controls stronger without making onboarding worse?
7. Does it teach Caravansary's abstraction instead of a vendor's dashboard?
8. Does the user's project still need only `CARAVANSARY_API_KEY`?

If the answer to #1 or #2 is "no," the milestone belongs off the main path.
