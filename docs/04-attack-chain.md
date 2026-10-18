# 4. Attack chain

```
Unsolicited SMS
  → shortened URL + affiliate tags
  → disposable casino site (object storage + CDN)
  → durable host: casino API + GOFLY helpdesk
  → chat script: tax, “fix account”, tasks
  → further deposits, no withdrawal in the sample
```

The site visitors see can be replaced in days. The business sits on the gate: API and chat on one host. Taking down only the front page does not stop the operators.

| Stage | What we observed |
|-------|------------------|
| Delivery | Unsolicited SMS, shortened link |
| Landing | Vue SPA, white-label gaming kit; old gates left in comments; Open Graph tags still point at a previous front |
| Engagement | Embedded helpdesk; the widget calls the default service agent, not the decoy on the login form |
| Monetization | In-app deposits plus chat pressure |
| Operator OPSEC | Weak passwords, hash leakage to admins, debug endpoint, incomplete session revocation |

A local payment-gateway checkout was seen in related work. **No technical link** was proven between that checkout and this landing or gate. Names and checkout identifiers are withheld.

Next: [What victims said](05-victim-narratives.md).
