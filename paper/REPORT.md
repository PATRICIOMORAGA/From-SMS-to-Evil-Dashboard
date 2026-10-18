# From SMS to the operator panel: hunting a live fake-casino smishing campaign

**Threat intelligence case study**  
**TLP:WHITE** for this narrative · **Live infrastructure identifiers: withheld (TLP:AMBER)**

| | |
|--|--|
| **Authors** | Patricio Moraga · Anthony de la Paz |
| **Venue** | 8.8 Unreal Chile |
| **Window** | 11–22 August 2026 (main collection: 19 August 2026) |
| **Status** | Assessed **still active**, or trivially movable onto the same kit |

**Abstract.** Original research into a smishing campaign that does not impersonate a bank: an SMS leads to a fake casino, and the extra charges happen in the operator chat. This text tells the hunt (entry, loss of access, failure to recover the JWT secret, continuation through another admin account) without domains, IPs, shorteners, bots, or passwords. Those indicators are not published while the threat remains active.

> **Handling.** Classic defanging (`example[.]com`, `hxxps://`) still identifies the actor. It is not used. Nothing in this repository should be treated as a locator, a login, or a reproduction guide.

---

## 1. Executive summary

We tracked a Spanish-language **smishing** operation aimed at people in Latin America, with clear impact in Chile. The lure is not a bank, a tax agency, or a courier. It is a **fake online casino**: an SMS, a shortened link, a disposable website, then a durable back end that takes deposits and runs a live helpdesk.

The helpdesk is a fork of the open-source product **GOFLY** (`go-fly-customer`), on the same host as the casino API. Operators do not steal card data in the chats we reviewed. They keep the victim inside a payment loop: a fake “bank tax” (often about 10%), a fee to “correct” the withdrawal account, a small top-up “to verify” the account, and in-game “tasks.” In the sample, **nobody was paid out**.

The hunt had four beats:

1. **Entry.** The public login form advertised a decoy agent. That account did not work. The agent actually embedded in the casino site was the product’s default service account, protected by a trivial password.
2. **Lockout.** After we had used that access to record scale and chats, the operator rotated that password. The previous login failed. Parts of the host also timed out or returned gateway errors.
3. **The token was not the door.** Sessions are JWTs (HMAC-SHA256, about 24 hours). The signing secret is **not** the upstream GOFLY default, and we did **not** recover it. Forging tokens was not how access continued. Already issued tokens also stayed valid until they expired, even after the password change.
4. **The other admin.** The panel’s own admin API returns bcrypt password hashes to a super-admin. From a hash collected before the lockout, an offline guess recovered the second super-admin. That account had the same privileges. Its password was the same weak pattern with a cosmetic prefix, not a real rotation. We used it only to confirm the operation was unchanged. Passwords are **not** printed here.

On 19 August 2026 the panel showed on the order of **5,900 visitors** and **57,700 messages**. We read **25 conversations** (about 0.42% of visitors). Recorded top-ups in that sample add up to about **CLP 89.7 million**. No card numbers were seen. Victim phone numbers, national IDs, and emails are not in this report.

---

## 2. Why this is not “bank smishing”

| Classic smishing | This campaign |
|------------------|---------------|
| SMS pretends to be your bank, tax office, or parcel | SMS sells credits, a bonus, or a casino |
| Goal is often a credential or a card on a cloned site | Goal is a deposit, then more deposits inside chat |
| One phishing page | Disposable front + durable operator gate |
| Victim may notice a fake domain | Victim is already “playing”; the scam is the withdrawal that never arrives |

That shift matters for detection. Filters aimed at “your account will be closed” miss “your bonus is waiting.”

---

## 3. The hunt

Dates below are from the observation window in August 2026. Infrastructure names are omitted on purpose.

### 3.1 The front gives away the kit

The website is a Vue single-page application served from Amazon S3 and CloudFront. It is built to be thrown away. The client itself points the lobby at one durable host: casino API plus helpdesk. Affiliate tags on the link (`tr`, `r`) show the SMS layer is a distribution network, not a single text.

The page also leaks history without any login:

- JavaScript comments still list **older API gates**, including one raw IP, with the current gate left uncommented.
- Open Graph tags still advertise a **previous** front (old domain, shared image path). The brand on the glass changes; the mould does not.

A Telegram bot is referenced in the same bundle. A passive look at the public profile showed an empty contact card, not a live “Start bot” page. The handle is withheld.

### 3.2 The decoy on the login form

The helpdesk is GOFLY, Internet-facing, including the upstream **install README** at the site root. The login page is pre-filled with an agent name and a default-looking password. That pair **failed**. It is bait, or a leftover from the stock template.

The casino embed does not call that decoy. It calls the product’s real default agent: the service account. That password was trivial and **worked**. The API expects the session in an HTTP header named `token` (a JWT), not `Authorization: Bearer`. The role on that token was super-admin.

### 3.3 What the first session showed

With that session we recorded, without taking a full dump:

- Panel statistics: about 5,900 visitors and 57,700 messages.
- A page of visitor records (500), then full threads for 25 numeric user ids.
- The agent list. Two super-admins. The API returned **bcrypt password hashes** for both. Publishing those hashes would help anyone attack the same panel, so they are not included.
- No MySQL password. The settings screen exists; the API route that used to return the database password responds **404** on this fork.

### 3.4 The operator cut the key

Later the same day the service account stopped accepting the old password. The host was also unstable (timeouts and HTTP 502). That was a real lockout of the password, not of every session: JWTs already issued **kept working until their expiry**, on the order of 24 hours. Changing the password did not revoke tokens.

### 3.5 We tried the JWT. It did not open

We tested whether the signature used the well-known GOFLY development secret, other short defaults, and common wordlists (HMAC-SHA256 / hashcat mode 16500 and an independent HMAC check). **No secret was found.** Unauthenticated heap profiles (`/debug/pprof/`, text mode) show call stacks, not a readable signing key. So the second door was not “we minted our own admin token.”

### 3.6 The other super-admin

The second account had been created later than the stock service user and had the same super-admin role. Its bcrypt hash was already in the earlier export. An offline dictionary guess matched a password that is the **same weak numeric pattern** as the first account, with a short prefix. That is a rotation in appearance only.

A normal login with that account issued a **new** JWT. It saw the same panel, the same visitor counts, and the same hash-export API. We then read the service account’s **new** hash (the operator had changed it) and, again offline, matched it to another weak variant of the same pattern. Those secrets are not written here. The finding is the pattern: both super-admins, before and after the lockout, were guessable, and the application helped by returning hashes to anyone who was already an admin.

### 3.7 What the debug endpoint added

`/debug/pprof/` answered with **no token**. That is a deployment mistake (Go’s profiler registered without the JWT middleware), not a CVE in GOFLY. It disclosed:

- the binary name and a developer filesystem path from the build machine (a macOS developer workstation);
- signals that the live host is likely on AWS (inferred, not confirmed), consistent with the front being served from AWS infrastructure;
- enough CPU/heap surface to degrade the process (`profile`, large heap dumps).

It did **not** hand over the JWT secret or a database URL. Assigning some other product’s pprof CVE to this host would be wrong; the **class** of bug (profiler open on a public port) is the same.

---

## 4. Attack chain

```
Unsolicited SMS
  → shortened URL + affiliate tags
  → disposable casino site (object storage + CDN)
  → durable host: casino API + GOFLY helpdesk
  → chat script: tax, “fix account”, tasks
  → further deposits, no withdrawal in the sample
```

The site visitors see can be replaced in days. The business sits on the gate: API and chat on one host. Taking down only the front page does not stop the operators.

---

## 5. Operator script, in the victims’ words

Reviewed threads are identified only by the panel’s internal user id. Excerpts are shortened. Where a slide deck reordered lines for effect, **this section follows the chat timestamps**.

The script, repeated across threads:

1. A withdrawal “failed” or is “under review.”
2. A bank, not the operator, supposedly demands about **10%** “income tax” (sometimes 5%) before the payout.
3. Or the payout account is “wrong,” and a further fixed top-up (often on the order of **CLP 50,000**, sometimes a smaller “verification” amount) will “correct” it.
4. If the victim has already paid a lot, the operator assigns **tasks**.
5. Each payment creates a new excuse. The withdrawal does not arrive.

**User 900274** (3 top-ups, about CLP 6.6 million), 19 August, evening. They say the withdrawal is not showing. The operator replies that the bank reported an anomaly and that 10% personal income tax must be paid before withdrawal. The user answers that they will not deposit again: they “fell straight into the scam.” A further operator message insisting on the tax is timestamped **after** that refusal, not before.

**User 725434** (4 top-ups, about CLP 6.1 million), 11–19 August. The thread is days long, not one screen. The user asks “Is this a scam?” **before** the full “10% anomaly” paragraph in that day’s sequence, and later says they do not have the money for the tax and demands the funds back. Days later the operator claims a transfer failed and the account must be corrected. Collapsing this into a single bubble reverses the order.

**User 869331** (4 top-ups, about CLP 4.6 million), 19 August. The user threatens to post screenshots, then calls the operators thieves, and says they will send evidence to an app store. The operator’s answer is another top-up (about CLP 50,000) to “change the withdrawal account” so the bank will pay. The demand to return the money sits around that demand, not as a tidy ending written for a slide.

**User 826038** (about CLP 7.0 million) says they have already deposited and asks whether they are about to be scammed. **User 889768** (about CLP 12.1 million) describes the bank story and the extra deposits as something the operators invent, and asks for the money back. **User 881495** is the largest in the sample (5 top-ups, about CLP 13.6 million) and also cannot withdraw.

Sixteen of the 25 reviewed users had at least one recorded top-up. The sample sum is about **CLP 89.7 million**. “Balance” inside the game is not money received.

---

## 6. Figures (sample, not a census)

**Sampling.** The 25 conversations were a **purposive sample, not a random one**. Starting from one page of visitor records (500), we followed the threads with recorded activity and deposits first. That choice deliberately over-represents paying victims: the per-user amounts and the 16-of-25 top-up ratio are an upper-leaning view of engaged users, not a panel-wide average. The totals describe this sample only and should not be multiplied out to all ~5,900 visitors.

| Measure | Value |
|---------|--------|
| Visitors on the panel | ~5,900 |
| Messages | ~57,700 |
| Conversations read | 25 (~0.42%) |
| Of those, with a top-up | 16 |
| Sum of those top-ups | ~CLP 89.7 million (~USD 94,000) |
| Direct “this is a scam / you are thieves” lines | 5 users in the structured review |

Chilean-peso amounts are converted at an approximate rate of ~950 CLP/USD for the observation window; treat USD figures as rounded estimates, not accounting.

---

## 7. MITRE ATT&CK

| ID | Technique | In this case |
|----|-----------|----------------|
| T1566.002 | Phishing: link | SMS to a shortened URL |
| T1583.001 | Acquire infrastructure: domains | Disposable front, separate durable gate (names withheld) |
| T1583.006 | Acquire infrastructure: web services | Front on S3 and CloudFront |
| T1071.001 | Web protocols | Casino API and helpdesk over HTTPS |
| T1078 | Valid accounts | Default and weak operator accounts |
| T1552 | Unsecured credentials | Admin API returns password hashes; secrets stay weak after “rotation” |
| T1657 | Financial theft | Advance-fee deposits; no payout in the sample |

---

## 8. Indicators

**None of the live indicators are in this repository.**

Not published, because the operation was still up when this was written: domains, IP addresses, shortened URLs, the Telegram handle, payment-checkout identifiers, merchant display names, operator passwords, bcrypt hashes, and session tokens.

What defenders can use without those locators:

- An Internet-facing GOFLY / `go-fly-customer` panel that still serves the install README and `/debug/pprof/` with no login.
- Response header `X-Application: go-fly-customer-service`.
- A casino page whose chat widget uses the default service agent, combined with a demand for tax or an account-correction fee before withdrawal.
- User reports using those phrases.

This report is published openly so that authorities, defenders, and the public can review the case. The live indicators above are withheld from this public file while the campaign is active, and can be shared with competent authorities on request.

---

## 9. What we did not get

| Question | Result |
|----------|--------|
| JWT signing secret | Not recovered |
| Database password | Not exposed by this fork (`/mysql` → 404) |
| Full chat archive | No — 25 threads, not ~5,900 |
| Telegram bot token | No |
| File read / path traversal into config | Not identified |
| Operator’s civil identity | No |
| A GOFLY product CVE that matches these bugs | No. Unauthenticated pprof is a known *class* of bug in other software; it is not a CVE number for this panel |
| Proof that a specific Chilean payment checkout belongs to this kit | No technical link in the artefacts we kept |

---

## 10. Recommendations

**CSIRTs and hosting abuse teams.** Prioritise the durable gate (API and helpdesk together), not only the CDN website. Use an authorized indicator exchange; do not reconstruct targets from this page.

**Payment-abuse teams.** Treat “pay a tax in the chat to withdraw casino winnings” as a typology even when the receipt is a normal local payment link. We are not asserting that one observed checkout belongs to this kit.

**People who receive the SMS.** If support asks for money before you can withdraw, stop. A bank does not collect tax inside a game chat. Do not open unexpected shortened links. If you already paid: keep receipts, call your bank, report it to your national CSIRT, and do not send more.

---

## 11. Method and ethics

- Original research by **Patricio Moraga** and **Anthony de la Paz**, for a technical talk at 8.8 Unreal Chile. Not a police statement and not a product advertisement.
- Collection mixed public pages (HTML, JavaScript, login form, profiler) with use of the operator panel’s own weak authentication. We are not releasing steps, commands, or secrets that would recreate that access.
- No payment-card data was observed.
- No victim phone numbers, national IDs, or emails are stored in this report.
- Quotes are tied only to internal user ids.

---

## 12. Timeline and publication

| Date (2026) | Event |
|-------------|-------|
| 11 Aug | Earliest victim message in the reviewed threads |
| 19 Aug | Main collection: panel read; first password lockout observed the same day; host instability (timeouts, HTTP 502) |
| 19–22 Aug | Second super-admin confirmed; operation assessed as unchanged |
| 22 Aug | Case documented for the 8.8 Unreal Chile talk |
| On release | Published openly so authorities, defenders, and the public can review it; live indicators withheld from this public edition while the campaign is active, available to competent authorities on request |

---

## 13. References

- MITRE ATT&CK, Enterprise matrix and technique catalogue. <https://attack.mitre.org>
- GOFLY (`go-fly`) open-source customer-service project. <https://github.com/taoshihan1991/go-fly>
- Hashcat, example hashes and JWT mode 16500 (HMAC-SHA256). <https://hashcat.net/wiki/doku.php?id=example_hashes>
- Go standard library, `net/http/pprof` profiling endpoints. <https://pkg.go.dev/net/http/pprof>
- OWASP, JSON Web Token (JWT) and credential-management guidance. <https://owasp.org>
- CSIRT de Gobierno de Chile, public cyber-security advisories on smishing (2024–2025). <https://www.csirt.gob.cl>

---

## Citation

Moraga, P., & de la Paz, A. (2026). *From SMS to the operator panel: hunting a live fake-casino smishing campaign.* Threat intelligence case study, 8.8 Unreal Chile. Public edition withholds infrastructure identifiers while the campaign remains active.

---

*Document version 1.0 (2026-08-22). One file, written to be published as-is. If you mirror it, do not re-insert live domains, addresses, or passwords.*
