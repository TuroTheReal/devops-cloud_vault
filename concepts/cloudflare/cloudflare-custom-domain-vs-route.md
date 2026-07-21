---
tags:
  - concept
  - cloudflare
  - workers
  - dns
  - routing
  - status/learning
created: 2026-07-21
difficulty: 3
time-to-master: 3h
---

# Custom Domain vs Route - Cloudflare Workers

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[cloudflare-origin-rules]], [[cloudflare-dns-proxied-records-and-placeholders]], [[cloudflare-request-traffic-sequence]], [[cloudflare-worker-shadows-origin-rule]]

> [!note] Stub — facts pre-filled (verified in docs), rewrite in my own words. `TODO(you)` markers.

---

## 🎯 TL;DR (30 seconds)

> Two ways to attach a Worker to a hostname, **not equivalent**:
> Custom Domain = the Worker **becomes the origin** for all paths.
> Route = the Worker attaches to a **path pattern**, the DNS/pipeline stays in place.

**Analogy**: Custom Domain = the Worker *is* the building. Route = the Worker is a doorman who only stops people going to certain floors; the building still exists.

---

## 🤔 When to Use?

### ✅ Custom Domain — good for
- A hostname served 100% by the Worker, nothing else needed at origin level.
- Same-zone Worker→Worker communication via `fetch()` without a service binding.

### ✅ Route — good for
- A hostname that must keep other origin-level behavior (Origin Rules, tag gateway).
- Pattern matching with wildcards (`example.com/*`).

### ❌ Anti-pattern
- Putting a **Custom Domain** on a hostname that needs a per-path Origin Rule → the Worker becomes terminal and the Origin Rule never runs. → [[cloudflare-worker-shadows-origin-rule]]

---

## 📚 Key Concepts

### 1. Custom Domain = Worker treated as origin

**Facts (official docs)**:
- "A Worker running on a Custom Domain is treated as an origin."
- "Custom Domains point **all paths** of a domain or subdomain to your Worker." (no wildcard, exact hostname)
- Cloudflare auto-creates the DNS record + an Advanced Certificate.
- You cannot create a Custom Domain on a hostname that already has a CNAME record.

**My understanding**:
> TODO(you): reword why "the Worker is the origin" = it terminates the request, so there is no separate origin left to rewrite.

### 2. Route = attach the Worker to a path pattern

**Facts**:
- Pattern like `example.com/*`, wildcards supported.
- The zone DNS record stays in place → the hostname still resolves to an origin (real or placeholder).
- The Worker runs for matched requests, but the zone keeps its normal pipeline.

**My understanding**:
> TODO(you)

### 3. Practical consequence
If a hostname must keep origin-level behavior (an Origin Rule that rewrites some paths to a third party), **Route + a proxied placeholder DNS record** preserves it, whereas a **Custom Domain** can swallow it.

---

## 💻 Minimal Example

```bash
# Route (preserves the zone pipeline)
curl -s -X POST "$API/zones/$ZONE/workers/routes" -H "$AUTH" \
  -d '{"pattern":"example.com/*","script":"my-worker"}'

# Custom Domain (Worker = origin, all paths)
curl -s -X PUT "$API/accounts/$ACCT/workers/domains" -H "$AUTH" \
  -d '{"hostname":"example.com","service":"my-worker","zone_id":"'"$ZONE"'","environment":"production"}'
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Custom Domain shadows an Origin Rule
See [[cloudflare-worker-shadows-origin-rule]] for the full case.
**Lesson**: a Custom Domain is NOT a drop-in for "put the worker on this host".

---

<!--
## 🧠 Retrieval Practice (status/learning)
Q1: Why can a per-path Origin Rule never run under a Custom Domain?
Q2: Which combination keeps both the Worker AND the Origin Rule working?
-->

---

## 📊 Stats

```yaml
Total time: TODO
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/workers/configuration/routing/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
