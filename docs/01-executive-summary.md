# 1. Executive summary

We tracked a Spanish-language **smishing** operation aimed at people in Latin America, with clear impact in Chile. The lure is not a bank, a tax agency, or a courier. It is a **fake online casino**: an SMS, a shortened link, a disposable website, then a durable back end that takes deposits and runs a live helpdesk.

The helpdesk is a fork of the open-source product **GOFLY** (`go-fly-customer`), on the same host as the casino API. Operators do not steal card data in the chats we reviewed. They keep the victim inside a payment loop: a fake “bank tax” (often about 10%), a fee to “correct” the withdrawal account, a small top-up “to verify” the account, and in-game “tasks.” In the sample, **nobody was paid out**.

The hunt had four beats:

1. **Entry.** The public login form advertised a decoy agent. That account did not work. The agent actually embedded in the casino site was the product’s default service account, protected by a trivial password.
2. **Lockout.** After that access had been used to record scale and chats, the operator rotated that password. The previous login failed. Parts of the host also timed out or returned gateway errors.
3. **The token was not the door.** Sessions are JWTs (HMAC-SHA256, about 24 hours). The signing secret is **not** the upstream GOFLY default, and it was **not** recovered. Forging tokens is not how access continued. Tokens already issued also stayed valid until they expired, even after the password change.
4. **The other admin.** The panel’s own admin API returns bcrypt password hashes to a super-admin. From a hash collected before the lockout, an offline guess recovered the second super-admin. Same privileges. The password was the same weak pattern with a cosmetic prefix, not a real rotation. Passwords are **not** printed in this repository.

On 19 August 2026 the panel showed on the order of **5,900 visitors** and **57,700 messages**. We read **25 conversations** (about 0.42% of visitors). Recorded top-ups in that sample add up to about **CLP 89.7 million**. No card numbers were seen. Victim phone numbers, national IDs, and emails are not in this report.

Next: [This is not bank smishing](02-not-bank-smishing.md).
