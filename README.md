# Caravansary

> One signup. Every vendor.
> The walled compound for your dev stack.

Two user actions. That's the whole product:

1. **Sign in** with Google or GitHub.
2. **Copy your key** or run `caravansary init`.

That's it. No setup wizard. No provider picker. No model selector. No region wheel. No "create your first project." No "verify your email." No card. No second screen.

You land on a page that already has a working API key on it. Copy it, delete it, or let the CLI retrieve it through your web session. Use it to call any category we abstract — domain and DNS, transactional email, file storage, error tracking, product analytics, payments, container hosting, LLMs — through a single endpoint, or expose the same capabilities to agents through the Caravansary MCP server. We pick the provider. We rotate the key. We absorb the rate limits. We give you one place to understand usage and, when you pay us, one Caravansary bill.

This is not an LLM gateway. It is one API key replacing twenty, across every category of infrastructure a project needs. LLMs happen to be one of those categories.

You ship. We deal with the twenty dashboards.

## Status

Pre-alpha. Planning phase. April 2026.

- [PLAN.md](./PLAN.md) — what we're building, in what phases, with what business model. Read this first.
- [PERSONAS.md](./PERSONAS.md) — who we're for. The "started seven side projects this year, finished zero" developer.
- [COMPETITORS.md](./COMPETITORS.md) — landscape across LLM gateways, integration aggregators, BaaS, identity. Where the white space is.
- [LEGAL.md](./LEGAL.md) — what we can resell, what needs partnership, what stays BYOK. The honest legal scoping.
- [RISKS.md](./RISKS.md) — what can kill the product, and how we route around it without compromising the developer experience.
- [IMPLEMENTATION.md](./IMPLEMENTATION.md) — how we phase the build while keeping onboarding, CLI, and setup paths ruthlessly short.

## The thesis

The signup tax is the bug.

A modern indie developer who wants to ship a side project today opens twenty browser tabs. Cloudflare. Neon. Upstash. Stripe. OpenAI. Anthropic. Resend. Sentry. Grafana. Fly. Vercel. GitHub. The first hour is signup forms. The second hour is finding the dashboard. The third hour is generating tokens, naming tokens, copying tokens, pasting tokens, rotating tokens, losing tokens, regenerating tokens, pasting tokens. The fourth hour is the first `console.log`.

It is a bad joke that this is the state of the art.

A real caravansary on the Silk Road was a walled inn spaced one day's journey from the next. Inside the gate: water, bread, fodder, currency exchange, fresh camels, a doctor, a notary, a guard. You arrived dusty, you slept, you left at dawn with everything you needed for the next leg. You did not negotiate with the well-keeper, the granary clerk, the moneychanger, the stable boy, the apothecary, and the captain of the watch as separate vendors. You paid one rate at the gate.

That's what this is. One gate. One bell. Twenty doors handled inside.

## Pricing

Free tier: $0. No card. Forever.

Generous enough to ship a real side project on. When you outgrow it, the upgrade is one button — not a contract negotiation. Paying customers get raised limits, connected-provider passthrough at zero markup, and an actual SLA. Their apps still carry only `CARAVANSARY_API_KEY`; vendor keys and OAuth grants live in Caravansary's control plane. There is no third tier and there is no enterprise sales motion you have to dodge.

## License

TBD — leaning Apache-2.0 for SDKs, source-available for the gateway core.
