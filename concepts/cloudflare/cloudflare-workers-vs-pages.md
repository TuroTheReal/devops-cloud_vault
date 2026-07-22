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
time-to-master: 6h
---

# Workers vs Pages - Cloudflare

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[cloudflare-custom-domain-vs-route]], [[edge-computing-fundamentals]]

---

## 🎯 TL;DR (30 seconds)

> **Workers** = general-purpose serverless functions at the edge (full request control, routing, SSR, APIs).
> **Pages** = opinionated static-site / Jamstack hosting built on top of Workers.
> Trend: Pages de-emphasized for app hosting in favor of "Workers + static assets".

**Analogy**: Workers = a programmable kitchen you can cook anything in; Pages = a meal-kit service built on top of that kitchen, convenient for static sites but boxed-in.

---

## 🤔 When to Use?

### ✅ Good for

- **Static + dynamic in one place**: routing, SSR, APIs alongside assets.
- **Migrating a static/SPA site** that also needs edge logic.

### ❌ Bad for

- **Pure static site, zero edge logic** → plain static hosting is simpler (though Pages is de-emphasized).

---

## 📚 Key Concepts

### 1. Workers

**Facts**: edge execution, full request control, static assets + dynamic logic in one primitive, Terraform-manageable.

**My understanding**:
Workers runs code on each request and can also serve static files. Pages only did the static part, so Workers can do everything Pages did, and more.

**Why important**: consolidating on one primitive means IaC-manageable, no Pages-specific CI, and less platform risk as Pages is de-emphasized.

**Mental model**: Workers = a programmable kitchen (cook anything per request); Pages = a meal-kit on top — convenient for static, boxed-in.

### 2. Pages

**Facts**: focused on static builds / preview deployments, Pages-specific CI.

### 3. Migration rationale Pages → Workers

One primitive for both static assets and dynamic logic, manageable in Terraform, and no Pages-specific CI to maintain. The provider is also de-emphasizing Pages for app hosting, so consolidating on Workers reduces long-term platform risk.

---

## 💻 Minimal Example

### Context

One Worker that serves **static assets AND** handles dynamic routes — what replaces a Pages app.

### wrangler.toml

```toml
name = "my-app"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[assets]
directory = "./dist"   # static files (what Pages used to serve)
# non-asset requests fall through to the Worker code in src/index.ts (dynamic)
```

One primitive for static + dynamic, managed by CLI/Terraform.

**Request flow**:

```text
request example.com/foo
   └─ Route example.com/*  → Worker
        ├─ /assets/*  → static assets ([assets] dir)
        └─ else       → Worker code (SSR / API)
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: treating the migration as a lift-and-shift

**Symptom**:

```text
The app "works" as a Pages site but behaves differently once it's a Worker
(SPA fallback, headers, or asset routing break).
```

**What went wrong**: routing, `wrangler` config and build output differ between Pages and Workers; the app was re-pointed, not re-reviewed.

**Solution**: review each app's config + build output; test SPA fallback and headers at the edge.

**Lesson**: Workers can do everything Pages did, but the wiring is different per app.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Does Workers replace Pages?</summary>

**Answer**: Yes — Workers serves static assets + dynamic logic in one primitive, so it subsumes Pages. But migrating is not a lift-and-shift.
</details>

<details>
<summary><strong>Q2:</strong> What differs when migrating Pages → Workers?</summary>

**Answer**: Routing, config (`wrangler`), build output and asset handling — each app needs its config/build reviewed.
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/workers/static-assets/ · https://developers.cloudflare.com/pages/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
