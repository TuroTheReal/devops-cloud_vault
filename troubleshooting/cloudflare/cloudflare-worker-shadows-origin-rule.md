---
tags:
  - troubleshooting
  - cloudflare
  - workers
  - origin-rules
  - status/documented
created: 2026-07-21
updated: 2026-07-21
severity: "🔴 Critical"
time-to-fix: 15min
---

# Worker Custom Domain shadows an Origin Rule - Quick Fix

**Related concept**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-origin-rules]]
**Affected**: any hostname served by a Worker Custom Domain that also needs a per-path Origin Rule

---

## 🔍 Symptom

> After putting a hostname (e.g. an apex) behind a Worker, specific paths handled by an
> Origin Rule (e.g. a first-party tag gateway's loader paths, rewritten to a vendor) stop
> returning the vendor's content. A third-party app breaks.

```bash
# The bypass paths no longer return the vendor JS (they return the Worker's response)
for p in loaderA loaderB; do curl -s -o /dev/null -w "$p -> %{content_type}\n" "https://example.com/$p"; done
```

**Scenarios**:
- Cutover of an apex to a Worker via **Custom Domain**.
- Pre-existing Origin Rule on paths of the same hostname.

---

## 🎯 Root Cause

> The hostname was attached to the Worker via a **Custom Domain** → the Worker becomes the
> **origin** for **all** paths → the Origin Rule's paths reach the Worker instead of the
> Origin Rule. The Worker is terminal, there is no origin left to rewrite.

**Why**: see [[cloudflare-custom-domain-vs-route]] (Custom Domain = Worker treated as origin, all paths).

---

## ✅ Quick Fix (no downtime)

### Solution (recommended)
```bash
# 1. Save the state BEFORE (audit + rollback)
curl -s "$API/accounts/$ACCT/workers/domains/$CD_ID" -H "$AUTH" | jq '.result' | tee /tmp/cd.json

# 2. Delete the Worker Custom Domain on the hostname
curl -s -X DELETE "$API/accounts/$ACCT/workers/domains/$CD_ID" -H "$AUTH"

# 3. Create a proxied placeholder DNS record (retry: hostname stays "in use" ~1-2s)
curl -s -X POST "$API/zones/$ZONE/dns_records" -H "$AUTH" \
  -d '{"type":"AAAA","name":"example.com","content":"100::","proxied":true,"ttl":1}'

# 4. Keep the Worker served via a Route
curl -s -X POST "$API/zones/$ZONE/workers/routes" -H "$AUTH" \
  -d '{"pattern":"example.com/*","script":"my-worker"}'
```

**Explanation**: the hostname does DNS-only again → the origin is no longer a Worker → the Origin Rule fires → the paths reach the vendor. The app stays served by the Worker via the Route.

**Verify**:
```bash
curl -sI https://example.com/ | head -1     # 200
# Origin Rule paths -> expected content_type (e.g. javascript)
```

---

## 🛡️ Prevention

- On a hostname that keeps origin-level behavior (Origin Rule, tag gateway): **Route + proxied placeholder record**, NOT a Custom Domain.
- **Rollback written and tested BEFORE** the swap (delete record → recreate Custom Domain). See [[safe-production-changes]].
- Compute the blast radius against the **live system**, not just the IaC (a shared apex can be referenced by invisible aliases/CNAMEs the `terraform plan` won't show).

---

## 🔗 Related

**Deep understanding**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-request-traffic-sequence]]
**Discipline**: [[safe-production-changes]]

---

## 📊 Stats

```yaml
Encountered: 1 time
Last occurrence: 2026-07-21
Typical fix time: 15min
```

---

**Last update**: 2026-07-21
