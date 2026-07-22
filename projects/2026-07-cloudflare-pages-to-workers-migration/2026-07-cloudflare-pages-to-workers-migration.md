---
tags:
  - project
  - cloudflare
  - workers
  - terraform
  - 2026-07
duration: Xh
status: in-progress
---

# Cloudflare Pages → Workers Migration - Learning Report

**Technologies**: Cloudflare Workers, DNS, Origin Rules, Terraform
**Goal**: migrate a fleet of edge apps off a legacy static-hosting product onto Workers, app by app, route-based cutover, with no downtime.
**Repo**: — (work context, kept generic here)

> [!note] Stub — narrative to fill in your own words, kept **generic/portable**. `TODO(you)`.

---

## 🎯 Project Context

### Objective
> TODO(you): why migrate (one primitive for assets + dynamic, IaC-manageable, retire the legacy product).

### Architecture
```
[browser] → Cloudflare (DNS → rules → Worker/route → cache) → assets/origin
```

### Tech Stack
- **Edge compute**: Cloudflare Workers (+ static assets)
- **Routing**: Worker Routes vs Custom Domains, Origin Rules
- **DNS**: proxied records / placeholders
- **IaC**: Terraform (partial ownership, reconcile)

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

---

## ⚠️ Challenges & Solutions

### Challenge 1: A Worker Custom Domain shadowed an Origin Rule (prod incident)
**Problem**: after cutting an apex over to a Worker via Custom Domain, a first-party tag gateway's paths (Origin Rule) broke → a third-party app went down.
**Root Cause**: Custom Domain = Worker origin for all paths → the Origin Rule no longer fires.
**Solution**: swap Custom Domain → proxied placeholder record (`AAAA 100::`) + keep the Worker via Route `example.com/*`.
**Lesson**: Route + placeholder when the hostname needs origin-level behavior.
**Detail**: [[cloudflare-worker-shadows-origin-rule]]

### Challenge 2: safe cutover with no downtime
**Solution**: pre-written + tested rollback, monitor host+path, reconcile TF afterwards. → [[safe-production-changes]]

---

## 🔧 Reusable Assets

**Scripts** (generic, out of repo):
- `route-cutover.sh` / `route-rollback.sh` — create/delete Worker route
- `apex-swap.sh` / `apex-rollback.sh` — swap Custom Domain ↔ placeholder record
- `monitor.sh` — poll host+path, who serves (header/body marker), detect the gap

**Cheatsheet**: [[cloudflare-api-cheatsheet]]

---

## 🎯 Outcomes

### What Worked Well
1. Pre-written rollback → the apex incident was reversible in one command.
> TODO(you).

### What Could Be Improved
1. Blast radius: better map live references (aliases/CNAMEs) before touching an apex.
> TODO(you).

---

## 📝 Extractions TODO

- [ ] Resolve the pipeline-order open question → [[cloudflare-request-traffic-sequence]]
- [ ] Extend [[terraform-fundamentals]] §count with the index/for_each footgun
- [ ] Time breakdown

---

**Started**: 2026-07
**Last updated**: 2026-07-21
