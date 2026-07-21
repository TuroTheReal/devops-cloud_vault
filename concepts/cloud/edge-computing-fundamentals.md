---
tags:
  - concept
  - cloud
  - edge
  - cdn
  - serverless
  - status/learning
created: 2026-07-22
difficulty: 2
time-to-master: 3h
---

# Edge Computing Fundamentals

**Prerequisites**: [[cloud-fundamentals]], [[networking-fundamentals]]
**Related to**: [[cloudflare-workers-vs-pages]], [[cloudflare-request-traffic-sequence]], [[first-party-tagging-edge-gateway]]

> [!note] Stub — facts pre-filled, rewrite "My understanding" in my words. `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> Edge computing = run compute (or store data) **as close to users as possible**,
> across many points of presence, instead of one central region. Goal: cut latency
> by shrinking the physical distance between the user and the code.

**Analogy**: fast-food on every street corner (edge, 300+ PoPs) vs one central restaurant (region-first cloud). The user goes to the closest one for speed.

---

## 🤔 When to Use?

### ✅ Good for
- Latency-sensitive, globally-distributed traffic (auth checks, redirects, personalization, A/B, geo-routing).
- Serving/transforming static assets and light dynamic logic near the user.

### ❌ Bad for
- Heavy/long compute, large in-memory state, or strict data-locality (GDPR) → keep in a region.

---

## 📚 Key Concepts

### 1. Edge vs origin (centralized)
**Facts**: a classic app runs in one (or few) cloud region(s) = the **origin**. Edge pushes compute/storage to many PoPs near users. Closer = lower round-trip latency.

**My understanding**:
> TODO(you)

### 2. From CDN to edge compute
**Facts**: a **CDN** (Content Delivery Network) first cached **static assets** near users. Edge compute is the next step: run **code** at those same PoPs, not just serve cached files.

### 3. The serverless-edge model + its constraints
**Facts**: edge functions are serverless (no server to manage, scale-to-zero). Constraints that shape what you can run:
- **cold start** on first hit
- tight **CPU / memory / execution-time limits** per request
- **no durable local state** (use edge KV / object store / origin for state)
- often a restricted runtime (Web APIs, not full Node)

**My understanding**:
> TODO(you)

### 4. Providers
**Facts**: Cloudflare Workers, AWS Lambda@Edge / CloudFront Functions, Fastly Compute, Deno Deploy, Vercel Edge Functions. Cloudflare Workers = one concrete instance → [[cloudflare-workers-vs-pages]].

---

## ⚠️ Pitfalls Experienced
> TODO(you) — e.g. assuming edge behaves like a normal origin (see [[cloudflare-worker-shadows-origin-rule]]).

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: extracted from my `noteanepasoublierpourplustard` Cloud — Fundamentals notes; validate against provider docs.

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
