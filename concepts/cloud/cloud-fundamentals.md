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
time-to-master: 8h
---

# Cloud Fundamentals - Compute, Storage, Service Models

**Prerequisites**: [[networking-fundamentals]]
**Related to**: [[edge-computing-fundamentals]], [[aws-fundamentals]], [[terraform-fundamentals]], [[docker-why-containers]]

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

- **Placement decisions**: where a workload should live (VM vs container vs serverless vs edge).
- **Right abstraction**: how much infra to manage vs delegate.

### ❌ Bad for

- **Provider-specific detail** → see [[aws-fundamentals]] for a concrete IaaS.

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
Everything a cloud sells is either doing work (compute) or holding state (storage); every managed "service" is just a pre-packaged mix of the two, with more or less operating taken off my plate. So I don't start from "which product", I start from "do I need something that runs or something that retains, and how much do I want to operate it myself".

**Mental model**: compute = the kitchen staff actively cooking (CPU = number of cooks, RAM = counter space); storage = the pantry that just holds ingredients.

### 2. Service models — IaaS / PaaS / SaaS

**Facts**:

| Layer | What it is | Examples |
|---|---|---|
| **IaaS** | Rent VMs / network / storage, manage everything above | EC2, GCE, Hetzner |
| **PaaS** | Deploy code, platform manages the VMs | Heroku, Vercel, Railway, Fly.io, Qovery |
| **SaaS** | Rent finished software | Notion, Slack, Salesforce |

The higher the layer, the less you manage (and the less control / more lock-in).

**My understanding**:
IaaS/PaaS/SaaS is one dial: how much of the stack I hand over. The useful question isn't "which is best" but "at which layer do I want to stop being responsible". Handing over more ships faster but trades away control and increases lock-in, so the choice is really a risk/speed call, not a popularity contest.

**Mental model**: a single dial — raw land (IaaS) → furnished flat (PaaS) → hotel room (SaaS). More furnished = less you do, less you control.

### 3. The trade-off axis

More managed (SaaS) = less ops burden, less control, more lock-in. Less managed (IaaS) = full control, full responsibility. PaaS is the middle ground:
- **Heroku** = historic PaaS (2007), pioneered "git push to deploy"; your code runs on **their** infra.
- **Qovery** = modern PaaS, Heroku-like UX, but your code runs on **your own** K8s (EKS in your VPC) → data residency + control, less lock-in.

---

## 💻 Minimal Example

### Context

Apply the model: map a few common workloads to the right compute + storage.

### Mapping

```text
Static marketing site   → object storage + CDN              (no compute per request)
JSON API                → serverless functions or container + managed DB
Background jobs          → container / serverless + a queue
Global low-latency logic → edge functions + edge KV          (see edge-computing-fundamentals)
```

The point isn't the exact product, it's "runs vs retains, and how much I manage".

---

## ⚠️ Common misconceptions

- **Serverless ≠ no servers**: there are still servers, I just don't manage them; cold starts and per-request limits are the price.
- **Higher abstraction ≠ no lock-in**: the more managed (PaaS/SaaS), the more provider-specific my deploy becomes.

---

## 🧠 Retrieval Practice

<details>
<summary><strong>Q1:</strong> Compute vs storage in one line?</summary>

**Answer**: Compute *runs* code (CPU+RAM); storage *retains* data. Every managed service is a packaged mix of the two.
</details>

<details>
<summary><strong>Q2:</strong> What do you trade going IaaS → SaaS?</summary>

**Answer**: More of the stack handed over — faster to ship, less to operate, but less control and more lock-in.
</details>

---

## 📊 Stats

```yaml
Total time: ~8h (learning)
Used in: [[2026-07-cloudflare-pages-to-workers-migration]]
```

**Sources**: AWS / GCP compute & storage docs; PaaS vendor docs (Heroku, Qovery). Cross-check provider limits before relying on a specific service.

---

**Last update**: 2026-07-22
**Next review**: 2026-08-22
