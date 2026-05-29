---
name: non-compete-check
description: >
  Validate a Finnish non-compete clause against TSL §3:5 — maximum duration,
  compensation requirements, scope reasonableness. Use when reviewing or
  drafting a non-compete, or asking "is this non-compete valid in Finland",
  "do we need to pay for non-compete", "how long can a non-compete be".
argument-hint: "[describe or paste the non-compete clause]"
---

# /fi-startup-legal:non-compete-check

1. Load profile.
2. Call `mcp__velvoite__get_finnish_statute("TSL", "3:5")` — fetch current non-compete statute text.
3. Validate against the live statute. Flag any issues.

---

## Validation (from live statute text)

Apply the fetched TSL §3:5 text to check:

### Duration
- Maximum: 12 months after employment ends
- If restriction > 12 months: **INVALID** under Finnish law — clause is unenforceable beyond 12 months

### Compensation requirement
- 0–6 months restriction: no compensation required
- 6–12 months restriction: compensation must be paid to the employee during the restricted period
  - Amount: extract from current statute text (historically 50% of salary — verify with live text)
  - Payment timing: typically monthly during the restriction period
- If compensation is required but not included in the contract: **RED — clause may be unenforceable**

### "Particularly weighty reason" requirement
Finnish law requires a "particularly weighty reason" (erityisen painava syy) for a non-compete to be valid. Legitimate reasons: access to trade secrets, customer relationships, sensitive competitive information. A blanket non-compete for all employees regardless of role is vulnerable.

Ask: does the employee have access to genuine trade secrets or key customer relationships? If no clear justification: **AMBER — validity risk**.

### Geographic scope
Unreasonably broad geographic scope (e.g. entire world for a role with no international dealings) is a validity risk. Finnish courts assess reasonableness.

### Scope of activities restricted
Must be related to the employee's actual duties and the employer's actual business. "Any employment in any competing company" without qualification is too broad.

## Output

| Issue | Status | Recommendation |
|---|---|---|
| Duration (max 12 months) | ✓/✗ | |
| Compensation (if >6 months) | ✓/✗ | |
| Weighty reason documented | ✓/✗ | |
| Scope reasonable | ✓/✗ | |

---

## Guardrail

Non-compete enforceability depends on facts and circumstances. Finnish courts have voided overly broad non-competes. This check identifies clear legal issues — full enforceability assessment requires legal advice.
