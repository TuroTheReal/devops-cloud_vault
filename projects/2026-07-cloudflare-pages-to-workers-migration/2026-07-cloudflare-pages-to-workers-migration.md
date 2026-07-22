---
tags:
  - project
  - cloudflare
  - workers
  - terraform
  - 2026-07
duration: ongoing (~2 months)
status: in-progress
---

# Cloudflare Pages → Workers Migration - Learning Report

**Technologies**: Cloudflare Workers, DNS, Origin Rules, Zero Trust, Terraform, CI
**Goal**: migrate a fleet of edge apps off a legacy static-hosting product onto Workers, app by app, route-based cutover, with no customer-facing downtime.
**Repo**: — (work context, kept generic/portable here)

---

## 🎯 Project Context

### Objective

The static-hosting product is being deprecated, and the whole public web surface runs on it. Migrating to Workers gives one primitive for both static assets and dynamic logic, fully IaC-manageable, with no product-specific CI to maintain, and retires the deprecated platform ahead of its sunset. The bar: every cutover safe, reversible, observable, and invisible to customers.

### Architecture

```
[browser] → Cloudflare (DNS → rules → Worker/route → cache) → assets/origin
```

### Tech Stack

- **Edge compute**: Cloudflare Workers (+ static assets)
- **Routing**: Worker Routes vs Custom Domains, Origin Rules
- **DNS**: proxied records / placeholders
- **Auth**: Zero Trust (Access) on protected hostnames
- **IaC**: Terraform (partial ownership, reconcile)
- **CI**: deploy/promote through a CLI command

---

## 📚 What I Learned

### 1. Custom Domain vs Route

- **Key insight**: Custom Domain = Worker becomes origin (all paths); Route = pattern, preserves the pipeline.
- **Extracted to**: [[cloudflare-custom-domain-vs-route]]

### 2. Origin Rules & pipeline

- **Extracted to**: [[cloudflare-origin-rules]], [[cloudflare-request-traffic-sequence]]

### 3. DNS proxied / placeholders

- **Extracted to**: [[cloudflare-dns-proxied-records-and-placeholders]]

### 4. IaC ownership & reconcile

- **Extracted to**: [[terraform-ignore-changes-partial-ownership]], [[terraform-reconcile-manual-drift]]

### 5. Edge fundamentals underneath it all

- **Extracted to**: [[edge-computing-fundamentals]], [[cloud-fundamentals]]

---

## ⚠️ Challenges & Solutions

### Challenge 1: A Worker Custom Domain shadowed an Origin Rule (prod incident)

**Problem**: after cutting an apex over to a Worker via Custom Domain, a first-party tag gateway's paths (Origin Rule) broke → the sites depending on that origin behaviour went down.
**Root Cause**: Custom Domain = Worker origin for all paths → the Origin Rule no longer fires.
**Solution**: swap Custom Domain → proxied placeholder record (`AAAA 100::`) + keep the Worker via Route `example.com/*`.
**Lesson**: Route + placeholder when the hostname needs origin-level behavior.
**Detail**: [[cloudflare-worker-shadows-origin-rule]]

### Challenge 2: safe cutover with no downtime

**Solution**: pre-written + tested rollback, a canary on the riskiest hostname, a monitor on host+path, and TF reconcile afterwards. → [[safe-production-changes]]

---

## 🔎 What to watch (hard-won)

- **Security tests differ sandbox vs real**: some checks can't run identically in a sandbox, so you sometimes have to modify the app (notably **CORS**) just to be able to test the flow there. Track those temporary changes and revert them.
- **Zero Trust + DNS**: watch **DNS redirects on ZT-protected hostnames** (and other auth layers); a routing change can quietly break the auth path.
- **Sandbox thoroughly** before prod, on the real setup, not a simplified one.
- **Test the rollbacks**, don't just write them: rehearse the swap+rollback on a throwaway env.
- **App config is not uniform**: SPA vs SSR vs static behave differently on Workers (routing, fallbacks, headers), and **the builds** differ too, so review each app's config and build output, don't re-point blindly.

---

## 🔧 Reusable Assets

**Scripts** (generic, out of repo):
- `route-cutover.sh` / `route-rollback.sh` — create/delete Worker route
- `apex-swap.sh` / `apex-rollback.sh` — swap Custom Domain ↔ placeholder record
- `monitor.sh` — poll host+path, who serves (header/body marker), detect the gap
- `purge-pages-deployments.sh` / `trim-pages-deployments.sh` — clean up old deployments when decommissioning the legacy platform (gotcha: you can't delete the **live/production** deployment, only stale ones)

**CI**: the new platform is shipped through CI via a `workers deploy` / `promote` command, so teams deploy reliably without hand-rolled steps. → [[ci-cd-basics]]

**Cheatsheet**: [[cloudflare-api-cheatsheet]]

---

## 🎯 Outcomes

### What Worked Well

1. **Pre-written rollback** → the apex incident was reversible in one command, no maintenance window.
2. **Canary on the riskiest hostname** → the highest-traffic property migrated observable and reversible instead of as a big-bang cutover.
3. **Reproducing the setup in a sandbox first** → caught silent failure modes before prod.

### What Could Be Improved

1. **Blast radius**: map live references (aliases / flattened CNAMEs) before touching a shared apex, the IaC plan won't show them.
2. **Ship smaller slices earlier**: on ambiguous parts I sometimes scoped several improvements at once; a smaller first slice would have created signal sooner.

---

## 📝 Follow-ups

- [ ] Resolve the pipeline-order open question → [[cloudflare-request-traffic-sequence]]
- [ ] Extend [[terraform-fundamentals]] §count with the index/for_each footgun
- [ ] Migrate the remaining special-case stacks (dynamic domains, non-standard setups)

---

**Started**: 2026-07
**Last updated**: 2026-07-22
