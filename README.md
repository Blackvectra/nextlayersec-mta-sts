# MTA-STS Policy Host — nextlayersec.io

This repository serves the MTA-STS policy for `nextlayersec.io` via GitHub Pages.  
Policy is accessible at: `https://mta-sts.nextlayersec.io/.well-known/mta-sts.txt`

---

## What is MTA-STS?

MTA-STS (Mail Transfer Agent Strict Transport Security) is an email security standard  
defined in [RFC 8461](https://datatracker.ietf.org/doc/html/rfc8461) that allows domain  
owners to enforce TLS encryption on inbound SMTP connections.

Unlike SPF, DKIM, and DMARC — which authenticate the sender — MTA-STS protects  
the transport layer. It prevents SMTP downgrade attacks and DNS-based MX hijacking  
by forcing sending MTAs to validate TLS certificates before delivering mail.

---

## Repository Structure
.nojekyll                  ← disables Jekyll to allow .well-known/ serving
.well-known/
mta-sts.txt              ← MTA-STS policy file
CNAME                      ← custom domain: mta-sts.nextlayersec.io
README.md

---

## Policy Configuration
version: STSv1
mode: enforce
mx: nextlayersec-io.mail.protection.outlook.com
max_age: 604800
| Field | Value | Notes |
|---|---|---|
| `version` | STSv1 | Only supported version |
| `mode` | enforce | Sending MTAs must validate TLS or reject delivery |
| `mx` | nextlayersec-io.mail.protection.outlook.com | Must match live MX record exactly |
| `max_age` | 604800 | Policy cache TTL — 7 days |

---

## DNS Records

| Type | Name | Value | Notes |
|---|---|---|---|
| CNAME | `mta-sts` | `blackvectra.github.io` | DNS-only, no proxy |
| TXT | `_mta-sts` | `v=STSv1; id=20260416` | Bump `id` on every policy change |
| TXT | `_smtp._tls` | `v=TLSRPTv1; rua=mailto:tlsrpt@nextlayersec.io` | TLS failure reporting |

---

## Deployment Notes

**Why `.nojekyll` is required**  
GitHub Pages uses Jekyll by default, which silently blocks dotfolders including `.well-known/`.  
An empty `.nojekyll` file disables Jekyll processing and allows static file serving.

**Why Cloudflare proxy must be disabled**  
GitHub Pages handles TLS termination for custom domains. Cloudflare proxying (orange cloud)  
intercepts this and breaks certificate provisioning. CNAME must be DNS-only (grey cloud).

**Updating the policy**  
Whenever `mta-sts.txt` is modified, the `id` value in the `_mta-sts` TXT record must be  
updated to a new value. Sending MTAs cache the policy — a changed `id` signals them  
to re-fetch. Use the current date in `YYYYMMDD` format as a simple versioning convention.

---

## Validation

```bash
# Verify policy file is serving
curl https://mta-sts.nextlayersec.io/.well-known/mta-sts.txt

# Verify DNS TXT record
nslookup -type=TXT _mta-sts.nextlayersec.io

# Full MTA-STS lookup
# https://mxtoolbox.com/SuperTool.aspx → MTA-STS Lookup → nextlayersec.io
```

---

## Related

- [MTA-STS Policy Host — nrgtechservices.com](https://github.com/Blackvectra/blackvectra.github.io)
- RFC 8461 — SMTP MTA Strict Transport Security
- RFC 8460 — SMTP TLS Reporting

---

*Maintained by [NextLayerSec](https://nextlayersec.io)*
