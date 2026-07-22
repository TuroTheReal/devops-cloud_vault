---
tags:
  - cheatsheet
  - cloudflare
  - api
created: 2026-07-21
version: v4
---

# Cloudflare API - Cheatsheet

**Official docs**: https://developers.cloudflare.com/api/
**Related concepts**: [[cloudflare-custom-domain-vs-route]], [[cloudflare-dns-proxied-records-and-placeholders]]

---

## 🚀 Quick Start

```bash
export CLOUDFLARE_API_TOKEN=***          # never in cleartext in a versioned file
AUTH="Authorization: Bearer $CLOUDFLARE_API_TOKEN"
API=https://api.cloudflare.com/client/v4
ACCT=<account_id>   ZONE=<zone_id>
```

---

## 📚 Commands by Operation

### Worker Custom Domains

```bash
# List / find by hostname
curl -s "$API/accounts/$ACCT/workers/domains?hostname=example.com" -H "$AUTH" \
  | jq -r '.result[]|select(.hostname=="example.com")|.id'

# Get (save before delete)
curl -s "$API/accounts/$ACCT/workers/domains/$CD_ID" -H "$AUTH" | jq '.result'

# Create / update (PUT)
curl -s -X PUT "$API/accounts/$ACCT/workers/domains" -H "$AUTH" \
  -d '{"hostname":"example.com","service":"my-worker","zone_id":"'"$ZONE"'","environment":"production"}'

# Delete
curl -s -X DELETE "$API/accounts/$ACCT/workers/domains/$CD_ID" -H "$AUTH"
```

### Worker Routes

```bash
curl -s -X POST "$API/zones/$ZONE/workers/routes" -H "$AUTH" \
  -d '{"pattern":"example.com/*","script":"my-worker"}'
```

### DNS Records

```bash
# Find
curl -s "$API/zones/$ZONE/dns_records?name=example.com&type=AAAA" -H "$AUTH" | jq -r '.result[].id'
# Create proxied placeholder
curl -s -X POST "$API/zones/$ZONE/dns_records" -H "$AUTH" \
  -d '{"type":"AAAA","name":"example.com","content":"100::","proxied":true,"ttl":1}'
# Delete
curl -s -X DELETE "$API/zones/$ZONE/dns_records/$REC_ID" -H "$AUTH"
```

---

## 🎯 Useful One-Liners

```bash
# Always check success + errors
curl ... | jq '{success, errors}'

# Retry after a delete (hostname may stay "in use" ~1-2s)
for try in 1 2 3 4 5; do OUT=$(curl ...); [ "$(echo "$OUT"|jq -r .success)" = true ] && break; sleep 2; done
```

---

## 💡 Personal Notes

- Save the mutated object BEFORE deleting (`| tee /tmp/x.json`) → audit + rollback.
- Token via env var, never committed. Minimal scope (Workers Routes / DNS Edit on the zone).

**Mistakes made**:
- **Delete-then-create on the same hostname without a retry**: CF still considers the hostname "in use" for ~1-2s, so the create fails and the host briefly 404s. → create-then-delete, or wrap the swap in a retry loop.
- **Deleting without saving state first** → no clean rollback. Always `tee` the object before the delete.
- **Too-broad API token** → scope to exactly Workers Routes + DNS Edit on the one zone.

---

**Last update**: 2026-07-21
