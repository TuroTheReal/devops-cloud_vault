---
tags:
  - concept
  - ci-cd
  - pipeline
  - automation
  - status/learning
created: 2026-07-22
difficulty: 2
time-to-master: 5h
---

# CI/CD Basics

**Prerequisites**: [[git-github-fundamentals]]
**Related to**: [[deployment-strategies]], [[2026-07-cloudflare-pages-to-workers-migration]]

---

## 🎯 TL;DR (30 seconds)

> **CI** (Continuous Integration) = every change is automatically built and tested.
> **CD** (Continuous Delivery/Deployment) = a passing change is automatically shipped
> (to a manual gate = delivery, all the way to prod = deployment).
> A pipeline is just: **build → test → gate → deploy**, run by a machine, not by hand.

**Analogy**: a pastry kitchen with tasting checkpoints: nothing leaves for the display case until it clears each stage (mix → bake → taste → plate).

---

## 🤔 When to Use?

### ✅ Good for

- **Reliable, repeatable deploys**: same steps every time, no "works on my machine".
- **Catching issues before prod**: regressions and security problems blocked at a gate.

### ❌ Bad for

- **Over-engineering a one-off**: a full pipeline for a throwaway script is wasted effort.

---

## 📚 Key Concepts

### 1. CI vs CD

**Facts**: CI = build + test on every push. CD = automatically deliver (to a gate) or deploy (to prod) what passed CI.

**My understanding**:
CI answers "is this change safe to integrate?"; CD answers "how does a safe change reach users without me doing it by hand?". Both exist to remove manual, error-prone steps.

**Mental model**: a factory belt with quality gates (build → test → security → deploy) — nothing moves to the next station until it passes.

### 2. Pipeline stages

**Facts**: typical order = build → test (unit/integration) → gates (lint, security, coverage) → deploy. A failing stage stops the pipeline.

### 3. Gates

**Facts**: automated checks that must pass to proceed (tests, linters, security scanners). A gate turns "we should check X" into "X is enforced".

---

## 💻 Where I've used it (3 real cases)

- **Cloudflare migration**: the new platform ships through CI via a `workers deploy` / `promote` command, so teams deploy reliably without hand-rolled steps. → [[2026-07-cloudflare-pages-to-workers-migration]]
- **This vault**: a small CI (pre-commit / lint) keeps notes consistent.
- **repo-analyzer** (a personal DevSecOps CI gate over Trivy / Checkov / gitleaks / grype / hadolint): mostly AI-generated, so I treat it less as something I built and more as how I **discovered the DevSecOps scope** — running security scanners as blocking, **cloud-agnostic** CI gates. My value there was understanding and testing the concepts.

---

## 💻 Minimal Example

### Context

A minimal pipeline: build → test → security gate → deploy. A failing gate stops everything.

### GitHub Actions (skeleton)

```yaml
on: [push]
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build
      - run: make test          # gate: fails → pipeline stops
      - run: trivy fs .          # security gate (blocking, cloud-agnostic)
      - run: ./deploy.sh         # runs only if every gate passed
```

---

## ⚠️ Pitfalls Experienced

- **A gate that only warns isn't a gate**: if it doesn't block the pipeline, it will be ignored.
- **Slow pipelines get bypassed**: keep CI fast (only run what changed) or people route around it.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> CI vs CD in one line?</summary>

**Answer**: CI = build + test every change; CD = ship a passing change automatically (to a gate = delivery, to prod = deployment).
</details>

<details>
<summary><strong>Q2:</strong> What's wrong with a gate that only warns?</summary>

**Answer**: It's not a gate — if it doesn't block the pipeline, it gets ignored.
</details>

---

## 📊 Stats

```yaml
Total time: ~5h (learning) + hands-on
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: GitHub Actions / CircleCI docs; general CI/CD practice.

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
