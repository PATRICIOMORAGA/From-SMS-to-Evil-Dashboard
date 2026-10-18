# 6. ATT&CK, indicators, limits

## MITRE ATT&CK

| ID | Technique | In this case |
|----|-----------|----------------|
| T1566.002 | Phishing: link | SMS to a shortened URL |
| T1583.001 | Acquire infrastructure: domains | Disposable front, separate durable gate (names withheld) |
| T1583.006 | Acquire infrastructure: web services | Front on S3 and CloudFront |
| T1071.001 | Web protocols | Casino API and helpdesk over HTTPS |
| T1078 | Valid accounts | Default and weak operator accounts |
| T1552 | Unsecured credentials | Admin API returns password hashes; secrets stay weak after rotation |
| T1657 | Financial theft | Advance-fee deposits; no payout in the sample |

## Indicators

**None of the live indicators are in this repository.**

Not published, because the operation was still up: domains, IP addresses, shortened URLs, the Telegram handle, payment-checkout identifiers, merchant display names, operator passwords, bcrypt hashes, and session tokens.

What defenders can hunt without those locators:

- An Internet-facing GOFLY / `go-fly-customer` panel that still serves the install README and `/debug/pprof/` with no login.
- Response header `X-Application: go-fly-customer-service`.
- A casino page whose chat widget uses the default service agent, plus a demand for tax or an account-correction fee before withdrawal.

This report is published openly so that authorities, defenders, and the public can review the case. The live indicators are withheld from this public tree while the campaign is active, and can be shared with competent authorities on request.

## What we did not get

| Question | Result |
|----------|--------|
| JWT signing secret | Not recovered |
| Database password | Not exposed (`/mysql` → 404 on this fork) |
| Full chat archive | No — 25 threads, not ~5,900 |
| Telegram bot token | No |
| File read into config | Not identified |
| Operator’s civil identity | No |
| A GOFLY product CVE for these bugs | No. Open pprof is a known class of bug elsewhere; it is not a CVE id for this panel |
| Proof a specific local payment checkout belongs to this kit | No technical link in the artefacts we kept |

Next: [Recommendations and ethics](07-recommendations-ethics.md).
