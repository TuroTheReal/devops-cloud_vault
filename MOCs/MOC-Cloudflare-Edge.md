---
tags:
  - moc
  - cloudflare
  - edge
  - workers
  - dns
  - terraform
  - status/roadmap
created: 2026-07-21
updated: 2026-07-21
status: active
---

# MOC - Cloudflare Edge (Workers, DNS, Cutover)

> **What**: Roadmap of atomic notes to write from a real Pages→Workers edge migration
> and a production incident (a Worker Custom Domain silently shadowed an Origin Rule).
>
> **Who**: me, later. DevOps engineers running edge/CDN cutovers.
>
> **Why**: capture the hard-won distinctions (Custom Domain vs Route, Origin Rules,
> proxied placeholder records) while they're fresh, plus the Terraform + cutover
> discipline around them.

> [!note] How to use this file
> This is a **backlog + learning path**, not a finished MOC. The `[[wikilinks]]`
> below are mostly **unresolved on purpose** — each one marks an atomic note to
> write. Workflow: dump raw understanding in `noteanepasoublierpourplustard.md`
> first if unsure, then consolidate into the target path listed here.
> Keep everything **generic/portable** (this vault is public, CC-BY): use
> `example.com`, `<worker>`, no employer specifics.

---

## 🎯 Learning objective

Understand how Cloudflare routes an incoming request through its products (DNS →
rules → Workers → cache → origin), and why *how* you attach a Worker to a hostname
(Custom Domain vs Route) changes which other features still fire. Then apply the
Terraform ownership + blue-green cutover patterns needed to migrate a fleet of edge
apps without a traffic gap.

**Real-world use case**: migrate apps off a legacy static-hosting product onto edge
Workers, cut over hostname by hostname with a pre-written rollback, and avoid the
trap where making the Worker the origin breaks path-level origin rewrites (e.g. a
first-party analytics gateway).

---

## 📚 Learning path

### Phase 0 — Foundations (generic, provider-agnostic)

- **[[cloud-fundamentals]]** — compute/storage, IaaS/PaaS/SaaS
- **[[edge-computing-fundamentals]]** — edge vs origin, CDN, serverless edge

### Phase 1 — Cloudflare request model (the core) ⭐

1. **[[cloudflare-request-traffic-sequence]]** — the order products run in
2. **[[cloudflare-custom-domain-vs-route]]** — the flagship distinction
3. **[[cloudflare-origin-rules]]** — rewriting the origin per path
4. **[[cloudflare-dns-proxied-records-and-placeholders]]** — orange cloud + dummy records
5. **[[cloudflare-worker-shadows-origin-rule]]** — the incident, distilled (troubleshooting)

### Phase 2 — Workers vs Pages

6. **[[cloudflare-workers-vs-pages]]** — edge compute models, migration rationale
7. **[[first-party-tagging-edge-gateway]]** — what a tag gateway is and why it needs an Origin Rule *(optional)*

### Phase 3 — IaC + cutover discipline

8. **[[terraform-fundamentals]]** (§count footgun) — count list index shifts → destroy/recreate
9. **[[terraform-ignore-changes-partial-ownership]]** — owning a resource partially
10. **[[terraform-reconcile-manual-drift]]** — cut over by hand → align TF → decommission
11. **[[safe-production-changes]]** — rollback-first, the safe cutover loop

### Phase 4 — Reference

12. **[[cloudflare-api-cheatsheet]]** — curl/jq patterns for domains, routes, DNS
13. **[[2026-07-cloudflare-pages-to-workers-migration]]** — the project report tying it together

---

## 📥 Notes backlog — what to write & where

Status: 🟡 = stub created (facts pre-filled, rewrite in your own words) · ✅ = already in the vault · ✍️ = to do yourself.

| # | Note | Type | Target path | Status |
|---|------|------|-------------|--------|
| 1 | `cloudflare-custom-domain-vs-route` | concept | `concepts/cloudflare/` | 🟡 stub |
| 2 | `cloudflare-origin-rules` | concept | `concepts/cloudflare/` | 🟡 stub |
| 3 | `cloudflare-worker-shadows-origin-rule` | troubleshooting | `troubleshooting/cloudflare/` | 🟡 stub |
| 4 | `cloudflare-dns-proxied-records-and-placeholders` | concept | `concepts/cloudflare/` | 🟡 stub |
| 5 | `cloudflare-request-traffic-sequence` | concept | `concepts/cloudflare/` | 🟡 stub (+ open Q) |
| 6 | `cloudflare-workers-vs-pages` | concept | `concepts/cloudflare/` | 🟡 stub |
| 7 | `cloudflare-api-cheatsheet` | cheatsheet | `cheatsheets/cloudflare/` | 🟡 stub |
| 8 | count index footgun / `for_each` | concept | `concepts/terraform/terraform-fundamentals.md` §5 | ✍️ extend (seed provided) |
| 9 | `terraform-ignore-changes-partial-ownership` | concept | `concepts/terraform/` | 🟡 stub |
| 10 | `terraform-reconcile-manual-drift` | concept | `concepts/terraform/` | 🟡 stub |
| 11 | rollback-first cutover | guide | `meta/guides/safe-production-changes.md` | ✅ already present |
| 12 | migration report | project | `projects/2026-07-cloudflare-pages-to-workers-migration/` | 🟡 stub |
| 13 | `first-party-tagging-edge-gateway` | concept | `concepts/networking/` | 🟡 stub |
| 14 | self-documenting prod scripts | guide | `meta/guides/safe-production-changes.md` §2 | ✅ already present |

Folders created: `concepts/cloudflare/`, `cheatsheets/cloudflare/`, `troubleshooting/cloudflare/`,
`projects/2026-07-cloudflare-pages-to-workers-migration/`. (`concepts/deployment/` turned out unnecessary:
the cutover/rollback discipline already lives in `meta/guides/safe-production-changes.md`.)

---

## 🌱 Seeds — the factual kernel per note

> Enough raw material to expand each note in my own words. Facts, not prose.

### 1. cloudflare-custom-domain-vs-route ⭐
**Kernel** — two ways to attach a Worker to a hostname, and they are NOT equivalent:
- **Custom Domain**: "A Worker running on a Custom Domain is **treated as an origin**."
  It points **all paths** of the hostname to the Worker (no wildcards, exact host).
  Cloudflare auto-creates the DNS record + certificate. The Worker becomes terminal
  for that hostname → there is no separate origin to rewrite to.
- **Route** (`example.com/*`): attaches the Worker to a **path pattern**. The zone's
  normal DNS record and pipeline stay in place; the Worker runs for matched requests
  but the hostname still resolves through DNS to a real (or placeholder) origin.
**Why it matters**: if any per-path origin behavior (Origin Rule, tag gateway) must
keep working on that hostname, a Custom Domain can swallow it because the Worker is
the origin for every path. A Route + a plain proxied DNS record preserves it.
**Mental model**: Custom Domain = "the Worker *is* the building". Route = "the Worker
is a doorman who only stops people going to certain floors; the building still exists".

### 2. cloudflare-origin-rules
**Kernel** — Origin Rules change *where/how* Cloudflare connects to the origin for
requests matching a filter (host/path/etc.): override **Host header**, **DNS record /
origin hostname**, **destination port**, **SNI**. Used to serve a third party's
content first-party (e.g. rewrite `/loader-path` to fetch from a vendor's servers)
so it shares your domain (dodges adblockers / third-party-cookie limits).
**Gotcha**: an Origin Rule only fires if the request actually reaches origin
resolution. If a Worker is the origin for that path (Custom Domain), origin
resolution is skipped → the rule is a no-op. See [[cloudflare-worker-shadows-origin-rule]].

### 3. cloudflare-worker-shadows-origin-rule (the incident) ⭐
**Symptom**: after moving an apex hostname onto a Worker, specific loader paths used
by a first-party tag gateway (random-looking paths, served via an Origin Rule that
rewrites them to the vendor) stopped returning the vendor's JS — an unrelated app
broke.
**Cause**: the apex was attached to the Worker via a **Custom Domain** → Worker
became the origin for **every** path → the tag gateway's bypass paths hit the Worker
instead of reaching the **Origin Rule**.
**Fix (no traffic gap)**:
1. Delete the Worker **Custom Domain** on the apex.
2. Create a **plain proxied placeholder DNS record** (`AAAA 100::`, orange-clouded)
   so the apex does DNS-only → origin is no longer a Worker.
3. Keep the Worker serving the app via a **Route** `example.com/*`.
Result: origin resolution happens again → the Origin Rule fires → loader paths reach
the vendor. App still served by the Worker via the route.
**Lesson**: a Worker Custom Domain is not a drop-in for "put the worker on this host".
It makes the Worker the origin and can shadow origin-level features. Prefer Route +
proxied record when the hostname needs other origin behavior.
**Rollback was written and tested BEFORE the swap** (delete record → recreate Custom
Domain). It's what made the swap safe. → [[safe-production-changes]]

### 4. cloudflare-dns-proxied-records-and-placeholders
**Kernel**:
- **Proxied (orange cloud)** = traffic goes through Cloudflare (WAF, cache, Workers,
  rules apply). **DNS-only (grey cloud)** = Cloudflare just answers DNS, no proxy.
- A **placeholder / dummy record** is a proxied record whose content is never really
  contacted — it exists only to give the orange cloud something to attach to so
  Workers/rules can run. Common values: `AAAA 100::` (IPv6 discard) or
  `A 192.0.2.1` (TEST-NET-1 documentation range).
- **Apex** (`example.com`, no subdomain) can't be a CNAME in classic DNS; Cloudflare
  uses **CNAME flattening**, but a proxied A/AAAA placeholder is the simplest apex origin.
**Why**: lets you run a Route/rules on a hostname without a real backend IP.

### 5. cloudflare-request-traffic-sequence
**Kernel** — a request flows through Cloudflare products in a defined order: DNS →
security (WAF/custom rules) → transform/redirect rules → **Origin Rules** → **Workers**
→ cache → origin. Which feature "wins" depends on this order AND on whether a feature
is terminal (a Worker can return a response and stop the chain).
> [!warning] Open question to verify against official docs
> Some docs state "when a Worker route matches, the Worker runs before origin
> resolution, so origin overrides are bypassed" — yet in the incident, a Route (`/*`)
> + plain DNS record let the Origin Rule fire, while a **Custom Domain** did not.
> The *observed* fact is certain; the *exact* Custom-Domain-vs-Route ordering nuance
> is NOT — resolve it from the official "HTTP request traffic sequence" page and the
> Origin Rules docs before asserting a mechanism. Don't guess in the final note.

### 6. cloudflare-workers-vs-pages
**Kernel** — edge compute options: **Workers** = general serverless functions at the
edge (full request control, routing, SSR, APIs). **Pages** = opinionated static-site /
Jamstack hosting built on top of Workers; being de-emphasized for app hosting in
favour of "Workers + static assets". Migration rationale: one primitive (Workers)
for both static assets and dynamic logic, Terraform-manageable, no Pages-specific CI.

### 7. cloudflare-api-cheatsheet
**Kernel** — curl/jq patterns (auth: `Authorization: Bearer $CF_API_TOKEN`):
- Worker Custom Domains: `GET/PUT/DELETE /accounts/{acct}/workers/domains`
- Worker Routes: `POST /zones/{zone}/workers/routes` `{pattern, script}`
- DNS records: `GET/POST/DELETE /zones/{zone}/dns_records`
- Extract an id: `jq -r '.result[]|select(.hostname=="...")|.id'`
- Always save the pre-change object to a file before deleting (`| tee /tmp/x.json`).
- Retry-with-sleep after a delete: CF may still consider the hostname in use for ~1-2s.

### 8. terraform-count-index-footgun
**Kernel** — `count` addresses resources by **numeric index**. Inserting/removing an
item mid-list shifts every later index → Terraform plans **destroy+recreate** for
everything after the change. Use `for_each` with a **stable key** for lists that
mutate. Guard genuinely critical resources with `lifecycle { prevent_destroy = true }`.

### 9. terraform-ignore-changes-partial-ownership
**Kernel** — `lifecycle { ignore_changes = [...] }` lets Terraform own a resource
**partially**: manage its existence/most attributes while an external system (a
deploy pipeline, another tool) owns specific attributes. Trap: "assets-only" partial
ownership can fail if the provider still diffs a required attribute — verify the plan
is truly empty after apply.

### 10. terraform-reconcile-manual-drift
**Kernel** — safe pattern for risky prod changes: **cut over by hand** (API/console)
first to control timing and rollback, THEN **reconcile Terraform** to match reality
(`import` / adjust config until `plan` is empty), THEN **decommission** the old path.
Gap-free = at no point is the hostname unserved. Lock the stacks during the manual
window so no apply races the human.

### 11. blue-green-cutover-with-prewritten-rollback
**Kernel** — cut traffic at the routing layer (DNS record / Worker route), not by
mutating the running app. Before touching prod: **write AND dry-run the rollback
script**, and a **monitor** (poll host+path, print HTTP code + who served via a
response header or body marker; `000`/DOWN = the gap). Order: monitor on → cutover →
observe → (rollback ready) → reconcile. A pre-written rollback is what turns a scary
swap into a reversible one.

### 12. 2026-07-cloudflare-pages-to-workers-migration (project report)
**Kernel** — genericized narrative: fleet of ~N edge apps, Pages→Workers, app by app,
route-based cutover, TF reconcile then Pages decommission. Include the incident above
as the flagship pitfall, and the count/ignore_changes/reconcile notes as what made
the IaC side safe. Link every concept note. Keep it employer-free.

### 13. first-party-tagging-edge-gateway *(optional)*
**Kernel** — a **tag gateway** (e.g. Google Tag Gateway / server-side GTM) serves a
third-party analytics loader from **your own domain** via an edge rewrite (Origin
Rule on specific loader paths → vendor servers). Why: first-party context dodges
ad-blockers and third-party-cookie restrictions, improving measurement. Relevant here
because it's exactly the feature the Custom Domain shadowed.

### 14. self-documenting-prod-scripts *(optional, borderline — could stay in feedback)*
**Kernel** — a script that mutates prod must be **readable while screen-shared during
an incident**: header comment stating what/why/rollback, timestamped step logs, save
state before delete, explicit "RUN ROLLBACK NOW" on failure. The reader shouldn't
have to parse jq to know what just happened.

---

## 🔗 Open questions

1. Exact Cloudflare traffic-sequence ordering: why did **Route + plain DNS** let the
   Origin Rule fire but **Custom Domain** didn't? (verify, don't guess — note 5)
2. Does a proxied placeholder record + Route incur any origin fetch latency vs a
   Custom Domain? (probably not, but confirm)

---

## 📚 Sources (official)

- Cloudflare Workers — Routing / Custom Domains vs Routes: https://developers.cloudflare.com/workers/configuration/routing/
- Cloudflare Origin Rules: https://developers.cloudflare.com/rules/origin-rules/
- Cloudflare HTTP request traffic sequence: https://developers.cloudflare.com/fundamentals/reference/
- Cloudflare DNS — proxy status / CNAME flattening: https://developers.cloudflare.com/dns/
- Terraform `count` vs `for_each`, `lifecycle`: https://developer.hashicorp.com/terraform/language/meta-arguments/for_each

---

**Last update**: 2026-07-21
**Next review**: 2026-08-21 (+1 month)
