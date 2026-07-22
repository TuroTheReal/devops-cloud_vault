---
tags:
  - concept
  - cloud
  - edge
  - cdn
  - serverless
  - status/learning
created: 2026-07-22
difficulty: 3
time-to-master: 6h
---

# Edge Computing Fundamentals

**Prerequisites**: [[cloud-fundamentals]], [[networking-fundamentals]]
**Related to**: [[cloudflare-workers-vs-pages]], [[cloudflare-request-traffic-sequence]], [[first-party-tagging-edge-gateway]]

---

## 🎯 TL;DR (30 seconds)

> Edge computing = run compute (or store data) **as close to users as possible**,
> across many points of presence, instead of one central region. Goal: cut latency
> by shrinking the physical distance between the user and the code.

**Analogy**: fast-food on every street corner (edge, 300+ PoPs) vs one central restaurant (region-first cloud). The user goes to the closest one for speed.

---

## 🤔 When to Use?

### ✅ Good for

- **Latency-sensitive traffic**: global users hitting auth checks, redirects, personalization, A/B, geo-routing.
- **Assets + light logic near the user**: serving/transforming static content at the edge.

### ❌ Bad for

- **Heavy or stateful work** → keep it in a region (large compute, in-memory state, GDPR data-locality).

---

## 📚 Key Concepts

### 1. Edge vs origin (centralized)

**Facts**: a classic app runs in one (or few) cloud region(s) = the **origin**. Edge pushes compute/storage to many PoPs near users. Closer = lower round-trip latency.

**My understanding**:
Edge optimizes round-trip time; origin optimizes capability and state. Most real apps need both, so I read it as "which part of the request is worth running near the user", not "edge vs cloud".

**Why important**: the win is purely latency, so you push only the latency-sensitive bits to the edge and keep heavy/stateful work at the origin.

**Mental model**: fast-food on every corner (edge, near everyone) vs one central restaurant (origin) — same food, the win is distance.

### 2. From CDN to edge compute

**Facts**: a **CDN** (Content Delivery Network) first cached **static assets** near users. Edge compute is the next step: run **code** at those same PoPs, not just serve cached files.

### 3. The serverless-edge model + its constraints

**Facts**: edge functions are serverless (no server to manage, scale-to-zero). Constraints that shape what you can run:
- **cold start** on first hit
- tight **CPU / memory / execution-time limits** per request
- **no durable local state** (use edge KV / object store / origin for state)
- often a restricted runtime (Web APIs, not full Node)

**My understanding**:
Edge is for **decisions** (auth, routing, redirects), not heavy work; anything that needs memory or a DB belongs at origin.

**Why important**: the constraints (cold start, tight CPU/time, no local state) are what force this — they decide what the edge can and can't do.

**Mental model**: an edge function = a stateless bouncer at the door — quick yes/no, no memory of you, not the venue.

### 4. Providers

**Facts**: Cloudflare Workers, AWS Lambda@Edge / CloudFront Functions, Fastly Compute, Deno Deploy, Vercel Edge Functions. Cloudflare Workers = one concrete instance → [[cloudflare-workers-vs-pages]].

---

## 💻 Schema — edge vs origin

```text
Origin-first:  user ─────────── long round-trip ───────────► one region (origin)

Edge:          user ── short ──► nearest PoP (edge worker) ──(only if needed)──► origin
```

The edge handles quick, stateless decisions near the user (auth, routing, redirects); the origin keeps state and heavy work.

---

## ⚠️ Pitfalls Experienced

- **Treating an edge worker like a normal origin**: making it the origin for a whole hostname silently bypassed a per-path origin rewrite and took down a dependent feature. → [[cloudflare-worker-shadows-origin-rule]]

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Why is edge faster, not more powerful?</summary>

**Answer**: Same compute, physically closer to the user; the gain is round-trip latency, not capacity.
</details>

<details>
<summary><strong>Q2:</strong> Where must edge state live?</summary>

**Answer**: In edge KV or the origin — edge instances are ephemeral and spread across PoPs.
</details>

---

## 📊 Stats

```yaml
Total time: ~6h (learning) + hands-on during the migration
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: Cloudflare Workers / Lambda@Edge / Fastly Compute docs; [[cloud-fundamentals]].

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
