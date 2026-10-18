# 3. The hunt

Dates are from the observation window in August 2026. Infrastructure names are omitted on purpose.

## 3.1 The front gives away the kit

The website is a Vue single-page application served from Amazon S3 and CloudFront. It is built to be thrown away. The client itself points the lobby at one durable host: casino API plus helpdesk. Affiliate tags on the link (`tr`, `r`) show the SMS layer is a distribution network, not a single text.

The page also leaks history without any login:

- JavaScript comments still list **older API gates**, including one raw IP, with the current gate left uncommented.
- Open Graph tags still advertise a **previous** front (old domain, shared image path). The brand on the glass changes; the mould does not.

A Telegram bot is referenced in the same bundle. A passive look at the public profile showed an empty contact card, not a live “Start bot” page. The handle is withheld.

## 3.2 The decoy on the login form

The helpdesk is GOFLY, Internet-facing, including the upstream **install README** at the site root. The login page is pre-filled with an agent name and a default-looking password. That pair **failed**. It is bait, or a leftover from the stock template.

The casino embed does not call that decoy. It calls the product’s real default agent: the service account. That password was trivial and **worked**. The API expects the session in an HTTP header named `token` (a JWT), not `Authorization: Bearer`. The role on that token was super-admin.

## 3.3 What the first session showed

With that session we recorded, without taking a full dump:

- Panel statistics: about 5,900 visitors and 57,700 messages.
- A page of visitor records (500), then full threads for 25 numeric user ids.
- The agent list. Two super-admins. The API returned **bcrypt password hashes** for both. Those hashes are not published.
- No MySQL password. The settings screen exists; the API route that used to return the database password responds **404** on this fork.

## 3.4 The operator cut the key

Later the same day the service account stopped accepting the old password. The host was also unstable (timeouts and HTTP 502). That was a lockout of the password, not of every session: JWTs already issued **kept working until expiry**, on the order of 24 hours. Changing the password did not revoke tokens.

## 3.5 The JWT did not open

We tested whether the signature used the well-known GOFLY development secret, other short defaults, and common wordlists (HMAC-SHA256, including hashcat mode 16500, and an independent HMAC check). **No secret was found.** Unauthenticated heap profiles (`/debug/pprof/`, text mode) show call stacks, not a readable signing key. The second door was not a forged admin token.

## 3.6 The other super-admin

The second account was created later than the stock service user and had the same super-admin role. Its bcrypt hash was already in the earlier export. An offline dictionary guess matched a password that is the **same weak numeric pattern** as the first account, with a short prefix. That is a rotation in appearance only.

A normal login with that account issued a **new** JWT. It saw the same panel, the same visitor counts, and the same hash-export API. The service account’s **new** hash (after the operator’s change) also matched, offline, another weak variant of the same pattern. Those secrets are not written here.

The finding is the pattern: both super-admins, before and after the lockout, were guessable, and the application helped by returning hashes to anyone who was already an admin.

## 3.7 The debug endpoint

`/debug/pprof/` answered with **no token**. That is a deployment mistake (Go’s profiler registered without the JWT middleware), not a CVE in GOFLY. It disclosed the binary family and a developer build path (a macOS developer workstation); the live host is likely on AWS (inferred, not confirmed), consistent with the front’s AWS hosting. Those filesystem strings are withheld. It did **not** hand over the JWT secret or a database URL.

Heap dumps large enough to matter for availability were retrieved. Text-mode heap profiles are stacks, not raw secrets. Assigning another product’s pprof CVE to this host would be wrong. The **class** of bug is the same: a profiler on a public port.

Next: [Attack chain](04-attack-chain.md).
