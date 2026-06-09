# MozaWeb DRM Bypass — Security Research

Independent security audit of [MozaWeb](https://hr.mozaweb.com) (Mozaik Education), a digital textbook platform used in Croatian schools.

## Vulnerabilities Discovered

| # | Vulnerability | CWE | Severity |
|---|---|---|---|
| 1 | Hardcoded AES key in client JavaScript | [CWE-321](https://cwe.mitre.org/data/definitions/321.html) | **CRITICAL** |
| 2 | API returns book data without ownership verification | [CWE-862](https://cwe.mitre.org/data/definitions/862.html) | **CRITICAL** |
| 3 | Predictable Book IDs (IDOR) | [CWE-639](https://cwe.mitre.org/data/definitions/639.html) | HIGH |
| 4 | Internal path disclosure via `dirWeb` | [CWE-200](https://cwe.mitre.org/data/definitions/200.html) | MEDIUM |
| 5 | All security logic is client-side only | [CWE-602](https://cwe.mitre.org/data/definitions/602.html) | **CRITICAL** |

## TL;DR

MozaWeb's DRM relies on a **hardcoded AES key** (`WWxIuw81Bu0xY`) embedded in client-side JavaScript. Combined with an API that serves book metadata for any Book ID regardless of purchase status, this allows automated extraction of all textbook content.

```
Catalog → Scrape all Book IDs
    ↓
API /Mblite/api/load → Get dirWeb, startpage, maxpage (no auth check)
    ↓
Encrypt image path with hardcoded key → AES(dirWeb + BID + '_' + page + '.jpg')
    ↓
GET mbLite/?ct=...&iv=...&s=... → Server decrypts, returns JPG
```

## Attack Chain

1. **Scrape catalog** — public endpoint lists all Book IDs in format `HR-{PUBLISHER}-{SUBJECT}{GRADE}-{NUMBER}`
2. **Enumerate via API** — `POST /hr/Mblite/api/load` with `bid=<any_id>` returns page count and image directory
3. **Construct & encrypt URLs** — using the hardcoded AES key from client JS
4. **Download** — server decrypts the URL and returns full-resolution page images. No ownership check, no rate limiting

## Status

> **⚠️ NOT FIXED** — All vulnerabilities remain exploitable in production as of June 2026.

Responsible disclosure was attempted before publication.

## Full Write-up

Detailed technical analysis: [Breaking MozaWeb's Client-Side DRM](https://dnikiforov.dev/blog/mozaweb-idor)

## Disclaimer

This research was conducted independently for educational purposes. No copyrighted content is distributed through this repository.
