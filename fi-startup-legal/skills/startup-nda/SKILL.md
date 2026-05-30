---
name: startup-nda
description: >
  NDA review and generation for Finnish startups — mutual vs one-way,
  EU Trade Secrets Directive (Liikesalaisuuslaki 595/2018) framing, duration,
  scope. Use when reviewing or drafting an NDA, asking "is this NDA okay",
  "NDA with investor", "NDA with employee".
argument-hint: "[paste NDA or describe: mutual/one-way, with whom, purpose]"
---

# /fi-startup-legal:startup-nda

1. Load profile.
2. Call `mcp__velvoite__get_finnish_statute("LSL", "2")` — trade secret definition under Finnish law.
3. Review or generate NDA. Output is draft — attorney review for high-stakes NDAs.

---

## Finnish NDA framework

The Business Secrets Act (Liikesalaisuuslaki 595/2018) implements the EU Trade Secrets Directive in Finland. Key definition from live statute:

A trade secret is information that:
1. Is secret (not generally known or readily accessible)
2. Has commercial value because it is secret
3. Has been subject to reasonable steps to keep it secret

### Review checklist

| Term | Finnish market standard | Status |
|---|---|---|
| Definition of confidential information | Broad but excluding: public domain, prior knowledge, independent development, required disclosure | |
| Mutual vs one-way | Mutual for most startup contexts; one-way acceptable for investor NDA | |
| Duration of obligation | 2–5 years standard; perpetual for true trade secrets (acceptable) | |
| Return/destruction | On request or termination | |
| Governing law | Finnish law | |
| Dispute resolution | Finnish courts or arbitration | |
| Remedies | §8 (injunction/prohibition) + §11 (damages and unjust enrichment) | |

### Investor NDA note
Finnish VCs and angels typically do NOT sign NDAs before a first meeting — standard practice. If an investor insists on signing before seeing your deck, it may signal inexperience. Consider sharing your deck without requiring an NDA for early-stage fundraising conversations.

### Employee NDA
Employment contracts should include confidentiality obligations. A separate NDA is typically redundant for employees — the employment contract + TSL Chapter 3 confidentiality provisions cover it.

---

## Guardrail

NDAs are enforceable contracts. For high-value situations (sharing trade secrets with a strategic partner, M&A diligence), have a lawyer review or draft.
