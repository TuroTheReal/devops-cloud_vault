---
tags:
  - concept
  - cloudflare
  - request-flow
  - status/learning
created: 2026-07-21
difficulty: 3
time-to-master: 2h
---

# Request Traffic Sequence - Cloudflare

**Prerequisites**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-origin-rules]]
**Related to**: [[cloudflare-worker-shadows-origin-rule]]

> [!note] Stub — contains an **open question to resolve in docs** before asserting. `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> A request travels through Cloudflare products in a **defined order**: DNS → security (WAF/custom rules) → transform/redirect rules → Origin Rules → Workers → cache → origin.
> Which feature "wins" depends on this order AND on whether a feature is **terminal** (a Worker can return a response and stop the chain).

---

## 📚 Key Concepts

### 1. The order (to confirm precisely)
**Facts (to verify)**: DNS → WAF/custom rules → transform/redirect → Origin Rules → Workers → cache → origin.

**My understanding**:
> TODO(you)

### 2. Terminal vs pass-through
A Worker can return a response (terminal) or fetch the origin (pass-through). A Custom Domain makes the Worker the origin → terminal for all paths.

---

## ⚠️ Open question (DO NOT guess)

> [!warning]
> Some docs state "when a Worker route matches, the Worker runs before origin resolution, so origin overrides are bypassed". Yet in the incident: a **Route** (`/*`) + plain DNS record let the Origin Rule fire, while a **Custom Domain** did not.
> The **observed fact** is certain. The **exact mechanism** (Custom-Domain-vs-Route ordering nuance) is not.
> → Resolve via the official "HTTP request traffic sequence" page + Origin Rules docs BEFORE writing the mechanism. `TODO(you)`.

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/fundamentals/reference/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
