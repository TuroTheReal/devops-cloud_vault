---
tags:
  - concept
  - cloudflare
  - rules
  - origin
  - status/learning
created: 2026-07-21
difficulty: 3
time-to-master: 6h
---

# Origin Rules - Cloudflare

**Prerequisites**: [[networking-fundamentals]], [[cloudflare-dns-proxied-records-and-placeholders]]
**Related to**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-request-traffic-sequence]], [[first-party-tagging-edge-gateway]]

---

## 🎯 TL;DR (30 seconds)

> An Origin Rule changes **where / how** Cloudflare connects to the origin for requests matching a filter: override the **Host header**, the **DNS record / origin hostname**, the **destination port**, the **SNI**.

**Analogy**: like telling the mailroom to forward certain letters to a different address depending on the sender, before they even leave the building.

---

## 🤔 When to Use?

### ✅ Good for

- **First-party serving of a third party**: rewrite `/loader-path` to a vendor → shares your domain, dodges adblockers / 3p cookies. → [[first-party-tagging-edge-gateway]]
- **Per-path origin routing**: send some paths to a different origin without changing public DNS.

### ❌ Bad for

- **Host whose origin is a Worker Custom Domain** → the rule never runs (no origin resolution). → [[cloudflare-worker-shadows-origin-rule]]

---

## 📚 Key Concepts

### 1. What an Origin Rule can override

**Facts**: Host header, DNS record / origin hostname, destination port, SNI. Each rule has a **filter expression** (host / path / etc.).

**My understanding**:
An Origin Rule is a per-request rewrite of *where/how* Cloudflare fetches from the backend (host, port, SNI, which DNS record). It doesn't change what the browser sees or the public URL, only the origin CF talks to behind the scenes.

**Mental model**: the mailroom re-addressing certain letters to a different building based on the sender, before they leave.

### 2. Prerequisite: the request must REACH origin resolution

**Key fact**: an Origin Rule only fires if the request reaches the origin step. If a Worker is the origin for that path (Custom Domain), the step is short-circuited → the rule is a no-op.

**Why important**:
The rule lives at the origin-resolution step. If a Worker Custom Domain already *is* the origin, that step never happens, so the rule is a silent no-op. This is exactly what broke the tag-gateway paths in the incident.

### 3. Precedence

**Fact**: if you override the hostname via an Origin Rule AND via load balancer config, the Origin Rule takes precedence.

---

## 💻 Minimal Example

### Context

Serve a vendor's analytics loader first-party: rewrite requests to `/loader-path` so Cloudflare fetches from the vendor instead of your origin. The browser still sees your domain.

### Rule (filter → override)

```text
When incoming requests match:  http.request.uri.path eq "/loader-path"
Then override:
  - Host header  → vendor.analytics.example
  - SNI / origin → vendor.analytics.example
```

The public URL stays `example.com/loader-path`; only where Cloudflare fetches from changes. If a Worker Custom Domain is the origin, this rule never runs (no origin resolution).

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: the rule silently no-ops under a Worker origin

**Symptom**:

```text
Paths meant to be rewritten to a vendor return the Worker's response instead
of the vendor content. No error — the rule just never applies.
```

**What went wrong**: the hostname's origin is a Worker (Custom Domain), so origin resolution is skipped and the Origin Rule never runs.

**Solution**: serve the Worker via a Route + a proxied placeholder record so DNS resolution (and the rule) stays in the pipeline. → [[cloudflare-worker-shadows-origin-rule]]

**Lesson**: an Origin Rule only exists at the origin step; if something upstream is terminal, the rule is dead code.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Does an Origin Rule fire under a Worker Custom Domain?</summary>

**Answer**: No. The Worker is the origin, so origin resolution is skipped and the rule is a silent no-op.
</details>

<details>
<summary><strong>Q2:</strong> What can an Origin Rule override?</summary>

**Answer**: Host header, DNS record / origin hostname, destination port, SNI — per a filter expression.
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/rules/origin-rules/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
