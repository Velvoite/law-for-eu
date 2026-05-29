# eu-legal Sub-Config Structure + enforcement-intelligence Skill

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add practice-area sub-config files so each plugin area (privacy, commercial, employment, corporate) can be configured independently, and add the `enforcement-intelligence` skill that uses Velvoite enforcement decisions + KHO/German case law to produce a "fix this first" action list.

**Architecture:** Four sub-config templates ship with the plugin alongside `CLAUDE.md`. The cold-start-interview is updated to explain the pattern. Each practice-area cold-start (Plans 2–5) writes to its own file. Skills load base profile first, then their practice file. The `enforcement-intelligence` skill adds a 7th regulatory skill calling `get_enforcement_decisions`, `get_enforcement_intelligence`, and `search_regulations` for case law.

**Tech stack:** Same as Plan 1 — SKILL.md with YAML frontmatter + markdown body, plugin template files.

---

## Files created or modified

```
eu-legal/
  privacy.md              ← new template (ships with plugin)
  commercial.md           ← new template
  employment.md           ← new template
  corporate.md            ← new template
  skills/
    cold-start-interview/
      SKILL.md            ← modified (add sub-config pattern explanation)
    enforcement-intelligence/
      SKILL.md            ← new skill
```

Config path for sub-configs (written by practice-area cold-starts, Plans 2–5):
```
~/.claude/plugins/config/eu-legal/
  CLAUDE.md               ← base profile (written by cold-start-interview)
  privacy.md              ← written by /eu-legal:privacy-cold-start (Plan 2)
  commercial.md           ← written by /eu-legal:commercial-cold-start (Plan 3)
  employment.md           ← written by /eu-legal:employment-cold-start (Plan 4)
  corporate.md            ← written by /eu-legal:corporate-cold-start (Plan 5)
```

---

## Task 1: Four practice-area template files

**Files:**
- Create: `eu-legal/privacy.md`
- Create: `eu-legal/commercial.md`
- Create: `eu-legal/employment.md`
- Create: `eu-legal/corporate.md`

- [ ] **Step 1: Write `eu-legal/privacy.md`**

```markdown
<!--
Practice-area config for privacy skills.
User config lives at: ~/.claude/plugins/config/eu-legal/privacy.md
This file is a template. Run /eu-legal:privacy-cold-start to populate it.
-->

# Privacy Practice Config
*Run `/eu-legal:privacy-cold-start` to set up. Until then skills run with base profile only.*

---

## Role and scope

**Primary role:** [PLACEHOLDER — controller | processor | both]
**Personal data categories processed:** [PLACEHOLDER — e.g. employee data, customer financial data, transaction data]
**DPO:** [PLACEHOLDER — name / not required / not appointed]
**Lead supervisory authority:** [PLACEHOLDER — FIN-FSA / BaFin / ICO / CNIL / DPC / other]

---

## DPA playbook

### When we are the processor

| Term | Our standard | Acceptable | Never |
|---|---|---|---|
| Audit rights | [PLACEHOLDER] | | |
| Breach notification window | [PLACEHOLDER — e.g. 24h to controller] | | |
| Subprocessor changes | [PLACEHOLDER — e.g. 30 days notice] | | |
| Data location | [PLACEHOLDER — EU only / EU + approved third countries] | | |
| Deletion on termination | [PLACEHOLDER — e.g. 30 days] | | |

### When we are the controller

| Term | We require | Acceptable | Never accept |
|---|---|---|---|
| Audit rights | [PLACEHOLDER] | | |
| Breach notification window | [PLACEHOLDER — e.g. 72h per GDPR Art. 33] | | |
| Subprocessor restrictions | [PLACEHOLDER] | | |

---

## DSAR process

**Verification method:** [PLACEHOLDER — e.g. ID document + account email match]
**Systems list:** [PLACEHOLDER — list every system where personal data lives]
**Response SLA:** GDPR: 1 month (extendable to 3 months for complex requests)
**Who handles routine requests:** [PLACEHOLDER]
**Escalation trigger:** [PLACEHOLDER — e.g. requests involving sensitive data, litigation hold]

---

## DPIA triggers (beyond GDPR Art. 35 mandatory list)

[PLACEHOLDER — any internal triggers beyond the mandatory list, e.g. new vendor with access to >10k records]

---

## AI Act posture

**High-risk AI systems in use:** [PLACEHOLDER — list any Annex III systems: credit scoring, fraud detection, employment decisions, etc.]
**FRIA required for:** [PLACEHOLDER — which systems require a Fundamental Rights Impact Assessment]
**Joint DPIA+FRIA approach:** [PLACEHOLDER — yes / no / per-system]
```

- [ ] **Step 2: Write `eu-legal/commercial.md`**

```markdown
<!--
Practice-area config for commercial contract skills.
User config lives at: ~/.claude/plugins/config/eu-legal/commercial.md
This file is a template. Run /eu-legal:commercial-cold-start to populate it.
-->

# Commercial Practice Config
*Run `/eu-legal:commercial-cold-start` to set up. Until then skills run with base profile only.*

---

## Standard positions

**Preferred governing law:** [PLACEHOLDER — Finnish law | German law | English law | other]
**Preferred dispute resolution:** [PLACEHOLDER — Finnish courts / ICC arbitration / SCC arbitration / LCIA / other]
**Seat of arbitration (if applicable):** [PLACEHOLDER — Helsinki / Stockholm / London / other]
**Standard limitation of liability:** [PLACEHOLDER — e.g. 12 months fees / unlimited for data breaches]
**Consequential loss exclusion:** [PLACEHOLDER — yes (standard) / carve-outs for: ...]

---

## Sales-side playbook (outbound contracts — we are the vendor)

| Term | Our standard | We'll accept | Never |
|---|---|---|---|
| Liability cap | [PLACEHOLDER] | | |
| IP ownership | [PLACEHOLDER — we retain all IP] | | |
| Data processing | [PLACEHOLDER] | | |
| Termination for convenience | [PLACEHOLDER — e.g. 90 days notice] | | |
| Audit rights | [PLACEHOLDER] | | |

---

## Purchasing-side playbook (inbound contracts — we are the customer)

| Term | We require | Acceptable | Never accept |
|---|---|---|---|
| Liability cap | [PLACEHOLDER] | | |
| SLA / uptime | [PLACEHOLDER] | | |
| Data location | [PLACEHOLDER — EU only] | | |
| DORA ICT clause | [PLACEHOLDER — required for ICT third-party providers] | | |
| Exit assistance | [PLACEHOLDER] | | |

---

## DORA ICT third-party requirements

**ICT third-party providers identified:** [PLACEHOLDER — list critical ICT providers]
**DORA Art. 30 contractual clauses status:** [PLACEHOLDER — in all contracts / partial / pending]
**Concentration risk assessment done:** [PLACEHOLDER — yes / no / date]

---

## Escalation thresholds

| Trigger | Escalate to |
|---|---|
| Contract value > [PLACEHOLDER] | [PLACEHOLDER — GC / CFO / Board] |
| Liability cap > [PLACEHOLDER] | [PLACEHOLDER] |
| Non-standard IP clause | [PLACEHOLDER] |
| Data transfer outside EU | [PLACEHOLDER] |
```

- [ ] **Step 3: Write `eu-legal/employment.md`**

```markdown
<!--
Practice-area config for employment skills.
User config lives at: ~/.claude/plugins/config/eu-legal/employment.md
This file is a template. Run /eu-legal:employment-cold-start to populate it.
-->

# Employment Practice Config
*Run `/eu-legal:employment-cold-start` to set up. Until then skills run with base profile only.*

---

## Workforce footprint

**Hiring countries:** [PLACEHOLDER — e.g. Finland, Germany, Estonia]
**Total headcount (approx):** [PLACEHOLDER]
**Employment contract type used:** [PLACEHOLDER — indefinite / fixed-term / both]
**Collective agreement (TES/Tarifvertrag):** [PLACEHOLDER — which agreement(s) apply, or none]

---

## Finland-specific (YT-laki / Employment Contracts Act)

**Works council / personnel representation:** [PLACEHOLDER — required (>20 employees) / not required / name of body]
**YT-laki cooperation obligation threshold reached:** [PLACEHOLDER — yes (>20 employees) / no]
**Annual holidays (Vuosilomalaki) entitlement used:** [PLACEHOLDER — statutory minimum / enhanced]
**Notice period standard (employment contracts):** [PLACEHOLDER — e.g. statutory / contractual: X months]

---

## Germany-specific (BetrVG / KSchG / TzBfG)

**Betriebsrat (works council) exists:** [PLACEHOLDER — yes / no / not applicable]
**KSchG dismissal protection applies:** [PLACEHOLDER — yes (>10 employees) / no]
**Standard notice period:** [PLACEHOLDER — statutory per KSchG §622 / contractual]

---

## Termination policy

**Standard severance offer (if any):** [PLACEHOLDER — none / X months per year of service]
**Redundancy process:** [PLACEHOLDER — individual / collective (YT-neuvottelut / Massenentlassung)]
**Escalation trigger:** [PLACEHOLDER — e.g. any termination involving protected class / works council consultation required]
**Outside counsel for terminations:** [PLACEHOLDER — always / threshold: X]

---

## Standard employment contract terms

**Non-compete clause:** [PLACEHOLDER — yes (max 12 months under Finnish law) / no]
**Confidentiality period post-employment:** [PLACEHOLDER]
**IP assignment:** [PLACEHOLDER — work-for-hire clause / assignment clause]

---

## KHO case law notes

[PLACEHOLDER — any notable Finnish Supreme Administrative Court decisions relevant to your employment practice — populated by /eu-legal:employment-cold-start or manually]
```

- [ ] **Step 4: Write `eu-legal/corporate.md`**

```markdown
<!--
Practice-area config for corporate governance skills.
User config lives at: ~/.claude/plugins/config/eu-legal/corporate.md
This file is a template. Run /eu-legal:corporate-cold-start to populate it.
-->

# Corporate Practice Config
*Run `/eu-legal:corporate-cold-start` to set up. Until then skills run with base profile only.*

---

## Entity register

| Entity name | Type | Jurisdiction | Business ID | Registered agent |
|---|---|---|---|---|
| [PLACEHOLDER] | [OY / AB / GmbH / SAS / BV / other] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

---

## Finland-specific (OY)

**PRH filing obligations:** Annual financial statements (4 months after FY end), trade register notifications for changes
**Auditor required:** [PLACEHOLDER — yes (exceeds 2 of 3 thresholds) / no / voluntary]
**Auditor name:** [PLACEHOLDER]
**Financial year end:** [PLACEHOLDER — e.g. 31 Dec]
**Next PRH annual report due:** [PLACEHOLDER]

---

## Germany-specific (GmbH / AG)

**Handelsregister filing:** [PLACEHOLDER — annual financial statements, changes to Geschäftsführer/Vorstand]
**Co-determination (Mitbestimmung):** [PLACEHOLDER — required (>500 employees) / not required]
**Supervisory board (Aufsichtsrat):** [PLACEHOLDER — required / not required]

---

## Board structure

**Board type:** [PLACEHOLDER — one-tier / two-tier]
**Board size:** [PLACEHOLDER]
**Board meeting cadence:** [PLACEHOLDER — e.g. quarterly]
**Resolution threshold:** [PLACEHOLDER — simple majority / supermajority for: ...]
**Written resolution permitted:** [PLACEHOLDER — yes / no (jurisdiction-specific)]

---

## M&A and transactions

**EU merger control threshold monitoring:** Worldwide turnover > €5B AND EU-wide > €250M → mandatory EC notification
**Current M&A activity:** [PLACEHOLDER]
**VDR platform used:** [PLACEHOLDER — Datasite / Intralinks / SharePoint / other]

---

## Shareholder structure

[PLACEHOLDER — major shareholders, any shareholders' agreement in place, drag-along/tag-along provisions]
```

- [ ] **Step 5: Verify all four files exist**

```bash
ls eu-legal/privacy.md eu-legal/commercial.md eu-legal/employment.md eu-legal/corporate.md
```

- [ ] **Step 6: Commit**

```bash
git add eu-legal/privacy.md eu-legal/commercial.md eu-legal/employment.md eu-legal/corporate.md
git commit -m "feat(eu-legal): add practice-area sub-config templates"
```

---

## Task 2: Update cold-start-interview to explain sub-config pattern

**Files:**
- Modify: `eu-legal/skills/cold-start-interview/SKILL.md`

Add a new section after the "Write the config" section that explains the sub-config pattern to users:

- [ ] **Step 1: Read the current end of the file**

Read `eu-legal/skills/cold-start-interview/SKILL.md` and locate the "Write the config" section near the end.

- [ ] **Step 2: Add sub-config explanation after the "Write the config" section**

Append this section before the final offer block (the "Setup complete. Try:" offer):

```markdown
---

## Practice-area configs (optional deeper setup)

The base profile captures entity type, jurisdiction, and regulatory posture. For each practice area, a separate config file enables more detailed, practice-specific behaviour:

| Practice area | Config path | How to populate |
|---|---|---|
| Privacy / GDPR + AI Act | `~/.claude/plugins/config/eu-legal/privacy.md` | `/eu-legal:privacy-cold-start` (Plan 2) |
| Commercial contracts | `~/.claude/plugins/config/eu-legal/commercial.md` | `/eu-legal:commercial-cold-start` (Plan 3) |
| Employment | `~/.claude/plugins/config/eu-legal/employment.md` | `/eu-legal:employment-cold-start` (Plan 4) |
| Corporate governance | `~/.claude/plugins/config/eu-legal/corporate.md` | `/eu-legal:corporate-cold-start` (Plan 5) |

Skills work without these files — they fall back to base profile. The practice-area configs unlock detailed behaviour: DPA playbook positions, contract standard terms, works council thresholds, board structure, Finnish KHO case law notes.
```

- [ ] **Step 3: Validate**

```bash
claude plugin validate ./eu-legal
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/cold-start-interview/SKILL.md
git commit -m "feat(eu-legal): document sub-config pattern in cold-start-interview"
```

---

## Task 3: enforcement-intelligence skill

**Files:**
- Create: `eu-legal/skills/enforcement-intelligence/SKILL.md`

This is the 7th regulatory skill. It uses enforcement decisions + KHO/German case law to produce a "fix this first" action list — the key differentiation from generic compliance tools.

- [ ] **Step 1: Create directory and write `eu-legal/skills/enforcement-intelligence/SKILL.md`**

```markdown
---
name: enforcement-intelligence
description: >
  What do regulators actually fine companies for — and what does case law say
  about it? Uses Velvoite enforcement decisions plus Finnish KHO and German
  court case law to produce a "fix this first" action list for your entity type.
  Use when asking "what are we most likely to get fined for", "enforcement
  trends", "what did BaFin fine payment institutions for", "KHO precedents on
  DORA", "top compliance risks", or similar.
argument-hint: "[regulation] [jurisdiction] — e.g. 'dora' or 'gdpr fi'"
---

# /eu-legal:enforcement-intelligence

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Check Velvoite: if `VELVOITE_API_KEY` not set, run Fallback.
3. Run the workflow below.
4. Output is for attorney review and compliance planning — not a legal opinion.

---

## Purpose

Most compliance tools tell you what the rules say. This skill tells you what regulators actually enforce. The difference: the rules are aspirational; enforcement decisions are evidence of what authorities prioritise, what they consider a material breach, and how much they fine for it. Combined with court case law (KHO for Finland, administrative courts for Germany), you get a defensible compliance posture grounded in real decisions, not just text.

## Parse args

- `regulation` (optional): focus on one regulation — `dora`, `gdpr`, `mica`, `ai_act`, `aml`, `mifid2`, etc. Default: all regulations relevant to entity type.
- `jurisdiction` (optional): `fi` (Finland — includes KHO case law), `de` (Germany — includes German court decisions), `eu` (EU-wide, no national case law layer). Default: primary jurisdiction from profile.

## Read from profile

- Entity type and actor roles → scope enforcement queries
- Primary jurisdiction → determines which case law layer to use

## Workflow

### Step 1: Enforcement landscape

Call `mcp__velvoite__get_enforcement_decisions` with entity type from profile and regulation filter if provided. Sort by fine amount descending.

Then call `mcp__velvoite__get_enforcement_intelligence` to get the top violated obligations summary.

Show:
**Enforcement summary for [entity type] — [regulation or all]**

| Rank | Violation area | Cases | Largest fine | Authority |
|---|---|---|---|---|

Cap at top 5 violation areas to keep output actionable.

### Step 2: Deep-dive top 3 violations

For each of the top 3 violation areas:

1. **What happened** — summarise 1–2 representative enforcement decisions: what the company did (or failed to do), what the authority found, what the fine was.

2. **What the authority expected** — extract the positive obligation from the decision: "The authority found that [entity type] must maintain X, test it Y-frequently, and document Z."

3. **Case law layer** — if jurisdiction is `fi` or `de`, call `mcp__velvoite__search_regulations` with a query combining the violation topic + jurisdiction (e.g. "ICT incident reporting DORA Finland KHO" or "Meldepflicht BaFin Zahlungsinstitut"). Surface any KHO or German administrative court decisions that have interpreted this obligation. If found, note: "KHO [case number / year]: [one-line holding]."

4. **What to check in your organisation** — 3 specific questions to ask your compliance team about this violation area.

### Step 3: Fix-this-first action list

Synthesise a prioritised action list:

**Top 5 actions to reduce enforcement risk — [entity type] / [regulation(s)]**

| Priority | Action | Based on | Urgency |
|---|---|---|---|
| 1 | [Specific action] | [Authority / case] | [High / Medium] |
| ... | | | |

Urgency is High if: fine > €100k, multiple enforcement actions for same violation, or obligation has a near-term deadline in the compliance calendar.

### Step 4: Cross-reference with obligations

Note: "Run `/eu-legal:obligations [regulation]` to see the full obligation list. Run `/eu-legal:reg-gap-analysis [regulation]` to assess your current compliance posture against these risk areas."

---

## Fallback (VELVOITE_API_KEY not set)

Enforcement intelligence requires the Velvoite corpus — enforcement decisions and case law are not available from public text alone.

Add `VELVOITE_API_KEY` to your `.envrc` and run `direnv allow`. Free 30-day trial at velvoite.eu.

In the meantime, the EU enforcement landscape by regulation:
- **GDPR**: largest fines from CNIL (France), DPC (Ireland), BaFin. Top violation areas: consent, data transfers, breach notification, data minimisation.
- **DORA**: enforcement begins in 2025. EBA, ESMA, EIOPA as Joint Oversight. Key focus: ICT incident reporting, third-party risk, resilience testing.
- **AML/CFT**: FIN-FSA has levied fines on Finnish payment institutions for inadequate transaction monitoring, CDD failures.
- **MiFID II**: ESMA and national NCAs. Top areas: best execution, transaction reporting, client onboarding.

---

## Guardrail

Enforcement decisions describe past regulatory action in specific factual contexts. They are not a definitive statement of legal obligation for your organisation. Case law citations are from the Velvoite corpus and should be verified at source before reliance. This output is a working document for attorney review — not a legal opinion.
```

- [ ] **Step 2: Validate**

```bash
claude plugin validate ./eu-legal
```

Expected: passes. Skill count should now be 7.

```bash
find eu-legal/skills -name 'SKILL.md' | wc -l
```

Expected: `7`

- [ ] **Step 3: Commit**

```bash
git add eu-legal/skills/enforcement-intelligence/
git commit -m "feat(eu-legal): add enforcement-intelligence skill with KHO/German case law layer"
```

---

## Task 4: Final validation

- [ ] **Step 1: Full sweep**

```bash
claude plugin validate ./eu-legal
python3 -c "import json,glob; [json.load(open(f)) for f in glob.glob('eu-legal/**/*.json',recursive=True)] and print('JSON OK')"
find eu-legal/skills -name 'SKILL.md' | wc -l
```

Expected: validation passes, JSON OK, skill count = 7.

- [ ] **Step 2: Confirm sub-config templates ship with plugin**

```bash
ls eu-legal/*.md
```

Expected: `CLAUDE.md  commercial.md  corporate.md  employment.md  privacy.md`

- [ ] **Step 3: Final commit if anything unstaged**

```bash
git -C /Users/mattipuhakka/law-for-eu status
```

Only commit if there are unstaged changes.
