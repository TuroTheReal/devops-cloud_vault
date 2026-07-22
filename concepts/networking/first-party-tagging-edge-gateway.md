---
tags:
  - concept
  - networking
  - edge
  - analytics
  - status/learning
created: 2026-07-21
difficulty: 3
time-to-master: 3h
---

# First-party Tagging via Edge Gateway - Concept

**Prerequisites**: [[cloudflare-origin-rules]]
**Related to**: [[cloudflare-worker-shadows-origin-rule]]

---

## 🎯 TL;DR (30 seconds)

> A **tag gateway** (e.g. Google Tag Gateway / server-side GTM) serves a third-party
> analytics loader from **your own domain** via an edge rewrite (Origin Rule on specific
> loader paths → vendor servers). Why: first-party context dodges adblockers and
> third-party-cookie restrictions, improving measurement.

**Analogy**: like re-labelling a courier's parcels with your own logo so the doorman (the adblocker) lets them through.

---

## 🤔 When to Use?

### ✅ Good for

- **Better analytics coverage**: serve a vendor's loader first-party (dodges adblockers / 3p-cookie limits).

### ❌ Bad for

- **Host whose origin is a Worker Custom Domain** → the edge rewrite silently stops working. → [[cloudflare-worker-shadows-origin-rule]]

---

## 📚 Key Concepts

### 1. Why first-party

**Facts**: adblockers + browsers block third-party requests/cookies. Serving the loader from your domain = first-party = less blocked.

**My understanding**:
Serving a third-party loader from my own domain makes it first-party, so it dodges adblockers and third-party-cookie limits and measures more traffic. The cost is that I now *own* that routing: if my edge setup changes (e.g. the origin becomes a Worker), I can silently break it.

**Mental model**: re-labelling a courier's parcels with your own logo so the doorman (the adblocker) lets them through.

### 2. How (edge mechanism)

Loader paths (often random) are rewritten by an **Origin Rule** to the vendor's servers. The browser only sees your domain.

### 3. Fragility

These paths depend on the Origin Rule running. If the hostname's origin becomes a Worker (Custom Domain), the rule no longer fires → the gateway breaks. → [[cloudflare-worker-shadows-origin-rule]]

---

## 💻 Minimal Example

### Context

The loader path is served first-party via an Origin Rule (see [[cloudflare-origin-rules]]).

### Rule

```text
When:  http.request.uri.path eq "/gtag-loader"
Then:  Host + SNI → vendor.analytics.example
```

Browser sees `example.com/gtag-loader` (first-party); Cloudflare fetches it from the vendor.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: an edge change kills the gateway

**Symptom**:

```text
Analytics coverage drops; the loader path returns the Worker's response, not the vendor's.
```

**What went wrong**: the hostname's origin became a Worker (Custom Domain), so the Origin Rule that served the loader stopped firing.

**Solution**: keep the Worker on a Route + proxied placeholder so origin resolution (and the rule) stays. → [[cloudflare-worker-shadows-origin-rule]]

**Lesson**: first-party tagging is only as stable as the Origin Rule it rides on.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Why serve a tag loader first-party?</summary>

**Answer**: First-party context dodges adblockers and third-party-cookie limits, so more traffic is measured.
</details>

<details>
<summary><strong>Q2:</strong> What makes it fragile?</summary>

**Answer**: It depends on the Origin Rule running; if the hostname's origin becomes a Worker (Custom Domain), the rewrite silently stops.
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: tagging vendor docs + [[cloudflare-origin-rules]]

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
