---
tags:
  - concept
  - networking
  - edge
  - analytics
  - status/learning
created: 2026-07-21
difficulty: 2
time-to-master: 1h
---

# First-party Tagging via Edge Gateway - Concept

**Prerequisites**: [[cloudflare-origin-rules]]
**Related to**: [[cloudflare-worker-shadows-origin-rule]]

> [!note] Stub — optional. `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> A **tag gateway** (e.g. Google Tag Gateway / server-side GTM) serves a third-party
> analytics loader from **your own domain** via an edge rewrite (Origin Rule on specific
> loader paths → vendor servers). Why: first-party context dodges adblockers and
> third-party-cookie restrictions, improving measurement.

**Analogy**: TODO(you).

---

## 📚 Key Concepts

### 1. Why first-party
**Facts**: adblockers + browsers block third-party requests/cookies. Serving the loader from your domain = first-party = less blocked.

**My understanding**:
> TODO(you)

### 2. How (edge mechanism)
Loader paths (often random) are rewritten by an **Origin Rule** to the vendor's servers. The browser only sees your domain.

### 3. Fragility
These paths depend on the Origin Rule running. If the hostname's origin becomes a Worker (Custom Domain), the rule no longer fires → the gateway breaks. → [[cloudflare-worker-shadows-origin-rule]]

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: tagging vendor docs + [[cloudflare-origin-rules]]

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
