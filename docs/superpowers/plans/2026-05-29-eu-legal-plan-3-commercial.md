# eu-legal Plugin — Plan 3: Commercial Skills

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 7 EU-first commercial contract skills to the eu-legal plugin, covering setup, contract and NDA review, vendor procurement with DORA ICT checks, escalation routing, playbook management, and renewal tracking — all scoped to EU governing law and EUR thresholds.

**Architecture:** Each skill is a flat SKILL.md under `eu-legal/skills/<name>/SKILL.md`. All commercial skills read a shared sub-config at `~/.claude/plugins/config/eu-legal/commercial.md` (written by `commercial-cold-start`); they gracefully prompt the user to run setup if this file is missing. Where the user is a financial entity and `VELVOITE_API_KEY` is set, `contract-review` and `vendor-agreement-review` call `mcp__velvoite__get_canonical_obligations` to surface DORA Art. 30 ICT third-party requirements; without the key both skills degrade to documented general guidance. The renewal register is a separate YAML at `~/.claude/plugins/config/eu-legal/renewal-register.yaml`, not inside commercial.md, so it survives commercial-cold-start re-runs.

**Tech stack:** Claude plugin format (SKILL.md with YAML frontmatter + markdown body).

**This plan covers:** `commercial-cold-start`, `contract-review`, `nda-review`, `vendor-agreement-review`, `escalation-flagger`, `playbook-init`, `renewal-tracker`.

---

## Files created in this plan

```
eu-legal/skills/
  commercial-cold-start/SKILL.md
  contract-review/SKILL.md
  nda-review/SKILL.md
  vendor-agreement-review/SKILL.md
  escalation-flagger/SKILL.md
  playbook-init/SKILL.md
  renewal-tracker/SKILL.md
```

Config paths (written at runtime, not created by this plan):
- `~/.claude/plugins/config/eu-legal/commercial.md` — written by `commercial-cold-start` or `playbook-init`
- `~/.claude/plugins/config/eu-legal/renewal-register.yaml` — written by `renewal-tracker` (Mode 1 ingest)

---

## Shared rules for all commercial skills

Apply these to every SKILL.md body in this plan. They are not repeated in full in each task, but each skill must implement them.

1. **Config load order:** Read `~/.claude/plugins/config/eu-legal/CLAUDE.md` first (base profile: entity type, jurisdiction, Velvoite posture). Then read `~/.claude/plugins/config/eu-legal/commercial.md` (commercial playbook). If `commercial.md` is missing or has `[PLACEHOLDER]` values, stop and prompt: "Run `/eu-legal:commercial-cold-start` first — I need your commercial playbook before I can proceed."
2. **EUR only.** All financial thresholds, liability caps, and escalation amounts are in EUR.
3. **No US defaults.** No references to Delaware, New York, California, AAA, JAMS, or US-only regulatory frameworks. EU governing law defaults only.
4. **DORA ICT checks are gated.** Only perform Velvoite MCP calls if `VELVOITE_API_KEY` is set AND the entity type from `CLAUDE.md` is a financial entity (`credit_institution`, `payment_institution`, `e_money_institution`, `investment_firm`, `aifm`, `ucits`, `insurance`, `reinsurance`). If the API key is absent, include a static DORA Art. 30 checklist instead.
5. **Guardrail on all review outputs:** "This analysis is a draft for attorney review and does not constitute legal advice."
6. **MCP tool prefix:** `mcp__velvoite__`

---

## Task 1: `commercial-cold-start` skill

**File:** `eu-legal/skills/commercial-cold-start/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/commercial-cold-start
```

- [ ] **Step 2: Write `eu-legal/skills/commercial-cold-start/SKILL.md`**

```markdown
---
name: commercial-cold-start
description: >
  Set up the commercial contracts practice for eu-legal — learns your governing
  law preferences, dispute resolution, liability positions, sales-side and
  purchasing-side playbooks, DORA ICT provider status, and escalation thresholds.
  Writes ~/.claude/plugins/config/eu-legal/commercial.md. Run on first use or
  with --redo to refresh. Use when setting up commercial contracts, updating
  playbook positions, or before running contract-review, nda-review, or
  vendor-agreement-review for the first time.
argument-hint: "[--redo] [--check-integrations]"
---

# /eu-legal:commercial-cold-start

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:cold-start-interview` first — I need your base profile before setting up commercial contracts."
2. If `commercial.md` exists and is populated (no `[PLACEHOLDER]` values) and `--redo` is not passed: confirm before overwriting. "Your commercial playbook is already configured. Passing `--redo` will overwrite it. Do you want to continue?"
3. Run the interview workflow below.
4. Write `~/.claude/plugins/config/eu-legal/commercial.md`. Show summary. Offer first task: "Run `/eu-legal:contract-review` to review a contract, or `/eu-legal:nda-review` for an NDA."

---

## --redo

Force a full re-run of the interview and overwrite `commercial.md`. Use when the company's legal posture, governing law preference, or liability positions change.

---

## Interview workflow

Ask questions conversationally — one topic per message. Never dump all questions at once.

### 1. Governing law preference

Ask:
> "Which governing law does your organisation prefer for commercial contracts?"

Present as a numbered list:
1. Finnish law (Finland — Courts of Helsinki)
2. German law (Germany — choice of city)
3. English law (England & Wales)
4. Swedish law (Sweden)
5. Dutch law (Netherlands)
6. French law (France)
7. Other EU member state (specify)

Save the machine-readable code:
| Selection | Code |
|---|---|
| 1 | `finnish_law` |
| 2 | `german_law` |
| 3 | `english_law` |
| 4 | `swedish_law` |
| 5 | `dutch_law` |
| 6 | `french_law` |
| 7 | `other_eu` |

### 2. Dispute resolution

Ask:
> "What's your preferred dispute resolution mechanism for cross-border contracts?"

1. Finnish courts (District Court of Helsinki / Helsinki Court of Appeal)
2. ICC arbitration (seat: Paris; or specify another seat)
3. SCC arbitration (Stockholm Chamber of Commerce — seat: Stockholm)
4. LCIA arbitration (London Court of International Arbitration — seat: London)
5. German courts (choice of city)
6. Netherlands Arbitration Institute (NAI)
7. Other (specify)

Note: Finnish courts for contracts where both parties are Finnish; ICC/SCC for cross-border EU contracts. LCIA is recommended only for English-law agreements.

Save: `dispute_resolution_preferred`, `dispute_resolution_acceptable`, `dispute_resolution_never`.

### 3. Standard liability positions

Ask:
> "Let's set your liability positions. What's your standard direct liability cap — typically expressed as a multiple of fees paid in the preceding 12 months?"

- Standard: "12 months of fees paid in the 12 months preceding the claim"
- Acceptable fallbacks (ask): higher caps for data breach / IP infringement?
- Consequential damages: excluded (standard)? Ask for carve-outs (data breach, IP, gross negligence, fraud).
- Never accept: uncapped indirect liability.

Save: `liability_direct_cap`, `liability_indirect`, `liability_carveouts`, `liability_never`.

### 4. Sales-side playbook

Ask:
> "You're selling. Walk me through your standard positions. Let's start with IP ownership — does IP created under a services agreement stay with you (licensor model) or transfer to the customer (work-for-hire)?"

Cover:
- IP ownership and background IP licence
- Audit rights (what you grant customers)
- Data processing: do you process customer personal data? (If yes: reference your DPA template)
- Termination: standard notice period, termination for cause vs. convenience
- The one deal-breaker (what you will never accept from a customer)

Save as `sales_side_playbook` with sub-fields for each position.

### 5. Purchasing-side playbook

Ask:
> "You're buying. What do you require from vendors? Let's start: do you require vendors to sign YOUR DPA as processor, or will you accept their DPA template with redlines?"

Cover:
- DPA requirement (your paper / their paper with redlines / either)
- Audit rights you require from vendors
- Subcontracting: do you require approval rights over subcontractors?
- SLA minimum: what uptime / response time do you require?
- Never accept from a vendor (e.g., vendor retains rights to use your data for training, uncapped liability waiver)

Save as `purchasing_side_playbook`.

### 6. DORA ICT provider status

Ask:
> "Has your organisation identified its critical ICT third-party providers under DORA Article 28? (Relevant if you are a financial entity.)"

Read entity type from `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If entity type is `other`, note that DORA ICT provider obligations may not apply directly but record the answer for context.

Options:
1. Yes — we maintain an ICT third-party register
2. Partially — we have an informal list but no formal register
3. No — we have not yet done this exercise
4. Not applicable — not a DORA-in-scope entity

Save: `dora_ict_register_status` (one of `formal_register` / `informal_list` / `not_done` / `not_applicable`).

If the answer is 1 or 2, ask:
> "You can list your critical ICT third-party providers here so the vendor-agreement-review skill can flag concentration risk. Do you want to add any now? (You can always add more later.)"

Save as `critical_ict_providers` (YAML list of vendor names).

### 7. Escalation thresholds

Ask:
> "Let's set your escalation matrix. Who can approve a contract, and at what threshold?"

Cover three levels (or fewer if a smaller team):
- Junior / paralegal / contract manager: threshold in EUR, what they can approve without escalation
- Lawyer / in-house counsel: threshold in EUR
- GC / head of legal: threshold in EUR
- Board / CFO: above GC threshold

Ask separately: automatic escalation triggers (regardless of EUR value):
- Any contract with governing law outside the EU?
- Any contract with a vendor that processes personal data outside the EU?
- Any ICT third-party agreement for a DORA-in-scope entity?
- Any contract with uncapped liability or IP assignment to the vendor?
- Liability cap below EUR [ask for threshold]?

Save: `escalation_matrix` (table), `escalation_auto_triggers` (list).

---

## Write commercial.md

After completing the interview, write `~/.claude/plugins/config/eu-legal/commercial.md`:

```markdown
# Commercial Contracts Playbook
*Written by /eu-legal:commercial-cold-start on [date]. Re-run with `--redo` to update.*

---

## Governing law and dispute resolution

**Preferred governing law:** [governing_law_preferred]
**Acceptable:** [governing_law_acceptable]
**Never:** [governing_law_never]

**Preferred dispute resolution:** [dispute_resolution_preferred]
**Acceptable:** [dispute_resolution_acceptable]
**Never:** [dispute_resolution_never]

---

## Liability positions

**Direct liability cap:** [liability_direct_cap — e.g., "12 months of fees paid in the 12 months preceding the claim"]
**Indirect / consequential damages:** [liability_indirect — e.g., "excluded, with carve-outs for data breach, IP infringement, and gross negligence"]
**Carve-outs above cap:** [liability_carveouts]
**Never accept:** [liability_never]

---

## Sales-side playbook

*Applies when we are the vendor / service provider. Usually our paper.*

**IP ownership:** [ip_ownership]
**Background IP licence:** [background_ip]
**Audit rights granted to customers:** [audit_rights_sales]
**Data processing:** [data_processing_sales]
**Termination notice period:** [termination_notice]
**Termination for convenience:** [termination_convenience]

**The one deal-breaker (sales-side):** [sales_deal_breaker]

**NDA triage positions:**
[Populated from interview — what makes an NDA GREEN, YELLOW, RED on sales-side]

---

## Purchasing-side playbook

*Applies when we are the customer / buyer. Usually their paper.*

**DPA requirement:** [dpa_requirement]
**Audit rights required from vendors:** [audit_rights_purchasing]
**Subcontracting:** [subcontracting]
**SLA minimum:** [sla_minimum]

**The one deal-breaker (purchasing-side):** [purchasing_deal_breaker]

---

## DORA ICT provider status

**ICT register status:** [dora_ict_register_status]
**Critical ICT third-party providers:**
[critical_ict_providers as YAML list]

---

## Escalation matrix

| Can approve | Without escalation (EUR) | Escalates to | Via |
|---|---|---|---|
| [role] | [EUR threshold] | [role] | [Slack/email/meeting] |
| [role] | [EUR threshold] | [role] | [method] |
| [role] | [EUR threshold] | [Board/CFO] | [method] |

**Automatic escalation triggers (regardless of EUR value):**
- [trigger list]

---

## House style

**Output language:** [language — Finnish/German/English/other]
**Where work product goes:** [Drive folder / local path / inline only]
**Renewal alerts go to:** [Slack channel / email / inline only]
```

---

## Summary and next steps

After writing `commercial.md`, show a one-paragraph summary of what was configured, then offer:

> "Your commercial playbook is ready. What's next?
> 1. **Review a contract** — paste or attach one and run `/eu-legal:contract-review`
> 2. **Review an NDA** — run `/eu-legal:nda-review`
> 3. **Review a vendor agreement** — run `/eu-legal:vendor-agreement-review`
> 4. **Update a specific playbook section** — run `/eu-legal:playbook-init`"
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/commercial-cold-start/SKILL.md | head -10
```

Confirm: frontmatter parses, name matches directory name.

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/commercial-cold-start/SKILL.md
git commit -m "feat(eu-legal): add commercial-cold-start skill (Plan 3, Task 1)"
```

---

## Task 2: `contract-review` skill

**File:** `eu-legal/skills/contract-review/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/contract-review
```

- [ ] **Step 2: Write `eu-legal/skills/contract-review/SKILL.md`**

```markdown
---
name: contract-review
description: >
  Review a commercial agreement against your EU playbook — detects contract type,
  applies the right playbook side (sales or purchasing), checks GDPR data processing
  clause requirements, flags governing law outside the EU, and surfaces DORA Art. 30
  ICT requirements for financial entities. Use when the user says "review this
  contract", "check this MSA", "is this agreement okay", or attaches an inbound
  commercial agreement for review.
argument-hint: "[file path | Drive link | paste text]"
---

# /eu-legal:contract-review

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md` (base profile). If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/commercial.md` (commercial playbook). If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:commercial-cold-start` first — I need your commercial playbook before reviewing contracts."
3. Get the agreement (file path, Drive link, or pasted text). If none provided, ask.
4. Run the workflow below.

---

## Purpose

Review a commercial agreement against the team's EU-law playbook. Surface every material deviation, flag GDPR and DORA obligations where applicable, and produce a prioritised issue list the reviewer can act on in one pass.

**This is a draft for attorney review and does not constitute legal advice.**

---

## Step 1: Detect contract type and side

Read document titles and the first two pages. Classify:

| Type | Description | Deep-review skill |
|---|---|---|
| NDA / Confidentiality Agreement | Confidentiality as the main agreement | `/eu-legal:nda-review` gives deeper NDA-specific review |
| MSA / Services Agreement / SOW | Master services, statement of work, consulting | This skill |
| SaaS / Subscription / Cloud Services | Recurring fee, online service, auto-renewal | This skill |
| Vendor / Procurement Agreement | Purchasing from a supplier | This skill + DORA ICT check if financial entity |
| Other | Licence, reseller, partnership | This skill |

If the contract is an NDA: run this review but note at the top of the output:
> "This is an NDA. `/eu-legal:nda-review` provides a deeper NDA-specific review using EU Trade Secrets Directive framing. Consider running it after this general review."

**Which side?** Determine whether the company is sales-side (selling its product/service) or purchasing-side (buying from a vendor). Apply the matching playbook section from `commercial.md`. If not obvious, ask.

---

## Step 2: Deal-breaker check

Check the relevant "deal-breaker" from `commercial.md` first. If present:

```
## DEAL-BREAKER PRESENT

Section [X.X] contains [the deal-breaker]. Per the playbook, this is a hard no.
Recommended next step: escalate to [approver from commercial.md] before proceeding.

Detailed review below is for completeness but is moot unless this is resolved.
```

---

## Step 3: Term-by-term review

### 3a. Playbook comparison

For each playbook category in `commercial.md` (liability, IP, data, termination, governing law), find the contract section and compare. For each deviation:

```
### [Section X.X]: [Issue name]

**Playbook says:** [position from commercial.md]

**Contract says:**
> "[exact quote]"

**Gap:** Missing term | Weaker than standard | Outside fallbacks | Unacceptable

**Legal risk:** 🔴 Blocking | 🟠 High | 🟡 Medium | 🟢 Low
**Business friction:** 🔴 Blocks deals | 🟠 Slows deals | 🟡 Confuses customers | 🟢 Invisible

**Why it matters:** [one sentence in plain language]

**Proposed redline:** "[specific replacement language]"

**If they won't move:** [fallback from commercial.md, or "escalate to [person]"]
```

### 3b. EU Trade Secrets Directive check (confidentiality clauses)

Check confidentiality / trade secrets provisions against Directive (EU) 2016/943 on the protection of undisclosed know-how and business information.

- Does the definition of confidential information align with Art. 2(1) TSD? (Information that is secret, has commercial value because it is secret, and has been subject to reasonable steps to keep it secret.)
- Are common-law trade secret formulations present ("misappropriation", "improper means" in US-law sense)? Flag these: "This clause uses US common law trade secret language. Reframe using Directive 2016/943 Art. 2 definition and Art. 4 unlawful acquisition / use / disclosure framing. `[model knowledge — verify]`"
- If governing law is Finnish: note that Liikesalaisuuslaki (Business Secrets Act 595/2018) implements Directive 2016/943. `[model knowledge — verify]`
- If governing law is German: note that GeschGehG (Geschäftsgeheimnisgesetz 2019) implements Directive 2016/943. `[model knowledge — verify]`

### 3c. GDPR data processing clause check

Ask: does this vendor / counterparty process personal data on behalf of the company?

Signals in the contract: reference to "data", "personal data", "data processing", "GDPR", "DPA", "data subjects", customer data, employee data.

**If personal data is processed:**
- Is there a Data Processing Agreement (DPA) or Art. 28 GDPR-compliant data processing clause? If no: 🔴 RED — mandatory for any processor relationship under Art. 28 GDPR. Flag: "No DPA or data processing clause found. GDPR Art. 28 requires a written contract between controller and processor. This agreement cannot proceed to signature without one."
- If a DPA is referenced by URL ("DPA available at [URL]"), note: "DPA incorporated by reference but not reviewed. Route to `/eu-legal:dpa-review` (if installed) or fetch and review before signing. `[DPA unread — verify]`"
- Check for cross-border data transfer mechanism if vendor is outside the EEA: standard contractual clauses (SCCs), adequacy decision, binding corporate rules. If none: 🔴 RED.

**If no personal data is processed:** Note "No DPA required — agreement does not involve personal data processing."

### 3d. DORA ICT third-party check (financial entities only)

**Gate:** Only run this step if:
- Entity type in `~/.claude/plugins/config/eu-legal/CLAUDE.md` is a financial entity (one of: `credit_institution`, `payment_institution`, `e_money_institution`, `investment_firm`, `aifm`, `ucits`, `insurance`, `reinsurance`)
- AND this is a purchasing-side agreement (company is the customer)
- AND the vendor provides ICT services (network, hardware, software, data, cloud, managed services)

If all three conditions are met:

**If `VELVOITE_API_KEY` is set:**
Call `mcp__velvoite__get_canonical_obligations` with:
- `regulation`: `dora`
- `actor_role`: `financial_entity`

From the results, surface the Art. 30 minimum contract clause requirements and check each one against the draft agreement. Flag any missing or insufficient clause as 🔴 RED.

**If `VELVOITE_API_KEY` is not set:**
Use the static DORA Art. 30 checklist:

| Art. 30 requirement | Present in contract? |
|---|---|
| Clear description of all ICT services and functions to be provided | |
| Locations where ICT services are provided and data is processed, including sub-locations | |
| Provisions on availability, authenticity, integrity, and confidentiality of data | |
| Provisions ensuring access, recovery, and return of data in the event of insolvency, discontinuation, or resolution | |
| Service level descriptions and quantitative and qualitative performance targets | |
| Relevant provisions on ICT security, including minimum information security standards | |
| Sub-contracting provisions: approval of sub-contractors, obligation to inform of changes | |
| Full cooperation with authorities, including the financial entity's competent authority | |
| Termination rights, including notice periods | |
| Exit plan: assistance in transition, data portability | |
| Audit rights: financial entity's right to audit, right to appoint third-party auditors | |
| Incident reporting: ICT-related incident notification obligations | |

Flag each missing clause as 🔴 RED with note: "DORA Art. 30 — required minimum contract clause. Missing or insufficient in this draft. `[model knowledge — verify against DORA text]`"

### 3e. Governing law check

- If governing law is an EU member state: note which one; note any jurisdiction-specific issues.
- If governing law is outside the EU (UK post-Brexit, US, Singapore, etc.): 🟠 HIGH flag: "Governing law is [jurisdiction]. Your playbook prefers EU law. Non-EU governing law affects enforcement, regulatory obligations under GDPR and DORA, and jurisdiction of supervisory authorities. `[review]`"
- Compare to `commercial.md` governing law preference. Flag if not matching.

---

## Step 4: Favorable terms and gaps

**Better than our standard:** Terms where the counterparty gave more than the playbook requires. Note as trade bait.

**Missing entirely:** Standard provisions absent from the draft (common omissions: force majeure, assignment restriction, entire agreement / no reliance, notice provisions, set-off rights).

---

## Step 5: Escalation routing

Check `commercial.md` escalation matrix:
- Contract value vs. EUR thresholds
- Any 🔴 BLOCKING issues
- Any automatic escalation triggers

State clearly who needs to approve.

---

## Output

```
CONFIDENTIAL — INTERNAL LEGAL ANALYSIS — DRAFT FOR ATTORNEY REVIEW

# Contract Review: [Counterparty] [Agreement Type]

**Reviewed:** [date]
**Contract value:** EUR [amount] / [term] (if stated)
**Our role:** [Customer | Vendor]
**Governing law:** [jurisdiction]
**Playbook side applied:** [Sales-side | Purchasing-side]

---

## Bottom line

[Two sentences. Can we sign this? What must change first?]

**Issues (legal risk):** [N]🔴 [N]🟠 [N]🟡 [N]🟢
**Issues (business friction):** [N]🔴 [N]🟠 [N]🟡 [N]🟢

**Approval needed from:** [name or role from escalation matrix]

---

## Deal-breaker check

[Clear | DEAL-BREAKER PRESENT — see above]

---

## Issues by severity

[All deviation blocks from Step 3, grouped Critical → Low]

---

## GDPR data processing

[DPA status and findings]

---

## DORA ICT compliance

[Only present for financial entities + ICT vendor agreements. Table or "Not applicable."]

---

## Favorable terms

[list]

---

## Missing provisions

[list]

---

## Approval routing

[name, threshold, method]
```

**Guardrail:** Append to every output — "This analysis is a draft for attorney review and does not constitute legal advice."

---

## Redline granularity

Edit at the smallest possible granularity. Replace a word before a phrase, a phrase before a sentence, a sentence before a clause. Only replace a whole clause when surgical edits would be harder to read than a fresh draft — and say so when you do.

---

## Close with next-steps decision tree

> **What next?**
> 1. **Run `/eu-legal:nda-review`** — for deeper NDA-specific analysis (if this was an NDA)
> 2. **Run `/eu-legal:vendor-agreement-review`** — for deeper DORA ICT compliance table (if this is a vendor agreement)
> 3. **Escalate** — I'll draft a note to [approver] with the key issues and what decision is needed
> 4. **Get more facts** — I'd want to know [open questions]. I'll draft those as questions.
> 5. **Add to renewal tracker** — if this agreement has an auto-renewal clause, I'll add it to the register
> 6. **Something else**
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/contract-review/SKILL.md | head -10
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/contract-review/SKILL.md
git commit -m "feat(eu-legal): add contract-review skill (Plan 3, Task 2)"
```

---

## Task 3: `nda-review` skill

**File:** `eu-legal/skills/nda-review/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/nda-review
```

- [ ] **Step 2: Write `eu-legal/skills/nda-review/SKILL.md`**

```markdown
---
name: nda-review
description: >
  Dedicated EU-law NDA review — deeper than contract-review for NDAs. Applies
  EU Trade Secrets Directive 2016/943 framing, detects mutual vs. one-way, checks
  standard confidentiality provisions, and flags governing law deviations. Also
  loaded by /eu-legal:contract-review when an NDA is detected. Use when the user
  says "review this NDA", "check this confidentiality agreement", "is this NDA
  okay", or attaches an inbound NDA.
argument-hint: "[file path | Drive link | paste text]"
---

# /eu-legal:nda-review

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/commercial.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:commercial-cold-start` first."
3. Get the NDA (file path, Drive link, or pasted text). If none, ask.
4. Run the workflow below.

---

## Purpose

Most inbound NDAs are fine. A few have landmines. This skill sorts them quickly so legal only reads the ones that matter — using EU Trade Secrets Directive framing rather than US common law analysis.

**This is a draft for attorney review and does not constitute legal advice.**

---

## EU legal framework

This skill applies:
- **Directive (EU) 2016/943** on the protection of undisclosed know-how and business information (Trade Secrets Directive) — definition of trade secret (Art. 2(1)), unlawful acquisition (Art. 4(2)), unlawful use and disclosure (Art. 4(3)), lawful acquisition (Art. 3).
- **National implementing legislation:** Liikesalaisuuslaki 595/2018 (Finland); GeschGehG — Geschäftsgeheimnisgesetz (Germany); Wet bescherming bedrijfsgeheimen (Netherlands). Note the applicable national law based on governing law in `commercial.md`. `[model knowledge — verify]`
- **GDPR (Regulation (EU) 2016/679):** If the NDA involves disclosure of personal data or employee information, flag that the NDA does not substitute for GDPR compliance — personal data shared under confidentiality is still subject to data subject rights.

Do NOT apply US common law trade secrets analysis (Uniform Trade Secrets Act, Defend Trade Secrets Act). If the NDA uses US common law language, flag it and propose EU TSD framing as a replacement.

---

## Step 1: Orientation

Before triaging, answer:

| Question | Answer |
|---|---|
| Mutual or one-way? | |
| Whose paper is it? | |
| Which side are we? | |
| Purpose of disclosure | |
| Governing law | |
| Duration | |
| Contemplated transaction | |

**One-way NDA check:** If the NDA is one-way (only one party discloses), do not auto-flag RED. Ask:
> "Is your organisation the only party disclosing information, or should this be mutual? A one-way NDA is appropriate when only one side shares — for example, sharing your technology with a vendor who shares nothing back."
Use the answer plus the `commercial.md` playbook to determine the bucket.

---

## Step 2: Scope check

**Before reviewing NDA-specific provisions, check whether the document is doing more than its name suggests.** NDAs can hide: standstills, licensing grants, exclusivity, non-solicits, non-competes, IP assignments, right of first refusal, MFN clauses, arbitration provisions that govern far more than confidentiality disputes.

If the NDA contains obligations beyond confidentiality: **auto-YELLOW regardless of NDA-term analysis.** Flag:
> "This document is labelled an NDA but contains [provision]. It is more than an NDA. Route for attorney review."

---

## Step 3: Triage

Classify into GREEN / YELLOW / RED using `commercial.md` NDA triage positions.

**GREEN — route to signature**

The NDA satisfies every position in `commercial.md`, no term triggers a RED flag, and the playbook has attorney-reviewed NDA positions. GREEN cannot be issued against missing or `[PLACEHOLDER]` positions.

**YELLOW — needs a lawyer's eyes on specific items**

One or more terms deviate from the playbook but are not categorical deal-breakers, OR a term appears that `commercial.md` doesn't address.

**RED — stop, talk to legal first**

The NDA hits a "never accept" position in `commercial.md`, or a term is incompatible with the company's standard posture.

---

## Step 4: Detailed checks

For each check, apply the position from `commercial.md`. If the playbook is silent on a term, ask the user and offer to record the answer.

### Definition of Confidential Information (EU TSD framing)

Check against Art. 2(1) TSD three-limb test:
1. The information is secret (not generally known or readily accessible in the relevant circles)
2. The information has commercial value because of its secrecy
3. Reasonable steps have been taken to keep it secret

Flag if the definition is either too narrow (excludes orally disclosed information without confirmation window) or too broad (captures publicly available information). Flag any US common law language ("misappropriation", "improper means") and propose:
> "Replace with EU TSD Art. 2(1) definition: 'information that (a) is secret in the sense that it is not, as a body or in the precise configuration and assembly of its components, generally known among or readily accessible to persons within the circles that normally deal with the kind of information in question; (b) has commercial value because it is secret; and (c) has been subject to reasonable steps under the circumstances, by the person lawfully in control of the information, to keep it secret.' `[Directive (EU) 2016/943 Art. 2(1) — model knowledge — verify]`"

### Standard carve-outs (Art. 3 TSD — lawful acquisition)

The five standard carve-outs that align with Directive Art. 3 (lawful acquisition of trade secrets):
1. Information that is or becomes public other than through breach
2. Information the receiving party already had before disclosure
3. Information independently developed without reference to the disclosed information (Art. 3(1)(b) TSD)
4. Information received from a third party without restriction (Art. 3(1)(c) TSD)
5. Information required to be disclosed by law or court order (with notice to discloser where legally permitted)

Flag missing carve-outs. Flag if carve-out (3) uses US language ("developed without use of confidential information") rather than EU TSD framing ("independent discovery or creation" per Art. 3(1)(b)).

### Unlawful acquisition, use, and disclosure (Arts. 4(2) and 4(3) TSD)

Check that the prohibited activities section aligns with TSD Art. 4 rather than US UTSA / DTSA language. The TSD frames unlawful use and disclosure as: use or disclosure by a person who acquired the trade secret unlawfully, or who obtained it in breach of a confidentiality or limited use obligation.

If the NDA uses "improper means" (UTSA language) without a TSD equivalent, flag it.

### Duration and survival

Check: initial term, post-term survival of confidentiality obligations, whether trade secrets are carved out with longer / indefinite protection.

Note: Under TSD Art. 10(2), courts may set a limited period for confidentiality injunctions where the secret has ceased to be a trade secret. Perpetual NDAs are enforceable in the EU but courts may interpret them narrowly for information that becomes public.

Apply the duration position from `commercial.md`. Flag if outside the playbook range.

### Residuals clause

A residuals clause allows the receiving party to use information retained in unaided human memory. Flag if present; apply `commercial.md` position. If `commercial.md` is silent: ask. "A residuals clause is present in Section [X]. Should this be GREEN (acceptable), YELLOW (negotiable), or RED (never accept) for your organisation? I'll record your answer."

### Return or destruction on termination

Check: is there an obligation to return or destroy confidential information on termination? Is there a carve-out for backup / archival copies and legal holds?

EU data protection note: if confidential information includes personal data, return/destruction must also comply with GDPR Art. 17 (right to erasure) and any applicable retention obligations. Flag if no alignment.

### Governing law

Compare to `commercial.md` governing law preference.
- If EU governing law: note the applicable national TSD implementation.
- If Finnish law: "Liikesalaisuuslaki (Business Secrets Act 595/2018) implements Directive 2016/943 in Finland. Art. 4 of the Act defines trade secrets; §§ 4–7 define unlawful acquisition, use, and disclosure. `[model knowledge — verify]`"
- If German law: "GeschGehG (Gesetz zum Schutz von Geschäftsgeheimnissen, 2019) implements Directive 2016/943 in Germany. `[model knowledge — verify]`"
- If outside EU: 🟠 flag. "Governing law is [jurisdiction]. Your playbook prefers EU law. TSD protections may not apply. `[review]`"

### Restrictive covenants

Check for non-solicits, non-competes, exclusivity. Flag any covenants that go beyond confidentiality — these auto-YELLOW the NDA regardless of other terms.

---

## Step 5: Redline granularity

Edit at the smallest possible granularity — replace a word before a phrase, a phrase before a sentence. Only replace a whole clause when surgical edits would be harder to read than a fresh draft.

---

## Output

```
CONFIDENTIAL — INTERNAL LEGAL ANALYSIS — DRAFT FOR ATTORNEY REVIEW

## NDA Triage: [Counterparty]

[GREEN — route to signature | YELLOW — flag for [approver] | RED — stop, talk to legal first]

### Executive Summary

[For GREEN: "No red flags identified under the playbook and EU TSD framework. Route for signature per standard process."]
[For YELLOW/RED: bullet list of specific actionable items]

### Flagged items (YELLOW/RED only)

**1. [Issue]** — Section [X]
   What: [one line]
   Why flagged: [playbook position or "playbook is silent on this"]
   Legal risk: 🔴/🟠/🟡/🟢 | Business friction: 🔴/🟠/🟡/🟢
   Proposed redline: "[specific language]"

[repeat for each flag]

### Everything else

| Check | Status | Framework |
|---|---|---|
| Mutual / one-way | [pass/flag] | commercial.md position |
| CI definition (TSD Art. 2(1)) | [pass/flag] | Directive 2016/943 |
| Standard carve-outs (TSD Art. 3) | [pass/flag] | Directive 2016/943 |
| Unlawful use / disclosure (TSD Art. 4) | [pass/flag] | Directive 2016/943 |
| Duration and survival | [pass/flag] | commercial.md position |
| Residuals | [pass/flag] | commercial.md position |
| Return / destruction | [pass/flag] | commercial.md position |
| Governing law | [pass/flag] | commercial.md preference |
| Restrictive covenants | [pass/flag] | commercial.md / auto-YELLOW |
```

**Guardrail:** Append to every output — "This analysis is a draft for attorney review and does not constitute legal advice."

---

## Close with next-steps decision tree

> **What next?**
> 1. **Route for signature** — if GREEN, confirm the approval process
> 2. **Escalate** — I'll draft a note to [approver] with the flagged items
> 3. **Redline and return** — I'll produce specific redline language for each flagged item
> 4. **Use our paper instead** — recommend if counterparty is a startup or small vendor
> 5. **Something else**
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/nda-review/SKILL.md | head -10
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/nda-review/SKILL.md
git commit -m "feat(eu-legal): add nda-review skill (Plan 3, Task 3)"
```

---

## Task 4: `vendor-agreement-review` skill

**File:** `eu-legal/skills/vendor-agreement-review/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/vendor-agreement-review
```

- [ ] **Step 2: Write `eu-legal/skills/vendor-agreement-review/SKILL.md`**

```markdown
---
name: vendor-agreement-review
description: >
  Dedicated vendor agreement review for procurement — applies the purchasing-side
  playbook, runs a DORA Art. 30 ICT third-party compliance check for financial
  entities, and flags concentration risk if the vendor already appears in your ICT
  register. Most relevant for financial institutions procuring technology or services.
  Use when the user says "review this vendor agreement", "check this MSA from a
  supplier", "DORA compliance check on this contract", or attaches an inbound
  vendor agreement for procurement review.
argument-hint: "[file path | Drive link | paste text]"
---

# /eu-legal:vendor-agreement-review

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/commercial.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:commercial-cold-start` first."
3. Get the agreement (file path, Drive link, or pasted text). If none, ask.
4. Run the workflow below.

---

## Purpose

Review an inbound vendor agreement against the purchasing-side playbook. For financial entities procuring ICT services, perform a full DORA Art. 30 minimum contract clause check. Flag concentration risk if the vendor is already in the ICT register.

**This is a draft for attorney review and does not constitute legal advice.**

---

## Step 1: Orient

Read the whole agreement once, fast. Answer:

| Question | Answer |
|---|---|
| Agreement type | MSA / SaaS subscription / Professional services / Licence / Other |
| Vendor / counterparty | Name |
| Is this an ICT service? | Yes (network, hardware, software, data, cloud, managed service) / No |
| Dollar / EUR value | Annual / total contract value if stated |
| Term | Length, renewal mechanics |
| DPA status | Attached / referenced by URL / missing |
| Order form | Separate doc or integrated |
| Concentration risk | Is this vendor already in `commercial.md` critical_ict_providers list? |

**EUR value handling:** If the MSA does not state a value (price is in an Order Form not in hand), stop and ask:
> "The MSA itself doesn't state a contract value. The Order Form carries the price. Your escalation threshold is EUR [from commercial.md]. Before routing, I need the ACV. Options: (1) paste the Order Form value, (2) tell me above or below EUR [threshold] and I'll route accordingly, (3) route conservatively to the higher approver."

**DPA by reference:** If the DPA is incorporated by URL, note: "DPA referenced at [URL] but not reviewed. Route to `/eu-legal:dpa-review` (if installed) before signing. `[DPA unread — verify]`"

**Concentration risk flag:** If the vendor name appears in `commercial.md` `critical_ict_providers`, flag:
> "Concentration risk: [Vendor] is already listed as a critical ICT third-party provider in your ICT register. This additional agreement increases concentration in this provider. Consider whether this is acceptable under your DORA ICT risk management framework."

---

## Step 2: Deal-breaker check

Check `commercial.md` purchasing-side deal-breaker first. If present, flag at the top and stop detailed review until resolved.

---

## Step 3: DORA Art. 30 ICT third-party compliance check

**Gate:** Run this step only if:
- Entity type in `CLAUDE.md` is a financial entity (`credit_institution`, `payment_institution`, `e_money_institution`, `investment_firm`, `aifm`, `ucits`, `insurance`, `reinsurance`)
- AND the vendor provides ICT services (as determined in Step 1)

**If `VELVOITE_API_KEY` is set:**

Call `mcp__velvoite__get_canonical_obligations` with:
- `regulation`: `dora`
- `actor_role`: `financial_entity`

From the obligations returned, extract the Art. 30 minimum contract clause requirements. For each requirement, check whether it is present and adequate in the draft agreement.

**If `VELVOITE_API_KEY` is not set:**

Use the static DORA Art. 30 checklist below. Each clause marked MISSING or INSUFFICIENT is 🔴 RED.

| Art. 30 Minimum Clause | Status | Notes |
|---|---|---|
| **Description of all services and functions** — clear description of all ICT services and functions to be provided by the ICT third-party service provider, including indication of whether sub-contracting of an ICT service supporting a critical or important function is permitted (Art. 30(2)(a)) | | |
| **Locations** — locations where ICT services are to be provided and where data is to be processed, including sub-locations; ICT third-party service provider must notify the financial entity if it intends to change such locations (Art. 30(2)(b)) | | |
| **Data provisions** — provisions on availability, authenticity, integrity, and confidentiality in relation to the protection of data, including personal data (Art. 30(2)(c)) | | |
| **Access and recovery** — provisions ensuring access, recovery, and return in an easily accessible format of personal and non-personal data processed by the financial entity in the case of insolvency, resolution, or discontinuation of business operations of the ICT third-party service provider (Art. 30(2)(d)) | | |
| **Service levels** — service level descriptions, including updates and revisions thereof, with precise quantitative and qualitative performance targets, allowing effective monitoring by the financial entity so that corrective actions can be undertaken without undue delay (Art. 30(2)(e)) | | |
| **Information security** — relevant provisions on ICT security, including minimum information security standards (Art. 30(2)(f)) | | |
| **Sub-contracting** — provisions on the conditions for the sub-contracting of ICT services supporting critical or important functions, including the right to object to sub-contracting (Art. 30(2)(g)) | | |
| **Cooperation with authorities** — full cooperation of the ICT third-party service provider with competent authorities and resolution authorities of the financial entity (Art. 30(2)(h)) | | |
| **Termination rights** — termination rights of the financial entity, including a reasonable notice period for the termination of the ICT contracts (Art. 30(2)(i)) | | |
| **Exit plan** — participation and full support of the ICT third-party service provider to the financial entity's ICT exit plan, including the transfer of data and assistance in transition (Art. 30(2)(j)) | | |
| **Audit rights** — the unconditional right to inspect and audit the ICT third-party service provider, including the right to appoint third-party auditors (Art. 30(2)(k)) | | |
| **Incident reporting** — clear reporting by the ICT third-party service provider of ICT-related incidents (Art. 30(2)(l)) | | |

`[DORA Art. 30 — model knowledge — verify against Regulation (EU) 2022/2554 text]`

**Output format for DORA table:**

```
## DORA Art. 30 Compliance Table

Financial entity type: [from CLAUDE.md]
Vendor: [counterparty name]
ICT service: [yes/no and description]

| Clause | Status | Section in contract | Gap / action needed |
|---|---|---|---|
| Description of services | 🔴 MISSING / 🟠 INSUFFICIENT / 🟢 PRESENT | | |
[one row per Art. 30(2) requirement]

**DORA compliance summary:** [N] present, [N] insufficient, [N] missing.
[N] missing or insufficient clauses require amendment before this agreement can be signed by a DORA-regulated entity.
```

---

## Step 4: Purchasing-side playbook comparison

Apply `commercial.md` purchasing-side playbook. For each deviation, produce the standard finding block:

```
### [Section X.X]: [Issue name]

**Playbook says:** [position]
**Contract says:** > "[exact quote]"
**Gap:** [type]
**Legal risk:** 🔴/🟠/🟡/🟢 | **Business friction:** 🔴/🟠/🟡/🟢
**Why it matters:** [one sentence]
**Proposed redline:** "[specific replacement language]"
**If they won't move:** [fallback or "escalate to [person]"]
```

Categories to cover: SLA, liability, IP, data protection / DPA, governing law, termination, audit rights, auto-renewal.

---

## Step 5: GDPR data processing clause check

Same as `contract-review` Step 3c. If personal data processed: check for DPA / Art. 28 GDPR clause, cross-border transfer mechanism if outside EEA.

---

## Step 6: Favorable terms and gaps

**Better than our standard:** Note as trade bait.
**Missing entirely:** Common omissions — force majeure, assignment restriction, business continuity / disaster recovery (relevant for ICT), step-in rights.

---

## Step 7: Escalation routing

Check `commercial.md` escalation matrix against EUR value, 🔴 issues, and automatic escalation triggers (including: ICT third-party agreement for financial entity → always flag to legal for DORA review).

---

## Output

```
CONFIDENTIAL — INTERNAL LEGAL ANALYSIS — DRAFT FOR ATTORNEY REVIEW

# Vendor Agreement Review: [Counterparty] [Agreement Type]

**Reviewed:** [date]
**Contract value:** EUR [amount] / [term]
**Our role:** Customer / Purchasing-side
**Governing law:** [jurisdiction]
**ICT service provider:** [Yes / No]
**DORA check:** [Applicable — [N] gaps found | Not applicable]

---

## Bottom line

[Two sentences. Can we sign this? What must change first?]

**DORA gaps:** [N]🔴 [N]🟠 (if applicable)
**Playbook issues (legal risk):** [N]🔴 [N]🟠 [N]🟡 [N]🟢
**Playbook issues (business friction):** [N]🔴 [N]🟠 [N]🟡 [N]🟢
**Approval needed from:** [name or role]

---

## Deal-breaker check

[Clear | DEAL-BREAKER PRESENT]

---

## DORA Art. 30 Compliance Table

[Table from Step 3, or "Not applicable — not an ICT service / not a financial entity."]

---

## Issues by severity

[All playbook deviation blocks]

---

## GDPR data processing

[DPA status and findings]

---

## Favorable terms and missing provisions

[lists]

---

## Approval routing

[escalation details]
```

**Guardrail:** Append to every output — "This analysis is a draft for attorney review and does not constitute legal advice."

---

## Redline granularity

Edit at the smallest possible granularity. Replace a word before a phrase, a phrase before a sentence, a sentence before a clause.

---

## Close with next-steps decision tree

> **What next?**
> 1. **Escalate** — I'll draft a note to [approver] with the DORA gaps and playbook issues
> 2. **Redline and return** — I'll produce a consolidated redline package
> 3. **Add to renewal tracker** — if auto-renewal found, I'll add it to the register
> 4. **Add vendor to ICT register** — if this is a critical ICT provider, I'll update `commercial.md`
> 5. **Route DPA for review** — hand the DPA URL or text to `/eu-legal:dpa-review`
> 6. **Something else**
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/vendor-agreement-review/SKILL.md | head -10
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/vendor-agreement-review/SKILL.md
git commit -m "feat(eu-legal): add vendor-agreement-review skill (Plan 3, Task 4)"
```

---

## Task 5: `escalation-flagger` skill

**File:** `eu-legal/skills/escalation-flagger/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/escalation-flagger
```

- [ ] **Step 2: Write `eu-legal/skills/escalation-flagger/SKILL.md`**

```markdown
---
name: escalation-flagger
description: >
  Route a contract issue to the right approver per the escalation matrix in
  commercial.md, and draft the ask. EU-scoped: thresholds in EUR, EU-specific
  auto-triggers (cross-border data transfers outside EEA, ICT third-party DORA
  check, non-EU governing law, liability cap below regulatory exposure minimum).
  Use when the user says "who needs to approve this", "escalate this", "does this
  need GC sign-off", "route this for approval", or when another skill finds an issue
  that exceeds the reviewer's authority.
argument-hint: "[describe the issue, or reference a review memo]"
---

# /eu-legal:escalation-flagger

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/commercial.md` escalation section. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:commercial-cold-start` first — I need your escalation matrix before routing."
3. Get the issue: from the user's description, or from a review memo referenced in the args.
4. Run the workflow below.

---

## Purpose

Names the right approver for a contract issue per the EUR escalation matrix in `commercial.md` and drafts the ask — in a form the approver can decide from without pulling up the contract.

---

## Step 1: Characterise the issue

What is being escalated?

- **EUR threshold:** Contract value exceeds someone's approval authority
- **Term deviation:** A term is outside the playbook fallbacks — someone more senior must decide whether to accept
- **Automatic EU trigger:** One of the always-escalate items (see below)
- **Business decision:** Not a legal call — needs the business owner

Do not escalate issues that are within the playbook's documented fallbacks. If the term is acceptable per `commercial.md`, it does not need to go up.

---

## Step 2: Check EU-specific automatic escalation triggers

These escalate regardless of EUR value. Check each:

| Trigger | Action |
|---|---|
| Any contract with cross-border data transfers outside the EEA (no adequacy decision, no SCCs, no BCRs) | Legal review required — GDPR Art. 44–49 |
| Any ICT third-party agreement for a financial entity (DORA in scope) | DORA compliance check required — route to `/eu-legal:vendor-agreement-review` if not yet run |
| Governing law outside the EU | Flag to legal — enforcement of EU rights (GDPR, DORA, TSD) may be impaired |
| Liability cap below EUR [threshold from commercial.md] | Flag — potential exposure exceeds protected threshold |
| Any uncapped liability provision | Automatic escalation regardless of EUR value |
| Any IP assignment to the vendor | Automatic escalation — legal sign-off required |
| Any term on commercial.md "never accept" list | Automatic escalation |

Plus any additional triggers in `commercial.md` `escalation_auto_triggers`. Check the list from `commercial.md` and apply all of them.

---

## Step 3: Match to the EUR escalation matrix

```
Is the issue an automatic trigger?
  → YES: escalate to the named person for that trigger (from commercial.md)
  → NO: continue

Is the contract value above the reviewer's EUR threshold?
  → YES: escalate to whoever has authority at that EUR level (from commercial.md)
  → NO: continue

Is the term deviation outside all documented fallbacks?
  → YES: escalate to whoever can approve non-standard terms (from commercial.md)
  → NO: reviewer can approve — no escalation needed
```

---

## Step 4: Name the approver

Be specific. Not "escalate to legal leadership" — name the person or role from `commercial.md`. If the matrix doesn't name anyone for this situation: "The escalation matrix doesn't cover [situation]. Suggest asking [GC name or role from commercial.md] who owns this."

---

## Step 5: Draft the ask

The approver should be able to decide from the message alone.

```
**Escalating to:** [name or role from commercial.md]
**Via:** [Slack / email / meeting — from commercial.md]
**Urgency:** [deadline, if any]

---

Hey [name] —

Need your call on the [Counterparty] [agreement type]. [One sentence on deal context.]

**The issue:** [Plain language, one paragraph. What the contract says, why it's outside
our standard, what the actual risk is. EUR amounts throughout.]

**What the contract says:**
> "[exact quote]"

**What our playbook says:** [from commercial.md — the specific position violated]

**Applicable EU framework:** [DORA Art. 30 / GDPR Art. 28 / TSD Art. 4 / N/A] `[model knowledge — verify]`

**Options:**
1. **Accept** — [one line on why this might be okay]
2. **Push back with:** "[proposed counter-language]" — [one line on likely counterparty reaction]
3. **Walk** — [one line on whether that's realistic given the business context]

**My recommendation:** [which option and why, briefly]

**Need a decision by:** [date, if deadline exists]

[Link to full review memo]
```

---

## Calibration: when in doubt, escalate with a note

The cost of an unnecessary escalation is ~30 seconds of the approver's time. The cost of a missed escalation is signing an unapproved term. The costs are not symmetric. **When in doubt, escalate.**

If a term appears that `commercial.md` doesn't address, don't guess the threshold — ask the reviewing attorney whether this class of issue should escalate, and offer to record the answer in `commercial.md` so future reviews are consistent.

---

## What this skill does not do

- It does not approve anything. It routes.
- It does not decide between the options. The draft includes a recommendation but the approver decides.
- It does not send the escalation message — it drafts it. The lawyer sends it after reading.
- It does not apply US-specific thresholds or triggers. All amounts are in EUR.
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/escalation-flagger/SKILL.md | head -10
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/escalation-flagger/SKILL.md
git commit -m "feat(eu-legal): add escalation-flagger skill (Plan 3, Task 5)"
```

---

## Task 6: `playbook-init` skill

**File:** `eu-legal/skills/playbook-init/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/playbook-init
```

- [ ] **Step 2: Write `eu-legal/skills/playbook-init/SKILL.md`**

```markdown
---
name: playbook-init
description: >
  Guided setup or update of the commercial contracts playbook — walk through each
  playbook section conversationally and write commercial.md. Use when skipping
  commercial-cold-start, resetting specific positions, or when the user says "update
  my playbook", "change my liability position", "set my governing law", or "I want to
  reset my NDA positions". Note: /eu-legal:commercial-cold-start covers this as part
  of full setup.
argument-hint: "[section to update — e.g. 'liability' | 'governing-law' | 'nda-positions' | 'escalation' | blank for full run]"
---

# /eu-legal:playbook-init

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If missing or has `[PLACEHOLDER]`, stop: "Run `/eu-legal:cold-start-interview` first."
2. If `commercial.md` exists and is populated, load it. This is an update run: only modify the sections the user asks about, preserve the rest.
3. If `commercial.md` is missing or has all `[PLACEHOLDER]` values: run full setup (all sections below). Equivalent to `commercial-cold-start` playbook-only mode.
4. If an arg is provided (e.g., `liability`, `governing-law`): jump directly to that section. Otherwise guide through all sections.

---

## Purpose

Helps the user build or update their `commercial.md` playbook section by section, conversationally. Pre-populated with EU-sensible defaults so the user can accept, modify, or reject each position.

Note: `/eu-legal:commercial-cold-start` covers this as part of full plugin setup — run that on first install. Use `/eu-legal:playbook-init` to update a specific section or rebuild the playbook without re-running the full cold-start interview.

---

## EU-sensible defaults

Pre-populate these as starting points. Present them as defaults the user can accept, modify, or reject — do not present them as the "right" answer.

| Section | Default starting point |
|---|---|
| Preferred governing law | Finnish law (for Finnish-primary entities); German law (for German-primary entities) — read from `CLAUDE.md` primary jurisdiction |
| Cross-border dispute resolution | ICC arbitration, seat: Stockholm |
| Direct liability cap | 12 months of fees paid in the 12 months preceding the claim |
| Consequential damages | Excluded, with carve-outs for data breach, IP infringement, and gross negligence |
| Auto-renewal default notice | 30 days minimum (some EU member states impose longer statutory minimums — flag this) |
| DPA requirement (purchasing) | Vendor signs our DPA as processor |
| ICT audit rights | Right to audit (directly or via third-party auditors) — required for DORA in-scope entities |

---

## Section-by-section guide

Ask one section at a time. For each section:
1. Show the current value from `commercial.md` (or the EU-sensible default if not yet set)
2. Ask: "Does this match your position? Accept / modify / tell me your position"
3. Record the answer

### Section: Governing law and dispute resolution

> "Your current governing law preference is [from commercial.md or default]. Does this match your standard?
> - Preferred: [show current / default]
> - Acceptable: [show current / default]
> - Never: [show current / default — flag that outside-EU governing law is an automatic escalation trigger in the EU context]"

Offer to set `governing_law_never` to "any jurisdiction outside the EU" as an automatic escalation trigger.

For Finnish-law agreements: note that Finnish courts (District Court of Helsinki) are well-suited for domestic contracts. For cross-border EU: ICC or SCC arbitration is more common.

### Section: Liability cap

> "Your current direct liability cap is [from commercial.md or default: '12 months of fees paid in the preceding 12 months']. For each position below, tell me your standard, acceptable fallback, and never-accept:"

Walk through:
- Direct cap (multiple of fees)
- Indirect / consequential damages
- Carve-outs above the cap (data breach, IP, gross negligence, fraud — EU standard)
- Cap base definition ("fees paid in the 12 months preceding the claim" vs. other definitions)

Note: EU unfair contract terms rules (Directive 93/13/EEC for B2C; national implementations for B2B — e.g., Finnish Kuluttajansuojalaki, German BGB §§ 305–310) may constrain limitation clauses in standard terms. Flag this: "If you use standard terms (your form sent to customers or vendors without individual negotiation), EU unfair terms rules may apply. Blanket liability exclusions may be invalid. `[model knowledge — verify with local counsel]`"

### Section: NDA triage positions

> "Let's set your NDA triage positions — these determine when an NDA is GREEN (route to signature), YELLOW (flag for review), or RED (stop, talk to legal). For each item, tell me your position."

Walk through:
- Mutual vs. one-way NDAs (when is one-way acceptable?)
- Definition of confidential information (how broad?)
- Duration (acceptable range)
- Residuals clause (acceptable / not acceptable)
- Governing law (EU only? any EU? never outside EU?)
- Restrictive covenants in an NDA (auto-YELLOW or auto-RED?)

Note: "GREEN can only be issued if these positions have been reviewed by an attorney. I'll flag GREEN outputs against `[PLACEHOLDER]` positions as YELLOW automatically."

### Section: Sales-side playbook

Walk through each position: IP, audit rights, data processing, termination, deal-breaker.

### Section: Purchasing-side playbook

Walk through each position: DPA, audit rights, subcontracting, SLA minimum, deal-breaker.

### Section: Escalation thresholds

Walk through: EUR thresholds per role, automatic triggers.

---

## Write result

After all confirmed sections, update `~/.claude/plugins/config/eu-legal/commercial.md` — only the sections touched in this session. Preserve unchanged sections exactly as they were.

Show a diff-style summary of what changed:
```
Updated sections:
- Governing law: Finnish law preferred (was: [old value])
- Direct liability cap: 12 months fees (was: [PLACEHOLDER])
- NDA triage positions: added (was: not set)

Unchanged sections: purchasing-side playbook, escalation matrix
```

---

## After writing

> "Playbook updated. To use it:
> - Review a contract: `/eu-legal:contract-review`
> - Review an NDA: `/eu-legal:nda-review`
> - Review a vendor agreement: `/eu-legal:vendor-agreement-review`"
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/playbook-init/SKILL.md | head -10
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/playbook-init/SKILL.md
git commit -m "feat(eu-legal): add playbook-init skill (Plan 3, Task 6)"
```

---

## Task 7: `renewal-tracker` skill

**File:** `eu-legal/skills/renewal-tracker/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p eu-legal/skills/renewal-tracker
```

- [ ] **Step 2: Write `eu-legal/skills/renewal-tracker/SKILL.md`**

```markdown
---
name: renewal-tracker
description: >
  Track expiring contracts and flag renewal/termination decisions before notice
  windows close. Reads the renewal register at
  ~/.claude/plugins/config/eu-legal/renewal-register.yaml. Flags contracts with
  notice periods approaching (90/30/7 days), highlights auto-renewal clauses, and
  notes Finnish hiljainen uusiminen (automatic renewal) provisions. Use when the
  user asks "what's renewing soon", "what renewals are due", "did we miss a
  cancellation window", "add this to the renewal tracker", or on a scheduled basis.
argument-hint: "[--days N | --missed | --add]"
---

# /eu-legal:renewal-tracker

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md` (base profile — for entity type and jurisdiction context).
2. Load `~/.claude/plugins/config/eu-legal/commercial.md` if it exists (for escalation matrix and business owner defaults).
3. Read `~/.claude/plugins/config/eu-legal/renewal-register.yaml`. If missing or empty: see "Empty register" section below.
4. Run the workflow below.

---

## Purpose

Nobody reads a contract twice. The renewal date is extracted once at review time and lives in the register. This skill surfaces what is coming before the window closes — not after.

Not Velvoite-dependent. Works without `VELVOITE_API_KEY`.

---

## The register

Lives at `~/.claude/plugins/config/eu-legal/renewal-register.yaml`. Each entry:

```yaml
- counterparty: "Acme SaaS Oy"
  agreement: "Acme Platform Subscription Agreement"
  signed_date: 2025-06-15
  initial_term_end: 2026-06-15
  current_term_end: 2026-06-15           # rolls forward after each auto-renewal
  renewal_mechanism: "auto-renew annual"
  notice_period_days: 60
  notice_method: "email"                 # email / portal / kirjattu kirje (registered post) / per contract §X
  transit_buffer_days: 0                 # 0 for email; 5 for domestic registered post; 10 for international
  cancel_by_calendar: 2026-04-16         # current_term_end minus notice_period_days
  cancel_by_effective: 2026-04-16        # rolled back to last business day in governing-law jurisdiction
  send_by_effective: 2026-04-16          # cancel_by_effective minus transit_buffer_days
  cancel_by_roll_note: ""                # e.g., "rolled back from Sunday 2026-11-01"
  cancel_by_provenance: "[model calculation — verify against the notice clause]"
  hiljainen_uusiminen: false             # Finnish automatic renewal — flag explicitly
  price_on_renewal: "then-current list (uncapped)"
  annual_value_eur: 48000                # EUR (not USD)
  business_owner: "juha@company.fi"
  status: "active"                       # active | cancelled | renewed | lapsed
  notes: "Pricing uncapped — revisit before renewal."
```

**Rolling renewals:** Store `initial_term_end` for the record, but compute cancel-by dates from `current_term_end`. When a renewal fires (cancel window passes, no notice given), prompt:
> "This contract auto-renewed on [date]. Update the register: new `current_term_end` is [date + renewal period], new `cancel_by_effective` is [computed], new `send_by_effective` is [computed]. Confirm?"

**Transit time:** Alert off `send_by_effective`, not `cancel_by_effective`. A 60-day window with registered post (kirjattu kirje) is effectively ~55 days if transit_buffer_days is 5.

---

## Business-day check — EU governing law calendars

When computing or ingesting a cancel-by date, roll back to the last business day. Business day calendars depend on governing law jurisdiction:

- **Finnish law:** Finnish public holidays (yleiset pyhäpäivät) — Uudenvuodenpäivä, Loppiainen, Pitkäperjantai, Pääsiäinen, Vappu, Helatorstai, Juhannuspäivä, Pyhäinpäivä, Itsenäisyyspäivä, Joulu. `[model knowledge — verify against current Finnish holiday calendar]`
- **German law:** Federal holidays (gesetzliche Feiertage) plus Bundesland-specific holidays — if governing law is German, ask which Bundesland. `[model knowledge — verify]`
- **English law:** England and Wales bank holidays. `[model knowledge — verify]`
- **Other EU:** flag "Business-day roll-back uses [jurisdiction] as stated in governing law. Verify against [jurisdiction] public holiday calendar."

Roll BACK (never forward). Record both `cancel_by_calendar` (raw arithmetic) and `cancel_by_effective` (last effective business day), plus a `cancel_by_roll_note` if they differ.

---

## Finnish hiljainen uusiminen (automatic renewal) — explicit flag

Finnish law recognises hiljainen uusiminen — an agreement that renews automatically if neither party gives notice. This is common in Finnish commercial contracts and service agreements.

When adding a Finnish-law contract to the register, or when reviewing an agreement governed by Finnish law:
- Check explicitly for automatic renewal (hiljainen uusiminen) provisions.
- Flag these in the register: `hiljainen_uusiminen: true`.
- Output note: "This contract contains an automatic renewal clause (hiljainen uusiminen) under Finnish law. If no termination notice is given by [cancel_by_effective], the contract renews automatically for [renewal period]. Ensure the business owner is aware."

Do not assume a Finnish-law contract lacks this feature — it is often implied rather than explicit. If governing law is Finnish and the renewal mechanism is ambiguous, flag for review: "Finnish law may allow automatic renewal (hiljainen uusiminen) in the absence of an explicit notice-of-termination provision. Verify the renewal clause with local counsel. `[model knowledge — verify]`"

---

## Modes

### Mode 1: Ingest a renewal (handoff from review skill)

When `contract-review`, `nda-review`, or `vendor-agreement-review` finds an auto-renewal clause, it hands off a record. Append to register. If the counterparty already has an entry, ask: replacement (renewed agreement) or additional agreement?

When adding via `--add`: ask the user for each field. Pre-compute cancel_by_calendar, cancel_by_effective, send_by_effective. Show the computed dates and ask for confirmation before writing.

### Mode 2: What's coming up (default)

Default window: next 90 days. Urgency bands are half-open intervals:
- 🔴 **0–13 days** (send_by_effective in less than 14 days — including today)
- 🟠 **14–44 days**
- 🟡 **45–89 days**

Fire alerts off `send_by_effective`, not `cancel_by_effective`.

```markdown
## Renewals — next 90 days

### 🔴 Send notice within 0–13 days

| Counterparty | Send by | Cancel by | Renewal date | EUR/year | Owner | Notes |
|---|---|---|---|---|---|---|
| [name] | **[send_by]** | [cancel_by_effective] | [term_end] | [EUR] | [owner] | [notes] |

### 🟠 Send notice in 14–44 days

[same table]

### 🟡 Send notice in 45–89 days

[same table]

---

**Recommended actions:**
- [ ] [Counterparty] — ping [business_owner]: do we want to keep this?
- [ ] [Counterparty] — pricing is uncapped; get a quote from an alternative before losing leverage
- [ ] [Counterparty] — hiljainen uusiminen clause: ensure business owner is aware before [cancel_by_effective]
```

If register has more than ~10 renewals in window: offer a dashboard. Offer the sortable view with counts by urgency tier (🔴 / 🟠 / 🟡), cancel-by timeline, and EUR totals by band.

### Mode 3 (--days N): Change the window

Apply N days as the lookback horizon instead of 90. Same output format, same urgency bands.

### Mode 4 (--missed): Missed windows

```markdown
## Missed cancellation windows

| Counterparty | Cancel-by was | Renewal date | Status |
|---|---|---|---|
| [name] | [date] | [date] | Will auto-renew on [date] |

**Options:**
- Negotiate late cancellation (depends on vendor and relationship)
- Accept the renewal; mark next year's cancel-by now
- Check agreement for other termination rights (for convenience, for cause, change of control)
```

---

## Empty register

If `renewal-register.yaml` is missing or empty:

> "Your renewal register is empty. I can:
> 1. **Add a contract now** — tell me the counterparty, agreement name, renewal date, and notice period
> 2. **Walk through adding contracts** — I'll ask for each field one by one
> 3. **Accept handoffs from reviews** — run `/eu-legal:contract-review` or `/eu-legal:vendor-agreement-review` on a contract; if an auto-renewal clause is found, I'll add it to the register automatically"

---

## Gate: acting on a renewal

Tracking is research. Acting (sending a non-renewal notice, letting an auto-renewal fire, countersigning a renewal form) is a legal step with binding effect.

Before the user proceeds to act on a renewal:
> "Sending a termination / non-renewal notice or allowing an auto-renewal to fire past the cancel-by date has legal and contractual consequences. Have you confirmed the decision with the business owner and your legal team? If no, here's a brief to bring to them: [counterparty, term end, cancel-by date, what happens if no notice is given, next-period cost, and the three things to verify before the window closes]."

Do not proceed past this gate without an explicit yes.

---

## What this skill does not do

- It does not cancel contracts. It tells you when to decide.
- It does not decide whether to renew. It surfaces the deadline and the business owner.
- It does not read contracts to find renewal dates — that happens at review time via `contract-review` or `vendor-agreement-review`. If a contract is in the register without a computed cancel-by date, flag it as incomplete and ask the user to fill in the gap.
- It does not use Velvoite or require `VELVOITE_API_KEY`.
```

- [ ] **Step 3: Validate**

```bash
cat eu-legal/skills/renewal-tracker/SKILL.md | head -10
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/renewal-tracker/SKILL.md
git commit -m "feat(eu-legal): add renewal-tracker skill (Plan 3, Task 7)"
```

---

## Task 8: Final validation and skill count

- [ ] **Step 1: Verify all 7 skill directories exist**

```bash
ls eu-legal/skills/
```

Expected output includes: `commercial-cold-start`, `contract-review`, `nda-review`, `vendor-agreement-review`, `escalation-flagger`, `playbook-init`, `renewal-tracker` (plus the 7 from Plan 1 and however many from Plan 2).

- [ ] **Step 2: Check skill count**

```bash
ls eu-legal/skills/ | wc -l
```

Expected after Plan 3: **23 total** (7 from Plan 1 + 9 from Plan 2 + 7 from Plan 3 = 23).

> Note: if Plan 2 delivered a different count, adjust the expected total accordingly. The Plan 3 contribution is always 7 skills.

- [ ] **Step 3: Validate frontmatter on all 7 new skills**

```bash
for skill in commercial-cold-start contract-review nda-review vendor-agreement-review escalation-flagger playbook-init renewal-tracker; do
  echo "=== $skill ==="
  head -8 eu-legal/skills/$skill/SKILL.md
done
```

Confirm for each: `name` field matches directory name; `description` is non-empty; `argument-hint` is present.

- [ ] **Step 4: Verify MCP prefix on Velvoite calls**

```bash
grep -r "mcp__velvoite__" eu-legal/skills/contract-review/SKILL.md eu-legal/skills/vendor-agreement-review/SKILL.md
```

Confirm: only `mcp__velvoite__get_canonical_obligations` calls appear (no `mcp__plugin_eu-finreg_velvoite__` prefix, no bare `get_canonical_obligations`).

- [ ] **Step 5: Verify EUR throughout (no USD)**

```bash
grep -rn "USD\|\$[0-9]" eu-legal/skills/commercial-cold-start/SKILL.md eu-legal/skills/escalation-flagger/SKILL.md eu-legal/skills/renewal-tracker/SKILL.md
```

Expected: no matches (all amounts are in EUR).

- [ ] **Step 6: Verify no US-only references**

```bash
grep -rn "Delaware\|New York\|California\|AAA arbitration\|JAMS\|UTSA\|DTSA\|SOC 2" eu-legal/skills/
```

Expected: no matches in Plan 3 skills.

- [ ] **Step 7: Final summary commit**

```bash
git add eu-legal/skills/
git commit -m "feat(eu-legal): Plan 3 commercial skills complete — 7 skills added (commercial-cold-start, contract-review, nda-review, vendor-agreement-review, escalation-flagger, playbook-init, renewal-tracker)"
```

---

## Summary

| Task | Skill | Key EU differentiators |
|---|---|---|
| 1 | `commercial-cold-start` | EU governing law options (Finnish/German/English/ICC/SCC), DORA ICT register, EUR thresholds, EU-specific auto-triggers |
| 2 | `contract-review` | TSD 2016/943 confidentiality check, GDPR Art. 28 DPA gate, DORA Art. 30 ICT check via Velvoite or static checklist, non-EU governing law flag |
| 3 | `nda-review` | TSD Art. 2(1) CI definition, Arts. 3 & 4 carve-outs and prohibited use, Finnish Liikesalaisuuslaki + German GeschGehG citations, no US UTSA/DTSA analysis |
| 4 | `vendor-agreement-review` | Full DORA Art. 30 12-clause compliance table, concentration risk check against ICT register, Velvoite-backed or static fallback, EUR thresholds |
| 5 | `escalation-flagger` | EUR amounts throughout, EU auto-triggers (EEA data transfer, DORA ICT, non-EU governing law), no US-specific triggers |
| 6 | `playbook-init` | EU-sensible defaults (Finnish law / ICC-Stockholm), unfair terms Directive note for standard-form contracts, Finnish-jurisdiction auto-renewal awareness |
| 7 | `renewal-tracker` | Finnish hiljainen uusiminen explicit flag, EU business-day calendars (Finnish/German/English), EUR annual values, no Velvoite dependency |

**Expected skill count after Plan 3:** 16 (after Plan 2) + 7 = **23 skills**.
