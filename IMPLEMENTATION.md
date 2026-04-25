# IMPLEMENTATION.md - Caravansary

*How we build toward the seamless product without letting implementation details leak into the first-run experience.*

April 2026. Pre-alpha planning doc.

---

## 0. Implementation anchor

The implementation exists to protect the product promise:

> **Sign in once. Copy one key. Ship.**

Every phase should reduce the distance between idea and first working `curl`. If a technical milestone makes the backend more correct but makes the user's first five minutes worse, it is not ready for the main path.

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

### Phase 1: make setup feel one-shot

Add the developer tooling that turns the API key into a working project:

- CLI install.
- `caravansary init`.
- Framework-aware examples.
- GitHub Actions workflow generation.
- Environment variable wiring.
- Local smoke test that proves the key works.

The CLI must never become a second onboarding wizard. It should be a project bootstrapper for users who already have a key.

### Phase 2: make graduation invisible

Add BYOK, provider preferences, paid limits, and activation flows without changing the SDK or endpoint shape.

The paid user should feel like the same product got stronger, not like they migrated to a different product.

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

`caravansary init` should:

1. Detect the project type.
2. Ask for the Caravansary key only if it cannot find one.
3. Write the smallest safe environment file change.
4. Install the SDK only when the project stack has an obvious package manager.
5. Add one runnable example route/script.
6. Offer to add GitHub Actions.
7. Run a smoke test.

It should not ask the user to choose vendors, regions, models, log retention, billing plans, or rate limits.

---

## 3. One-shot setup target

The target flow:

```sh
npm create next-app my-thing
cd my-thing
caravansary init
npm run dev
```

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

## 5. Skeleton examples

The skeleton should cover common intent, not every provider:

- "Ask the model" -> `/v1/llm/chat`
- "Send a dev email" -> `/v1/email/send`
- "Store a file" -> `/v1/file/put`
- "Track an event" -> `/v1/event/track`
- "Create a test checkout" -> `/v1/payment/checkout`

Each example uses the same key, the same base URL, and the same SDK client. The code teaches the category abstraction. It should never mention Gemini, Groq, Resend, Stripe, or Cloudflare unless the user asks what happened under the hood.

---

## 6. Review checklist for every implementation milestone

Before shipping a milestone, answer:

1. Does this reduce time-to-first-working-call?
2. Does it preserve sign in -> key -> endpoint?
3. Does it avoid provider choice on the main path?
4. Does it keep the user's code stable across backend mode changes?
5. Does it fail safely without surprise bills or live side effects?
6. Does it make abuse controls stronger without making onboarding worse?
7. Does it teach Caravansary's abstraction instead of a vendor's dashboard?

If the answer to #1 or #2 is "no," the milestone belongs off the main path.
