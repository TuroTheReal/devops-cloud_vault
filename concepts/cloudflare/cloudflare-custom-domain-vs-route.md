---
tags:
  - concept
  - cloudflare
  - workers
  - dns
  - routing
  - status/learning
created: 2026-07-21
updated: 2026-07-22
difficulty: 4
time-to-master: 12h
---

# Custom Domain vs Route - Cloudflare Workers

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[cloudflare-origin-rules]], [[cloudflare-dns-proxied-records-and-placeholders]], [[cloudflare-request-traffic-sequence]], [[cloudflare-worker-shadows-origin-rule]]

---

## 🎯 TL;DR (30 seconds)

> Two ways to attach a Worker to a hostname, and they are **not equivalent**:
> a **Custom Domain** makes the Worker the **origin** for every path, while a **Route**
> attaches the Worker to a **path pattern** and leaves the zone's DNS + pipeline intact.
> Pick wrong and you silently lose origin-level features (Origin Rules, tag gateways).

**Analogy**: Custom Domain = the Worker *is* the building — knock on any door, you get the Worker, there's nothing behind it. Route = the Worker is a doorman on certain floors; the building (origin) still exists behind him.

---

## 🤔 When to Use?

### ✅ Custom Domain — good for

- **Fully Worker-served host**: the Worker answers every path and nothing else needs to act at origin level.
- **Same-zone Worker→Worker `fetch()`**: call another Worker on the same zone by hostname, without a service binding.
- **Zero DNS/cert fuss**: Cloudflare auto-creates the DNS record and an Advanced Certificate for you.

### ✅ Route — good for

- **Host that must keep origin-level behavior**: Origin Rules, tag gateway, custom origin still fire because the pipeline stays.
- **Wildcard / path matching**: `example.com/*` or `example.com/api/*`, several Workers on one hostname by path.
- **Coexisting with a real (or placeholder) origin**: the hostname still resolves via DNS.

### ❌ Bad for

- **Custom Domain on a host that also needs a per-path Origin Rule** → the Worker becomes terminal and the rule never runs (silent). → [[cloudflare-worker-shadows-origin-rule]]
- **Route without any DNS record on the hostname** → nothing resolves, so the Worker is never reached (needs a real or placeholder proxied record).

---

## 📚 Key Concepts

### 1. Custom Domain = the Worker is treated as the origin

**Facts (official docs)**:

- "A Worker running on a Custom Domain is treated as an origin."
- "Custom Domains point **all paths** of a domain or subdomain to your Worker" — no wildcards, exact hostname.
- Cloudflare auto-creates the DNS record + an Advanced Certificate on the zone.
- You **cannot** create a Custom Domain on a hostname that already has a CNAME record.

**My understanding**:
The Custom Domain makes the Worker **terminal** for that hostname: the Worker *is* the origin, so there is no separate backend behind it to route to or rewrite. Requests to any path end at the Worker.

**Why important**:
Anything that acts at the **origin-resolution** step (Origin Rules, custom origin overrides, a first-party tag gateway) has nothing left to act on, because the origin is the Worker itself. You lose those features **silently** — no error, just different behavior.

**Mental model**:
The Worker *is* the building. Every door (path) opens onto the Worker; there is no "behind the building" to send someone else to.

### 2. Route = attach the Worker to a path pattern

**Facts**:

- Pattern like `example.com/*` (wildcards supported); several routes/Workers can share a hostname by path.
- The zone's DNS record stays in place → the hostname still resolves to an origin (real or a proxied placeholder).
- The Worker runs for **matched** requests, but the zone keeps its normal request pipeline.

**My understanding**:
A Route just says "run this Worker for URLs matching this pattern". The hostname still resolves through DNS, so the Worker is **one participant** in the chain, not the endpoint. You can combine it with origin-level features.

**Why important**:
Because the pipeline is intact, Origin Rules / cache / redirects still fire around the Worker. This is what lets a hostname be *both* Worker-served *and* keep a per-path origin rewrite — impossible with a Custom Domain.

**Mental model**:
The Worker is a doorman stopping visitors to certain floors. The building (origin) still exists; unmatched or origin-level traffic flows past him as usual.

### 3. Practical decision rule

- Host is **100% the Worker**, no other origin behavior needed → **Custom Domain** (simplest, auto DNS+cert).
- Host must **keep origin-level features** (Origin Rule, tag gateway) → **Route + a proxied placeholder DNS record** ([[cloudflare-dns-proxied-records-and-placeholders]]).
- When unsure, prefer the Route: it composes; the Custom Domain is all-or-nothing.

---

## 💻 Minimal Example

### Context

Put a Worker on a hostname **while keeping** an existing per-path Origin Rule working. The correct shape is a **Route + proxied placeholder record**, not a Custom Domain.

### Code

```bash
# ✅ Route: the Worker runs for matched paths, the zone pipeline stays intact
curl -s -X POST "$API/zones/$ZONE/workers/routes" -H "$AUTH" \
  -d '{"pattern":"example.com/*","script":"my-worker"}'

# ...paired with a proxied placeholder so the hostname still "resolves" (no real backend)
curl -s -X POST "$API/zones/$ZONE/dns_records" -H "$AUTH" \
  -d '{"type":"AAAA","name":"example.com","content":"100::","proxied":true,"ttl":1}'

# ❌ Custom Domain: the Worker becomes THE origin for every path (Origin Rules stop firing)
curl -s -X PUT "$API/accounts/$ACCT/workers/domains" -H "$AUTH" \
  -d '{"hostname":"example.com","service":"my-worker","zone_id":"'"$ZONE"'","environment":"production"}'
```

### Line-by-line

```text
workers/routes  {pattern, script}
  → "run my-worker for example.com/*", but DNS still decides the origin
dns_records  AAAA 100:: proxied
  → gives the hostname a proxied origin with no real backend, so the pipeline (Origin Rules) runs
workers/domains  {hostname, service}
  → binds the Worker AS the origin for the whole hostname → terminal, origin features skipped
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: a Custom Domain silently disabled an Origin Rule

**Symptom**:

```text
After moving an apex onto a Worker, specific paths served by an Origin Rule
(a first-party tag gateway's loader paths → rewritten to a vendor) stopped
returning the vendor's content. The sites depending on that origin behavior
went down. No error in the Worker logs — just "wrong" responses.
```

**What went wrong**: the Worker was attached via a **Custom Domain** → it became the origin for *every* path → the Origin Rule (which acts at origin resolution) never ran.

**Solution**:

```bash
# delete the Custom Domain, drop a proxied placeholder, keep the Worker via a Route
DELETE /accounts/$ACCT/workers/domains/$CD_ID
POST   /zones/$ZONE/dns_records   {AAAA 100:: proxied}
POST   /zones/$ZONE/workers/routes {pattern: example.com/*, script: my-worker}
```

**Lesson**: a Custom Domain is **not** a drop-in for "put the Worker on this host". It makes the Worker the origin and can shadow origin-level features. When the host needs origin behavior, use **Route + placeholder**. Full write-up: [[cloudflare-worker-shadows-origin-rule]].

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Why can a per-path Origin Rule never run under a Custom Domain?</summary>

**Answer**: A Custom Domain makes the Worker the origin (terminal) for all paths. Origin Rules act at the origin-resolution step, which is skipped when the Worker *is* the origin — so the rule is a silent no-op.
</details>

<details>
<summary><strong>Q2:</strong> Which combination keeps both the Worker AND the Origin Rule working?</summary>

**Answer**: A Worker **Route** (`example.com/*`) + a plain **proxied placeholder** DNS record. DNS-only resolution keeps the zone pipeline, so the Origin Rule fires; the Route still runs the Worker.
</details>

<details>
<summary><strong>Q3:</strong> Why can't you create a Custom Domain on a hostname that already has a CNAME?</summary>

**Answer**: Cloudflare auto-creates its own DNS record for the Custom Domain; a pre-existing CNAME would conflict with it, so it's disallowed.
</details>

---

## 📊 Stats

```yaml
Total time: ~12h (incl. debugging the incident)
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: [Cloudflare Workers — Routing](https://developers.cloudflare.com/workers/configuration/routing/)

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
