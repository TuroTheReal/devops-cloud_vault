---
tags:
  - concept
  - cloudflare
  - dns
  - proxy
  - status/learning
created: 2026-07-21
difficulty: 2
time-to-master: 2h
---

# Proxied DNS Records & Placeholders - Cloudflare

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-origin-rules]]

> [!note] Stub — `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> **Proxied (orange cloud)** = traffic goes through Cloudflare (WAF, cache, Workers, rules).
> **DNS-only (grey cloud)** = Cloudflare only answers DNS, no proxy.
> A **placeholder record** = a proxied record whose IP is never actually contacted; it exists only to give the orange cloud something to attach to.

**Analogy**: TODO(you).

---

## 📚 Key Concepts

### 1. Proxied vs DNS-only
**Facts**: orange cloud = Cloudflare features active on that hostname. Grey = DNS resolution only.

**My understanding**:
> TODO(you)

### 2. Placeholder / dummy record
**Facts**: a proxied record whose content is never really reached. Common values:
- `AAAA 100::` (IPv6 discard prefix)
- `A 192.0.2.1` (TEST-NET-1, RFC 5737 documentation range)
Used to run a Route / rules on a hostname **without a real backend**.

**Why important**:
> TODO(you) — this is what replaces the Custom Domain in the incident fix.

### 3. Apex and CNAME flattening
**Facts**: an apex (`example.com`, no subdomain) cannot be a CNAME in classic DNS. Cloudflare uses **CNAME flattening**, but a proxied A/AAAA placeholder is the simplest apex origin.

---

## ⚠️ Pitfalls Experienced
> TODO(you) if encountered.

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/dns/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
