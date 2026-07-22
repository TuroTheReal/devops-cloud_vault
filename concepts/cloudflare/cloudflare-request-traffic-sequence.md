---
tags:
  - concept
  - cloudflare
  - request-flow
  - status/learning
created: 2026-07-21
difficulty: 4
time-to-master: 6h
---

# Request Traffic Sequence - Cloudflare

**Prerequisites**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-origin-rules]]
**Related to**: [[cloudflare-worker-shadows-origin-rule]]

---

## 🎯 TL;DR (30 seconds)

> A request travels through Cloudflare products in a **defined order**: DNS → security (WAF/custom rules) → transform/redirect rules → Origin Rules → Workers → cache → origin.
> Which feature "wins" depends on this order AND on whether a feature is **terminal** (a Worker can return a response and stop the chain).

---

## 🤔 When to Use?

### ✅ Good for

- **Predicting behavior**: which feature handles a request, and whether it stops the chain.
- **Debugging**: "why didn't my rule/Worker fire?" on a hostname.

### ❌ Bad for

- **Design bets on the exact internal order** → prefer the observed rule (below) over assumptions.

---

## 📚 Key Concepts

### 1. The order (to confirm precisely)

**Facts (to verify)**: DNS → WAF/custom rules → transform/redirect → Origin Rules → Workers → cache → origin.

**My understanding**:
To predict behaviour I ask two questions: *what runs first*, and *does it hand off or terminate*. The order tells me which feature sees the request first; "terminal" tells me whether it stops the chain. A Worker that returns a response ends it; a rule that rewrites the origin only matters if the request actually reaches the origin step.

**Why important**: the surprising cases (a Custom Domain bypassing an Origin Rule) come from a *terminal* feature short-circuiting the chain, not from the order you'd guess.

**Mental model**: a conveyor of stations (DNS → rules → Origin Rules → Workers → cache → origin); a Worker can grab the item and stop the belt (terminal); a rule only acts if the item reaches its station.

### 2. Terminal vs pass-through

A Worker can return a response (terminal) or fetch the origin (pass-through). A Custom Domain makes the Worker the origin → terminal for all paths.

---

## 📌 What I rely on (observed)

The rule I trust in practice: **a Route + a plain/placeholder DNS record keeps origin-level features (Origin Rules) working, while a Custom Domain makes the Worker the origin and can bypass them.** That behaviour is observed and reliable.

I don't assert the exact internal position of Workers vs Origin Rules here; if I ever need the precise mechanism, the official "HTTP request traffic sequence" page is the source. Day to day, the observed rule above is enough.

---

## 💻 Schema — request flow

### Context

The observed order a request passes through, and where each feature can act or stop the chain.

### Flow

```text
client → DNS → security (WAF / custom rules) → transform / redirect rules
       → Origin Rules → Workers → cache → origin
```

- A **Worker** can return a response and **stop here** (terminal) — nothing after it runs.
- An **Origin Rule** only acts if the request reaches the **origin** step.
- A **Custom Domain** makes the Worker the origin, so the origin-step features are skipped → the observed incident.

---

## ⚠️ Pitfalls Experienced

- **Guessing the order from intuition**: the surprising cases (a Custom Domain bypassing an Origin Rule) come from a *terminal* feature short-circuiting the chain, not from the order you'd expect. → [[cloudflare-worker-shadows-origin-rule]]

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> What decides which feature "wins"?</summary>

**Answer**: The order it runs in AND whether it is terminal (a Worker can return a response and stop the chain).
</details>

<details>
<summary><strong>Q2:</strong> The one reliable rule to remember?</summary>

**Answer**: Route + a plain/placeholder DNS record keeps Origin Rules working; a Custom Domain makes the Worker the origin and can bypass them.
</details>

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: https://developers.cloudflare.com/fundamentals/reference/

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21
