---
tags:
  - concept
  - cloudflare
  - dns
  - proxy
  - status/learning
created: 2026-07-21
difficulty: 3
time-to-master: 7h
---

# Proxied DNS Records & Placeholders - Cloudflare

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-origin-rules]]

---

## 🎯 TL;DR (30 seconds)

> **Proxied (orange cloud)** = traffic goes through Cloudflare (WAF, cache, Workers, rules).
> **DNS-only (grey cloud)** = Cloudflare only answers DNS, no proxy.
> A **placeholder record** = a proxied record whose IP is never actually contacted; it exists only to give the orange cloud something to attach to.

**Analogy**: orange cloud = mail routed through Cloudflare's sorting office (which can inspect and act on it); grey cloud (DNS-only) = Cloudflare just hands you the address and steps aside.

---

## 🤔 When to Use?

### ✅ Good for

- **Backend-less hostname**: run a Route or rules where there's no real origin.
- **Apex origin without a server**: a proxied placeholder record.

### ❌ Bad for

- **Host that must reach a real backend** → use a normal proxied record pointing at that IP/host.

---

## 📚 Key Concepts

### 1. Proxied vs DNS-only

**Facts**: orange cloud = Cloudflare features active on that hostname. Grey = DNS resolution only.

**My understanding**:
Proxied means the hostname's traffic actually flows through Cloudflare, so every edge feature (WAF, cache, Workers, rules) can act on it. DNS-only means CF answers the lookup and nothing else, so none of those features apply.

**Mental model**: orange cloud = your mail goes through Cloudflare's sorting office (it can inspect and act); grey cloud = Cloudflare just tells you the address and steps aside.

### 2. Placeholder / dummy record

**Facts**: a proxied record whose content is never really reached. Common values:
- `AAAA 100::` (IPv6 discard prefix)
- `A 192.0.2.1` (TEST-NET-1, RFC 5737 documentation range)
Used to run a Route / rules on a hostname **without a real backend**.

**Why important**:
A placeholder gives the orange cloud something to bind to without a real backend. In the incident fix it replaced the Worker Custom Domain on the apex: the hostname then did DNS-only resolution to a dummy IP, so it was no longer a Worker origin, the Origin Rule fired again, and the Worker still ran via a Route.

### 3. Apex and CNAME flattening

**Facts**: an apex (`example.com`, no subdomain) cannot be a CNAME in classic DNS. Cloudflare uses **CNAME flattening**, but a proxied A/AAAA placeholder is the simplest apex origin.

---

## 💻 Minimal Example

### Context

Give an apex a proxied origin with **no real backend**, so a Worker Route + rules can run on it (the incident fix).

### Code

```bash
# proxied placeholder record (orange cloud ON), IP never actually contacted
curl -s -X POST "$API/zones/$ZONE/dns_records" -H "$AUTH" \
  -d '{"type":"AAAA","name":"example.com","content":"100::","proxied":true,"ttl":1}'

# the Worker still serves the app via a Route → zone pipeline (Origin Rules) stays intact
curl -s -X POST "$API/zones/$ZONE/workers/routes" -H "$AUTH" \
  -d '{"pattern":"example.com/*","script":"my-worker"}'
```

`100::` is the IPv6 discard address: never reached, it just gives the orange cloud something to attach to.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: placeholder left grey-clouded

**Symptom**:

```text
The Route / rules don't apply, and requests seem to hit a dead IP.
```

**What went wrong**: the placeholder record was DNS-only (grey). No edge features run, and Cloudflare tries to actually reach the fake IP.

**Solution**: turn the record **proxied** (orange cloud ON). The IP stays a placeholder, but the pipeline runs.

**Lesson**: a placeholder only works proxied — the point is the orange cloud, not the IP.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> What is a proxied `AAAA 100::` record for?</summary>

**Answer**: A placeholder origin: an orange-cloud anchor with no real backend, so a Route/rules can run. `100::` is never actually contacted.
</details>

<details>
<summary><strong>Q2:</strong> Can an apex be a CNAME?</summary>

**Answer**: Not in classic DNS. Cloudflare uses CNAME flattening, but a proxied A/AAAA placeholder is the simplest apex origin.
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/dns/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
