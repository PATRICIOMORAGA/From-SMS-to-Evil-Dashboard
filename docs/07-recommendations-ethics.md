# 7. Recommendations and ethics

## Recommendations

**CSIRTs and hosting abuse teams.** Prioritise the durable gate (API and helpdesk together), not only the CDN website. Use an authorized indicator exchange. Do not reconstruct targets from this repository.

**Payment-abuse teams.** Treat “pay a tax in the chat to withdraw casino winnings” as a typology even when the receipt is a normal local payment link. This report does not assert that one observed checkout belongs to this kit.

**People who receive the SMS.** If support asks for money before you can withdraw, stop. A bank does not collect tax inside a game chat. Do not open unexpected shortened links. If you already paid: keep receipts, call your bank, report it to your national CSIRT, and do not send more.

## Method and ethics

- Original research by **Patricio Moraga** and **Anthony de la Paz**, for a technical talk at 8.8 Unreal Chile. Not a police statement and not a product advertisement.
- Collection mixed public pages (HTML, JavaScript, login form, profiler) with use of the operator panel’s own weak authentication. This repository does not include steps, commands, or secrets that would recreate that access.
- No payment-card data was observed.
- No victim phone numbers, national IDs, or emails are stored here.
- Quotes are tied only to internal user ids.
- Classic defanging is rejected for this public edition: `example[.]com` and `hxxps://` still name the actor while the campaign is active.

## Disclosure and timeline

| Date (2026) | Event |
|-------------|-------|
| 11 Aug | Earliest victim message in the reviewed threads |
| 19 Aug | Main collection; first password lockout and host instability observed the same day |
| 19–22 Aug | Second super-admin confirmed; operation assessed as unchanged |
| 22 Aug | Case documented for the 8.8 Unreal Chile talk |
| On release | Published openly so authorities, defenders, and the public can review it; live indicators withheld while the campaign is active, available to competent authorities on request |

## References

- MITRE ATT&CK, Enterprise matrix. <https://attack.mitre.org>
- GOFLY (`go-fly`) open-source customer-service project. <https://github.com/taoshihan1991/go-fly>
- Hashcat, example hashes / JWT mode 16500. <https://hashcat.net/wiki/doku.php?id=example_hashes>
- Go standard library, `net/http/pprof`. <https://pkg.go.dev/net/http/pprof>
- OWASP, JSON Web Token and credential-management guidance. <https://owasp.org>
- CSIRT de Gobierno de Chile, public smishing advisories (2024–2025). <https://www.csirt.gob.cl>

## Citation

Moraga, P., & de la Paz, A. (2026). *From SMS to the operator panel: hunting a live fake-casino smishing campaign.* Threat intelligence case study, 8.8 Unreal Chile. Public edition withholds infrastructure identifiers while the campaign remains active.
