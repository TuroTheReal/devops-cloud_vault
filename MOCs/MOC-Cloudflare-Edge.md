---
tags:
  - moc
  - cloudflare
  - edge
  - workers
  - dns
  - terraform
created: 2026-07-21
updated: 2026-07-22
status: active
---

# MOC - Cloudflare Edge (Workers, DNS, Cutover)

> **What**: Learning path for edge routing on Cloudflare and the safe-cutover discipline around it.
>
> **Who**: DevOps engineers running edge/CDN migrations and cutovers.
>
> **Why**: Migrate a fleet of edge apps hostname by hostname, gap-free, without breaking origin-level features.

---

## 🎯 Learning Objective

Understand how Cloudflare routes a request through its products (DNS → rules → Workers → cache → origin), and why *how* you attach a Worker to a hostname (Custom Domain vs Route) changes which other features still fire. Then apply the Terraform ownership + progressive-cutover patterns needed to migrate a fleet of edge apps without a traffic gap.

**Real-world use case**: move apps off a deprecated static-hosting product onto edge Workers, cut over each hostname with a pre-written rollback, and avoid the trap where making the Worker the origin breaks a path-level origin rewrite (e.g. a first-party analytics gateway).

---

## 📚 Learning Path

### Phase 1: Foundations (14h total)

**Goal**: Understand edge vs origin before touching Cloudflare specifics.

1. **[[concepts/cloud/cloud-fundamentals]]** ⭐⭐ (8h)
   - **Why first**: compute/storage + IaaS/PaaS/SaaS frame everything else
   - **You'll learn**: where a workload lives, and how much of it you manage
2. **[[concepts/cloud/edge-computing-fundamentals]]** ⭐⭐⭐ (6h)
   - **Why now**: edge is the family Cloudflare Workers belongs to
   - **You'll learn**: edge vs origin, CDN → edge compute, serverless-edge constraints

---

### Phase 2: Cloudflare request model (37h total) ⭐

**Goal**: Master how a request flows and how Worker attachment changes behaviour.

3. **[[concepts/cloudflare/cloudflare-request-traffic-sequence]]** ⭐⭐⭐⭐ (6h)
   - **Why now**: the order/terminal model explains every later surprise
   - **You'll learn**: what runs first, and what stops the chain
4. **[[concepts/cloudflare/cloudflare-custom-domain-vs-route]]** ⭐⭐⭐⭐ (12h)
   - **Why now**: the flagship distinction, and the root of the incident
   - **You'll learn**: Custom Domain = Worker is origin; Route = pattern, pipeline stays
5. **[[concepts/cloudflare/cloudflare-origin-rules]]** ⭐⭐⭐ (6h)
   - **Why now**: the feature a Custom Domain can silently bypass
   - **You'll learn**: per-path origin rewrite, and when it no-ops
6. **[[concepts/cloudflare/cloudflare-dns-proxied-records-and-placeholders]]** ⭐⭐⭐ (7h)
   - **Why now**: the placeholder record is half the incident fix
   - **You'll learn**: proxied vs DNS-only, placeholder records, apex/CNAME flattening
7. **[[troubleshooting/cloudflare/cloudflare-worker-shadows-origin-rule]]** (troubleshooting)
   - **Why now**: the incident, distilled — symptom, cause, no-downtime fix
   - **You'll learn**: why sites tied to the origin went down, and the swap that fixed it

---

### Phase 3: Workers vs Pages (9h total)

**Goal**: Know what you're migrating from and to.

8. **[[concepts/cloudflare/cloudflare-workers-vs-pages]]** ⭐⭐ (6h)
   - **Why now**: the migration rationale
   - **You'll learn**: Workers subsumes Pages; migration is not a lift-and-shift
9. **[[concepts/networking/first-party-tagging-edge-gateway]]** ⭐⭐⭐ (3h)
   - **Why now**: it's the exact feature the incident broke
   - **You'll learn**: first-party tagging via an edge rewrite, and its fragility

---

### Phase 4: IaC + cutover discipline (28h total)

**Goal**: Make the fleet migration safe and reproducible.

10. **[[concepts/terraform/terraform-ignore-changes-partial-ownership]]** ⭐⭐ (6h)
    - **Why now**: owning a resource partially when a pipeline owns part of it
    - **You'll learn**: `ignore_changes` as an ownership contract, not drift-hiding
11. **[[concepts/terraform/terraform-reconcile-manual-drift]]** ⭐⭐⭐⭐ (8h)
    - **Why now**: how to align IaC after a manual cutover
    - **You'll learn**: edit code → plan/apply/push → re-apply verify empty
12. **[[concepts/terramate/terramate]]** ⭐⭐⭐ (5h)
    - **Why now**: orchestrating many stacks in a fleet
    - **You'll learn**: stacks, code-gen, change detection, run order
13. **[[concepts/deployment/deployment-strategies]]** ⭐⭐ (4h)
    - **Why now**: choosing how much traffic to expose at once
    - **You'll learn**: big-bang / rolling / blue-green / canary
14. **[[concepts/ci-cd/ci-cd-basics]]** ⭐⭐ (5h)
    - **Why now**: the new platform ships through CI
    - **You'll learn**: build → test → gate → deploy, and why gates must block
15. **[[safe-production-changes]]** (guide)
    - **Why now**: the discipline that ties it together
    - **You'll learn**: rollback-first, canary, readable scripts, blast radius

---

## 🛠️ Practice Project

**Apply your knowledge**: [[2026-07-cloudflare-pages-to-workers-migration]]

**What you'll build**: a gap-free Pages→Workers migration of a fleet, incident included.

**Why this project**: it forces every concept above into one live, reversible cutover.

---

## 📊 Progress Tracking

```yaml
Total estimated time: ~90h
Difficulty: ⭐⭐⭐⭐ (4/5)
Prerequisites: [[concepts/networking/networking-fundamentals]], [[concepts/terraform/terraform-fundamentals]]
```

---

## 🔄 Next Steps

After this path, related directions:
- **[[MOCs/MOC-Infrastructure-as-Code]]**: deeper Terraform + Ansible patterns
- **[[MOCs/MOC-Cloud-AWS]]**: the region-first side of the same cloud picture

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **The one rule**: a Route + a plain/placeholder DNS record keeps origin-level features (Origin Rules) working; a Custom Domain makes the Worker the origin and can bypass them.
2. **Terminal vs pass-through**: a Worker can return a response and stop the chain; a rule only fires if the request reaches its step.
3. **Reconcile = empty plan**: TF and reality agree only when a second `apply` reports no changes.

**Common pitfalls**:
- Custom Domain on a hostname that still needs an Origin Rule → the rule silently no-ops. → [[troubleshooting/cloudflare/cloudflare-worker-shadows-origin-rule]]
- No pre-written, rehearsed rollback → a "safe" cutover isn't safe.

**Cheatsheet**: [[cheatsheets/cloudflare/cloudflare-api-cheatsheet]]

---

**Created**: 2026-07-21
**Last updated**: 2026-07-22
