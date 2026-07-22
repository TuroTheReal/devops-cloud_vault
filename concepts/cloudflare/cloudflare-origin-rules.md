---
tags:
  - concept
  - cloudflare
  - rules
  - origin
  - status/learning
created: 2026-07-21
difficulty: 2
time-to-master: 2h
---

# Origin Rules - Cloudflare

**Prerequisites**: [[networking-fundamentals]], [[cloudflare-dns-proxied-records-and-placeholders]]
**Related to**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-request-traffic-sequence]], [[first-party-tagging-edge-gateway]]

> [!note] Stub — facts to verify/reword. `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> An Origin Rule changes **where / how** Cloudflare connects to the origin for requests matching a filter: override the **Host header**, the **DNS record / origin hostname**, the **destination port**, the **SNI**.

**Analogy**: TODO(you) — "forward the mail to a different address depending on the sender".

---

## 🤔 When to Use?

### ✅ Good for
- Serve a third party's content first-party (rewrite `/loader-path` to fetch from a vendor) → shares your domain, dodges adblockers / third-party cookies. → [[first-party-tagging-edge-gateway]]
- Route some paths to a different origin without changing public DNS.

### ❌ Bad for
- A hostname whose origin is a Worker Custom Domain → the Origin Rule does not run (no origin resolution). → [[cloudflare-worker-shadows-origin-rule]]

---

## 📚 Key Concepts

### 1. What an Origin Rule can override
**Facts**: Host header, DNS record / origin hostname, destination port, SNI. Each rule has a **filter expression** (host / path / etc.).

**My understanding**:
> TODO(you)

### 2. Prerequisite: the request must REACH origin resolution
**Key fact**: an Origin Rule only fires if the request reaches the origin step. If a Worker is the origin for that path (Custom Domain), the step is short-circuited → the rule is a no-op.

**Why important**:
> TODO(you) — this is exactly the incident's cause.

### 3. Precedence
**Fact**: if you override the hostname via an Origin Rule AND via load balancer config, the Origin Rule takes precedence.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: rule "invisible" under a Worker origin
See [[cloudflare-worker-shadows-origin-rule]].

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/rules/origin-rules/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
