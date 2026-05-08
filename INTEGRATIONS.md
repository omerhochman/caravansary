# INTEGRATIONS.md — Caravansary

*How we get from 9 hand-coded categories to hundreds of integrations without losing "sign in once, ship."*

April 2026. Pre-alpha planning doc. The companion to [PLAN.md](./PLAN.md) and [IMPLEMENTATION.md](./IMPLEMENTATION.md).

---

## Reading note — what this is and isn't

This is the platform doc for the integration layer: how we structure adapters, how new vendors get added, who can contribute, and how we keep the seamless DX promise as the connector count grows from ~18 (Phase 0) toward "every dev-infra service worth fronting."

It is **not** a YAML connector spec. We considered it. Section 3 explains why we said no, with receipts.

It is opinionated about cutting features that the platforms-of-record (Zapier, Airbyte, Workato, Nango pre-2025) have shipped and explicitly regretted.

The single sentence at the top: **three nouns, one file per adapter, capabilities passed in, AI-authored as the primary on-ramp, and untrusted code never runs in our hot path.**

---

## 0. The constraints we're optimizing under

1. **One engineer for Phase 0–1.** Anything that requires a 5-person platform team is wrong.
2. **Cloudflare Workers runtime.** Millisecond budget per call. No Docker-per-adapter. No Python.
3. **Hundreds of vendors is the destination, not Phase 0.** Phase 0 ships ~18 hand-written adapters and proves the contract.
4. **The user's app must keep working when we change adapters under it.** Canon stability is non-negotiable.
5. **The user must be able to see what we did with their request.** Transparency is a DX requirement, not a debug feature.

Every choice below is downstream of these.

---

## 1. Three nouns

The vocabulary. If a feature can't be expressed as one of these three, it doesn't ship.

### 1.1 Category — the Published Language

Defined by us. Versioned (`email.send@v1`). Has a Zod input schema and a Zod output schema. Frozen once published. Adapters fulfill it. Customers program against it.

The canon stays **thin**: only fields ≥80% of vendors agree on. Everything else goes in `vendor.raw` on the response. We resist universal-superset growth aggressively (this is the Hohpe-recanted Canonical-Data-Model lesson — superset canons accumulate cost forever).

### 1.2 Vendor — the connection, defined once

Auth scheme (bearer / basic / signed / oauth), base URL, rate-limit budget, status URL, abuse contact. One file per Vendor. N adapters reference it.

This is the "App as first-class object" pattern from Pipedream. Resend the Vendor is defined once; Resend's `email.send` adapter, Resend's `email.events` adapter, etc., reference it. Adding a second category for an existing vendor is one file, not two.

### 1.3 Adapter — the Anti-Corruption Layer

A TypeScript module. Takes typed canon input + a capabilities object. Returns typed canon output + raw. Pure function — no globals, no `process.env`, no top-level imports of `fetch` or `crypto`.

```ts
// adapters/resend/email.send.ts
import { adapter } from "@caravansary/sdk";
import { resend } from "../../vendors/resend";
import { EmailSendV1 } from "@caravansary/categories";

export default adapter({
  vendor: resend,
  category: EmailSendV1,
  exec: async (ctx, input) => {
    const r = await ctx.http.post("/emails", {
      from: input.from, to: input.to,
      subject: input.subject, html: input.html,
    });
    return { id: r.id, raw: r };
  },
});
```

That's the whole shape. One file. One function. The category's Zod schema runs on `input` before `exec` and on the return before it goes to the customer — adapter authors don't validate by hand.

---

## 2. The capabilities object — no ambient authority

`ctx` is the only thing an adapter is allowed to touch.

```ts
type AdapterContext = {
  http:    ScopedFetch;   // pre-configured to the vendor's base URL + auth
  kv:      ScopedKV;      // tenant-scoped, adapter-namespaced
  secrets: ScopedSecrets; // read-only, vendor-scoped
  log:     ScopedLogger;  // structured, per-request
  now:     () => Date;    // injectable for tests
};
```

`ctx.http` only reaches the vendor's allowlisted hostnames. `ctx.secrets` only sees this vendor's master/BYOK credentials. `ctx.kv` cannot read another adapter's namespace. The lint rule against `import "node:*"`, `globalThis`, `process`, and bare `fetch` is part of CI; PRs that break it are auto-rejected.

This is the object-capability discipline (Mark Miller's *Robust Composition*) translated into a TypeScript convention. We're not running Wasm yet — we're enforcing the *shape* now so that swapping the runtime to V8 isolates or Wasm components later is mechanical, not architectural.

---

## 3. Why TypeScript + Zod, not YAML

The user-visible promise is "add a vendor in a few lines." The instinctive shape is YAML. We deliberately rejected it. Four reasons, ordered by force:

1. **Nango killed YAML in production on Dec 1, 2025.** They had `nango.yaml` with sync/action definitions, removed it in favor of `createSync()` / `createAction()` in TypeScript with Zod schemas, and published the migration rationale: "no type safety, custom-model syntax was bespoke and ungoogleable, scattered config files, weak IDE support, weak error messages." That is the canary, and it's the closest analogue to our problem.
2. **Greenspun gravity.** Every successful config language eventually grows conditionals, then variables, then templating, then a worse programming language. Terraform HCL, Helm, Ansible, GitHub Actions, Webpack — each one walked the path. There's no reason to think we'd avoid it.
3. **Airbyte's data point cuts the other way only at first glance.** Their low-code YAML CDK is the most-cited "hundreds of connectors via YAML" success — but ~10% of connectors fall back to Python in a Docker image, the YAML schema is hundreds of fields with bespoke `${{ }}` interpolation, and every connector ships as a container. None of that fits a Workers-runtime gateway.
4. **AI-codegen targets TypeScript better than YAML.** A Claude prompt that emits a TS+Zod adapter file gets type-checking and lint feedback for free. A prompt that emits YAML emits "valid-looking YAML that breaks at runtime."

The "few lines" promise is met not by YAML, but by **(a)** a declarative-HTTP helper (Section 4) that makes the simple case read like a manifest, and **(b)** an AI scaffolder (Section 11) that generates the adapter from vendor docs in one command.

---

## 4. The declarative HTTP helper — the 80% case

Most adapters are "translate the canon input into an HTTP request, parse the response field." We ship a helper for that:

```ts
// adapters/postmark/email.send.ts
export default adapter.http({
  vendor: postmark,
  category: EmailSendV1,
  request: {
    method: "POST",
    path: "/email",
    body: ({ from, to, subject, html }) => ({
      From: from, To: to, Subject: subject, HtmlBody: html,
    }),
  },
  response: {
    id: (r) => r.MessageID,
  },
});
```

That's the same `Adapter` contract, expressed declaratively. The helper compiles to a function that calls `ctx.http`. AI-codegen targets this shape. When a vendor needs more (multi-step request, custom signing, weird pagination), the author drops back to the imperative `exec` form. Same file, one type. No second-class citizens.

This is n8n's declarative-vs-programmatic split, narrowed: there's one shape (`adapter`) and one helper (`adapter.http`) — not a full DSL.

---

## 5. The runtime tiers

Customer code never knows which tier served the call. They call `POST /v1/email/send`. The router picks. The response envelope (Section 7) shows the tier for inspection.

| Tier | Author | Lives in | Runtime | Overhead | Trust |
|---|---|---|---|---|---|
| **Official** | Caravansary | `caravansary/integrations` (monorepo, in-tree) | In-process Worker | <5ms | First-party |
| **Partner** | Vendor-employed engineer | Same repo, vendor-owned subdir | In-process Worker | <5ms | Verified |
| **Community** | Anyone, via PR | Same repo, community subdir | V8 isolate (Workers for Platforms dispatch) | <30ms | Sandboxed |
| **Custom MCP** | User's own MCP server | User's infra | Streamable HTTP | <200ms | User-only |

Phase 0 ships only Tier 1. Phase 1 opens Tiers 2 and 3. Phase 2 ships Tier 4. We don't open community contribution before the contract has stabilized — Section 14.

The Wasm Component Model is the academically correct answer for Tier 3, and we'll migrate when WASI Preview 2 has first-class Workers support. Until then, V8 isolates are the pragmatic isolation boundary.

---

## 6. Schema evolution — protobuf discipline on JSON

Categories are versioned by URL: `email.send@v1`, `email.send@v2`. A new version is a new endpoint, not a flag.

Within a version: **additive only.** New optional fields, never. New required fields, no. Removed fields, no. Renamed fields, no. Field "tags" (semantic identity) are stable even though we serialize JSON.

Adapters declare which versions they satisfy: `category: EmailSendV1`. When `@v2` ships, old adapters keep serving `@v1` traffic until their owners port them. There is no "migration deadline" enforced on customers — `@v1` lives until the last adapter is gone.

**Tolerant reader inside the adapter** (consuming vendor JSON — ignore unknown fields, default missing ones, never blow up on the wire).
**Strict producer at our API surface** (validate every output against the canon Zod schema before it leaves our gateway). This is the post-RFC-9413 reading of Postel's Law: tolerance at the foreign boundary, strictness within our trust boundary.

---

## 7. Transparency — the response envelope

Every successful canon response includes:

```json
{
  "id": "msg_…",
  "_caravansary": {
    "vendor": "resend",
    "vendor_request_id": "…",
    "tier": "official",
    "category_version": "email.send@v1",
    "latency_ms": 142,
    "raw": { /* full vendor response */ }
  }
}
```

Errors mirror the shape: a canonical code (`timeout` | `auth` | `ratelimit` | `validation` | `server` | `unavailable` | `unknown`), a human message, and `_caravansary.raw` carrying the verbatim vendor error.

This is non-negotiable. The DX literature (Fagerholm-Münch on cognitive load; Stripe's investment in error transparency) and the unified-API failure cases (Merge.dev's silently-dropped Salesforce custom fields) both converge on the same answer: **never make the customer guess what we did with their request.** They can ignore the envelope; they can't if they need it.

---

## 8. Reliability primitives — in the router, not in adapters

Adapters do one thing: translate. Everything else lives in the router so that 200 adapter authors don't each reinvent it badly.

- **Per-vendor bulkhead.** Separate concurrency budget per vendor. A slow Resend doesn't drain the SES pool.
- **Per-tenant per-vendor concurrency cap.** One tenant cannot starve another by saturating a shared adapter pool. (Stripe + Cloudflare rate-limiting literature.)
- **GCRA token bucket per tenant per category.** Smooth, memory-efficient, no thundering-herd.
- **Idempotency keys end-to-end.** Customer-supplied or auto-derived (hash of tenant + category + normalized input + minute bucket). Forwarded to vendors that honor `Idempotency-Key`.
- **Adaptive concurrency limits, not static circuit breakers.** Post-Hystrix, the lesson is that fixed thresholds drift; we measure latency and adjust budgets live.
- **Timeouts derived from vendor p99.9, exponential retry with full jitter.**

Adapters can declare hints (`idempotent: true`, `retryable: ["5xx", "429"]`) but cannot implement these themselves. This keeps the adapter contract tiny.

---

## 9. Multi-vendor failover — only where the category permits it

Stateless and fungible (LLM chat, embeddings, email send, error capture, event tracking): the router picks healthy adapters and falls back on 5xx/429. Customer never sees the failover.

Stateful or non-fungible (payment intents, DNS records, host deploys, anything tied to a vendor account): no failover. Sticky to the chosen vendor for the lifetime of the resource.

Pinning ("always use Anthropic for chat") is a paid-tier feature, not a free-tier knob. Free-tier users get our routing; they get the envelope so they can see what happened; they can graduate.

---

## 10. Contribution path

One repo: `caravansary/integrations`. Tiers 1–3 live here.

A new adapter is one PR with three files:
- `vendors/<vendor>.ts` — only if the Vendor doesn't exist yet.
- `adapters/<vendor>/<category>.ts` — the adapter.
- `tests/<vendor>/<category>.fixtures.json` — recorded vendor traffic for replay tests.

CI gates, all machine-checked:
1. **Schema conformance.** Adapter input/output match the category's Zod schemas.
2. **Capability lint.** No globals. No `process.env`. No bare `fetch`. No filesystem. No `node:*`.
3. **Recorded-traffic replay.** Fixtures replay against the adapter; behavior is byte-stable.
4. **Latency budget.** Synthetic benchmark against the fixture must hit the tier's budget.
5. **Owner heartbeat.** `OWNERS` file declares an active maintainer with a working email.

Anything passing CI lands in Tier 3 (Community). Promotion to Partner requires the vendor to claim ownership; promotion to Official requires Caravansary to adopt it. Demotion is automated on staleness, breakage, or owner-heartbeat failure — Tier 3 adapters that fail real-traffic SLOs for 14 days move to a `deprecated` directory and are removed from the router.

---

## 11. AI-authored adapters — the primary contributor on-ramp

Nango ships an AI builder skill that has produced 200+ adapters in production. We do the same, with one ergonomic improvement.

```sh
caravansary scaffold resend --category email.send
```

The scaffolder:
1. Looks up the vendor's OpenAPI spec (from APIs.guru, the vendor's docs site, or a URL we pass it).
2. Drafts `vendors/resend.ts`, `adapters/resend/email.send.ts`, and a fixtures file.
3. Runs the contract tests.
4. Opens a PR.

The output is reviewable TypeScript, not a generated blob. The scaffolder is also published as an MCP tool, so any AI coding agent (Claude Code, Cursor, Codex) can invoke it. This is how a one-engineer Phase 1 reaches a hundred connectors without hiring.

We do **not** ship a no-code "Connector Builder UI" in Phase 0–1. (Airbyte's Builder UI is excellent; it is also a quarter of their engineering surface area.) The AI scaffolder is our equivalent at 1/100 the build cost.

---

## 12. The Custom MCP path — the long tail

Some integrations are private. A user's internal CRM. A bespoke SMTP relay. A research-lab inference endpoint. We don't want their adapter in our monorepo, and they don't want it there either.

A user can register a Caravansary-shaped MCP server URL as a private adapter scoped to their tenant:

```sh
caravansary integrations register \
  --category email.send \
  --vendor my-corp-smtp \
  --mcp-server https://mcp.mycorp.internal
```

Caravansary calls their MCP server with the canon input; their server translates and replies. Their app code stays `POST /v1/email/send` — same key, same endpoint, same shape. We aren't hosting their code; they aren't on our roadmap.

This is the answer to "how does anyone connect a custom integration." It's also why aligning our adapter contract with MCP's tool-and-schema model (Section 1.3, the Zod input/output schemas) is worth the discipline — we get the AI agent ecosystem, the registry, and the long-tail story for free.

---

## 13. Quality tiers — surfaced to customers

A public ladder, machine-checked, modeled on Home Assistant's Bronze→Platinum:

- **Bronze** — passes CI. Has an owner. Has fixtures. (Minimum bar; Tier 3 default.)
- **Silver** — handles auth recovery. Has been live ≥30 days with <1% error rate. Has structured error mapping.
- **Gold** — full canon coverage. Includes a streaming/long-poll test if the category has one. Has been live ≥90 days.
- **Platinum** — sub-budget p99. Has at least one Caravansary employee as a reviewer. Owner heartbeats weekly.

The customer-facing UI shows the tier of whichever adapter served their last call. Paid users can pin a category to a minimum tier (`gold or above`) — that's a one-click setting, not a dashboard. Free users get whatever's healthy.

---

## 14. What we deliberately don't ship

Each of these is a feature a comparable product has shipped, and each one would break the constraints in Section 0.

- **No YAML DSL of our own.** (Greenspun. Nango's death-of-YAML migration. Type safety wins.)
- **No Docker-image-per-adapter.** (Airbyte's pattern; latency-fatal in a request-time gateway.)
- **No universal data model that grows to fit every vendor.** (Merge.dev's tax. Canon stays thin; raw is always reachable.)
- **No policy DSL for scopes.** (AWS IAM is universally hated. Scopes are flat: `email.send`, `payment.checkout:read`.)
- **No per-adapter npm package.** (Bookkeeping debt. One repo, one PR.)
- **No untrusted code in our main Worker.** (Tier 3 sandboxes. Tier 4 is out-of-process.)
- **No no-code connector builder UI in Phase 0–1.** (AI scaffolder ships first; UI later if data demands it.)
- **No "everything is a plugin" core.** (Backstage's regret. First-party adapters use the same contract as community ones, but the *core* is not a plugin runtime — it's a router that happens to call functions.)
- **No mandatory migration when canon versions advance.** (`@v1` lives until its last adapter dies.)

If a future feature lands in this list, it doesn't ship until somebody can explain on the PR which constraint in Section 0 changed.

---

## 15. Phasing

### Phase 0 — Define and freeze the contract (concurrent with PLAN.md Phase 0)

- Specify Category, Vendor, Adapter as TypeScript types in `@caravansary/sdk`.
- Implement the in-process router.
- Write the 9 Phase-0 categories' Zod schemas (`email.send@v1`, `file.put@v1`, `dns.record@v1`, `error.track@v1`, `event.track@v1`, `payment.checkout@v1`, `host.deploy@v1`, `llm.chat@v1`, `llm.embed@v1`).
- Hand-write 1–2 Official adapters per category.
- No external contribution path. No AI scaffolder yet. No tiers UI.

### Phase 1 — Open the doors

- Open `caravansary/integrations` repo. Document the contract.
- Ship `caravansary scaffold` (the AI scaffolder), exposed as a CLI command and an MCP tool.
- Stand up Tier 3 (Community) on Workers-for-Platforms dispatch.
- Accept first Partner adapters (vendor-employed engineers contributing).
- Ship the response envelope (`_caravansary.*`) on every endpoint.

### Phase 2 — Custom MCP and quality tiers

- Ship the private-MCP-adapter registration flow.
- Ship the public Bronze/Silver/Gold/Platinum tier ladder + automated promotion/demotion.
- Surface the tier in the customer UI.
- Add per-tenant pinning (paid feature).

### Phase 3 — Wasm Component Model migration (if/when ready)

- Re-evaluate WASI Preview 2 maturity in Workers.
- If green, port the Tier 3 isolate runtime to Wasm components with WIT-typed imports.
- If not, defer. The capability-typed `ctx` interface designed in Phase 0 makes the migration mechanical when the time comes.

---

## 16. Open questions (to resolve before Phase 1 opens)

1. **Idempotency-key derivation when the customer didn't supply one.** Hash of (tenant, category, normalized input, time bucket)? What's the bucket size? What's the storage TTL? Probably 24h Redis with a 1-minute bucket, but unverified.
2. **Vendor-error-code normalization granularity.** Seven canonical codes (Section 7) might be too coarse for `payment.*` and too fine for `event.track`. Leaving room to add per-category extension codes without bloating the canon.
3. **What stops a malicious Tier 3 adapter from making the V8 isolate burn CPU?** Workers-for-Platforms has CPU-time limits per request, but a chain of slow vendor calls could still be expensive. Need a per-tenant Tier-3 budget separate from the Tier-1 budget.
4. **How does a customer discover which vendor served them historically?** The envelope shows the current call. A "last 24h breakdown" UI is Phase-2 work, but customers will ask for it before then.
5. **Do we need a Category-deprecation policy?** What happens to `email.send@v1` in 2030? Probably nothing — it just has fewer adapters — but writing the policy now is cheap and writing it later is expensive.
