---
tags:
  - concept
  - cloudflare
  - workers
  - pages
  - edge-compute
  - status/learning
created: 2026-07-21
difficulty: 2
time-to-master: 2h
---

# Workers vs Pages - Cloudflare

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[cloudflare-custom-domain-vs-route]], [[edge-computing-fundamentals]]

> [!note] Stub — `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> **Workers** = general-purpose serverless functions at the edge (full request control, routing, SSR, APIs).
> **Pages** = opinionated static-site / Jamstack hosting built on top of Workers.
> Trend: Pages de-emphasized for app hosting in favor of "Workers + static assets".

**Analogy**: TODO(you).

---

## 📚 Key Concepts

### 1. Workers
**Facts**: edge execution, full request control, static assets + dynamic logic in one primitive, Terraform-manageable.

**My understanding**:
> TODO(you)

### 2. Pages
**Facts**: focused on static builds / preview deployments, Pages-specific CI.

### 3. Migration rationale Pages → Workers
One primitive for assets + dynamic, IaC-manageable, no Pages-specific CI.
> TODO(you): your concrete observed reasons.

---

## ⚠️ Pitfalls Experienced
> TODO(you).

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/workers/static-assets/ · https://developers.cloudflare.com/pages/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
