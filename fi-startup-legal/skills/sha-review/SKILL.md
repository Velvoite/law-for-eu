---
name: sha-review
description: >
  Review a Finnish shareholders' agreement (osakassopimus) against the seriesseed.fi
  market standard — three-tier leaver, pre-emption rights, drag-along, tag-along,
  ROFR, anti-dilution, board composition, protective provisions. Use when reviewing
  a SHA, asking "is our SHA standard", or preparing for a funding round.
argument-hint: "[paste or describe the SHA, or 'seriesseed-check' for a quick checklist]"
---

# /fi-startup-legal:sha-review

1. Load profile. If placeholders, stop.
2. Call `mcp__velvoite__get_finnish_statute("OYL", "3")` for share transfer and buyback rules.
3. Call `mcp__velvoite__get_finnish_statute("OYL", "9")` for shareholder rights.
4. Review against seriesseed.fi standard. Output: term-by-term RAG status.

---

## seriesseed.fi standard checklist

The Startup Foundation (startup-saatio.fi) seriesseed.fi templates are the Finnish market standard for seed rounds. Review each section:

### Leaver provisions
| Term | Market standard | Status |
|---|---|---|
| Good Leaver definition | Leaves voluntarily after cliff, or involuntary without cause | |
| Early Leaver definition | Leaves before cliff OR voluntarily before 2 years | |
| Bad Leaver definition | Termination for cause, material breach, non-compete violation | |
| Good Leaver share price | Fair market value | |
| Early/Bad Leaver share price | Subscription price (nominal) | |
| Vesting schedule | 4 years, 1-year cliff | |
| Acceleration | Single-trigger (involuntary termination post-M&A) market standard | |

### Transfer restrictions
| Term | Market standard | Notes |
|---|---|---|
| Pre-emption right | Existing shareholders have right to buy before external transfer | OYL §3 allows; must be in SHA or yhtiöjärjestys |
| ROFR (right of first refusal) | At same price as third-party offer | |
| Drag-along threshold | >2/3 of all shares (seriesseed.fi 4.0 standard) — 75% is above standard and more protective for founders; anything below 2/3 is aggressive | Must be in SHA; cannot be in yhtiöjärjestys alone |
| Tag-along | Minority can join any majority sale at same price | |
| Lock-up | Typically 12 months post-IPO if applicable | |

### Investor protections
| Term | Market standard | Notes |
|---|---|---|
| Anti-dilution | Broad-based weighted average (not full ratchet) | Full ratchet is aggressive — flag |
| Pro-rata right | Investor can maintain % in future rounds | |
| Information rights | Quarterly financials, annual audited accounts | |
| Board seat | Lead investor gets 1 board seat for rounds >€500k | |
| Protective provisions | New share issuance, M&A, budget approval require investor consent | List the standard matters |

**Cross-border governing law check:** If any investor is foreign, confirm the SHA specifies governing law explicitly. A SHA drafted under Finnish OYL assumptions with Swedish or UK governing law creates significant interpretation gaps. Flag for outside counsel if governing law is ambiguous.

### 2024 seriesseed.fi updates
The Startup Foundation updated templates in 2024 to add:
- KYC/AML representations from investors
- Sanctions screening obligations
- Three-tier leaver (replacing two-tier) — check if the SHA being reviewed uses three-tier or older two-tier format

---

## Guardrail

SHA review identifies deviations from market standard. Whether a deviation is acceptable depends on the specific investor relationship and negotiating dynamics. Attorney review required before signing.
