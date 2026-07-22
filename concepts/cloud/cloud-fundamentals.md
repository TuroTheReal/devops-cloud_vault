---
tags:
  - concept
  - cloud
  - fundamentals
  - iaas-paas-saas
  - compute
  - storage
  - status/learning
created: 2026-07-22
difficulty: 2
time-to-master: 4h
---

# Cloud Fundamentals - Compute, Storage, Service Models

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[edge-computing-fundamentals]], [[aws-fundamentals]], [[terraform-fundamentals]], [[docker-why-containers]]

> [!note] Stub — facts + my own scratch material pre-filled, rewrite "My understanding" in my words. `TODO(you)`.

---

## 🎯 TL;DR (30 seconds)

> The cloud rents two things: **compute** (runs code) and **storage** (retains data),
> at three levels of responsibility: **IaaS** (rent the machines), **PaaS** (rent the
> platform), **SaaS** (rent the finished software).

**Memorable line**: *Compute runs, storage retains.*

**Analogy (kitchen)**: IaaS = empty kitchen (you bring everything), PaaS = equipped kitchen (just cook), SaaS = restaurant (you just eat).

---

## 🤔 When to Use?

### ✅ Good for
- Reasoning about where a workload should live (VM vs container vs serverless vs edge).
- Choosing the right abstraction: how much infra do I want to manage vs delegate.

### ❌ Bad for
- Provider-specific detail → see [[aws-fundamentals]] for a concrete IaaS.

---

## 📚 Key Concepts

### 1. Compute vs Storage
**Facts**:
- **Compute** = capacity to execute code (CPU + RAM running). CPU ≈ number of cooks; RAM ≈ size of the workbench (bigger = fewer trips to the fridge).
- **Storage** = capacity to retain data. Conceptual opposite of compute.

| Compute type | Example |
|---|---|
| VMs | EC2, GCE, Hetzner |
| Orchestrated containers | ECS, EKS, GKE, Fargate |
| Serverless functions | Lambda, Cloudflare Workers, Cloud Functions |
| Edge compute | Cloudflare Workers, Lambda@Edge → [[edge-computing-fundamentals]] |

| Storage type | Example |
|---|---|
| Object | S3, GCS, R2 |
| Block | EBS, GCE Persistent Disk |
| Managed DB | RDS, Cloud SQL, DynamoDB |
| Cache | ElastiCache, Memorystore |

**My understanding**:
> TODO(you)

### 2. Service models — IaaS / PaaS / SaaS
**Facts**:

| Layer | What it is | Examples |
|---|---|---|
| **IaaS** | Rent VMs / network / storage, manage everything above | EC2, GCE, Hetzner |
| **PaaS** | Deploy code, platform manages the VMs | Heroku, Vercel, Railway, Fly.io, Qovery |
| **SaaS** | Rent finished software | Notion, Slack, Salesforce |

The higher the layer, the less you manage (and the less control / more lock-in).

**My understanding**:
> TODO(you)

### 3. The trade-off axis
More managed (SaaS) = less ops burden, less control, more lock-in. Less managed (IaaS) = full control, full responsibility. PaaS is the middle ground:
- **Heroku** = historic PaaS (2007), pioneered "git push to deploy"; your code runs on **their** infra.
- **Qovery** = modern PaaS, Heroku-like UX, but your code runs on **your own** K8s (EKS in your VPC) → data residency + control, less lock-in.

---

## ⚠️ Pitfalls Experienced
> TODO(you).

---

## 📊 Stats

```yaml
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: extracted from my `noteanepasoublierpourplustard` Cloud — Fundamentals notes; validate against provider docs.

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
