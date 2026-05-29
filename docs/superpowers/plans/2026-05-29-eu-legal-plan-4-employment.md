# eu-legal Plugin — Plan 4: Employment Skills

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build 7 employment law skills for the eu-legal plugin covering Finnish and German employment law, EU directives, and key cross-border scenarios — from cold-start setup through termination review, works council consultation, leave entitlements, worker classification, international expansion, and policy drafting.

**Architecture:** Skills are flat under `eu-legal/skills/<name>/SKILL.md` with YAML frontmatter. All employment skills load `~/.claude/plugins/config/eu-legal/CLAUDE.md` (base profile) and `~/.claude/plugins/config/eu-legal/employment.md` (employment-specific config written by `employment-cold-start`). Skills degrade gracefully when `employment.md` is missing — they fall back to base profile context and general guidance. No Velvoite dependency for employment skills (employment law is national/directive-based, not corpus-backed).

**Tech stack:** Claude plugin format (SKILL.md with YAML frontmatter + markdown body).

**This plan covers:** `employment-cold-start`, `worker-classification`, `termination-review`, `works-council-check`, `leave-review`, `international-expansion`, `policy-drafting`.

**Expected skill count after this plan:** 23 (after Plan 3) + 7 = 30.

---

## Files created in this plan

```
eu-legal/skills/
  employment-cold-start/SKILL.md       ← NEW
  worker-classification/SKILL.md       ← NEW
  termination-review/SKILL.md          ← NEW
  works-council-check/SKILL.md         ← NEW
  leave-review/SKILL.md                ← NEW
  international-expansion/SKILL.md     ← NEW
  policy-drafting/SKILL.md             ← NEW
```

Config path written at runtime (not shipped in plugin):
```
~/.claude/plugins/config/eu-legal/employment.md   ← written by employment-cold-start
```

---

## Important rules for all tasks

- Frontmatter fields: `name`, `description`, `argument-hint` — nothing else.
- All skills load base config (`~/.claude/plugins/config/eu-legal/CLAUDE.md`) first, then `employment.md` if it exists.
- If base config has `[PLACEHOLDER]` markers, stop: "Run `/eu-legal:cold-start-interview` first."
- If `employment.md` is missing, note it and continue with general guidance (do not hard-stop except in `employment-cold-start` itself).
- No US law references: no FLSA, FMLA, EEOC, OWBPA, COBRA, W-2/1099, US state law, WARN Act (US version), at-will employment.
- All financial amounts in EUR.
- All termination/leave calculations use Finnish/German/EU law only.
- Every skill ends with a guardrail: output is for attorney review; employment law carries significant liability exposure.
- No `Co-Authored-By` in commits.
- Commit message style: `feat: add <skill-name> skill` or `feat: add employment skills batch`.

---

## Task 1: `employment-cold-start` skill

**Files:**
- Create: `eu-legal/skills/employment-cold-start/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/employment-cold-start
```

- [ ] **Step 2: Write `eu-legal/skills/employment-cold-start/SKILL.md`**

```markdown
---
name: employment-cold-start
description: >
  Set up the employment practice profile — learns your hiring countries,
  headcount, collective agreements, works council status, and termination
  policy, then writes the employment config. Run on first use of any
  employment skill, when employment.md is missing or has placeholders, or
  when your employment situation changes. Use when the user says "set up
  employment", "configure employment", "onboard employment", or runs any
  employment skill with a missing config.
argument-hint: "[--redo] to force a full re-run"
---

# /eu-legal:employment-cold-start

1. Check `~/.claude/plugins/config/eu-legal/CLAUDE.md` — if placeholders, stop: "Run `/eu-legal:cold-start-interview` first. Employment setup requires the base profile."
2. Check `~/.claude/plugins/config/eu-legal/employment.md` — if populated and no `--redo`, ask: "Employment config already exists. Re-run and overwrite it?" Stop if user says no.
3. Run the interview workflow below.
4. Write `~/.claude/plugins/config/eu-legal/employment.md`.
5. Show summary. Offer next steps.

---

## Interview workflow

Ask questions one topic per message. Never dump all questions at once.

### 1. Hiring countries

Ask:
> "Which countries do you hire employees in? Select all that apply."

Present as a numbered checklist:
1. Finland (FI)
2. Germany (DE)
3. Estonia (EE)
4. Sweden (SE)
5. Netherlands (NL)
6. Other EU member state(s) — specify which

Accept multiple selections. Save as a list of country codes. At least one country must be selected.

If "Other EU" selected: ask which countries (free text). Record them under `other_eu_countries`.

### 2. Headcount

Ask:
> "Approximately how many employees do you have in total? And can you break it down by country if different?"

Collect:
- `total_headcount`: integer
- `headcount_by_country`: dict, e.g. `{FI: 25, DE: 8}`

**Threshold checks (run silently, record in config):**
- Finland ≥ 20 employees → YT-laki cooperation obligation applies; set `fi_yt_laki_applies: true`
- Germany ≥ 5 employees → Betriebsrat can be formed; set `de_betriebsrat_possible: true`
- Germany > 10 employees + ≥ 6 months tenure → KSchG applies; set `de_kschg_applies: true`
- Total ≥ 50 → whistleblower reporting channel mandatory; set `whistleblower_channel_required: true`

### 3. Collective agreements

Ask (separately per country if multiple):

**Finland:**
> "Does a collective agreement (työehtosopimus, TES) cover your employees in Finland? If yes, which one? (e.g., technology industry TES — Teknologiateollisuus, financial services, general)"

Record: `fi_collective_agreement: [name or none]`

**Germany:**
> "Is your German workforce covered by a collective agreement (Tarifvertrag)? If yes, which one? (e.g., IG Metall, ver.di, employer association membership)"

Record: `de_tarifvertrag: [name or none]`

### 4. Standard employment contract terms

Ask:
> "What are your standard employment contract terms? I'll ask about three things."

Ask separately:
1. "Do you use the statutory minimum notice periods, or longer contractual notice periods in Finland/Germany?"
   - Accept: `statutory` or a description like "3 months both sides in Finland"
   - Record: `fi_notice_period_policy` and `de_notice_period_policy`

2. "Do you include non-compete clauses in employment contracts? If yes, for how long, and in which countries?"
   - Record: `noncompete_used: true/false`, `noncompete_duration_months: N`, `noncompete_countries: [list]`
   - Note internally: Finnish law caps at 12 months; compensation required if >6 months (TSL Ch. 3 §5)

3. "Do you include IP assignment / invention assignment clauses?"
   - Record: `ip_assignment: true/false`

### 5. Finland — personnel representation

If FI in hiring countries:
> "In Finland, has the YT-laki cooperation obligation threshold (≥20 employees) been reached? And do you have existing personnel representation — a luottamusmies (shop steward) or henkilöstöedustaja (personnel rep)?"

Record:
- `fi_yt_laki_active: true/false` (may differ from threshold check if mixed workforce)
- `fi_personnel_rep_type: luottamusmies | henkilostoedustaja | none`
- `fi_union_affiliated: true/false`

### 6. Germany — works council

If DE in hiring countries:
> "In Germany, does a Betriebsrat (works council) exist? If yes, how many members?"

Record:
- `de_betriebsrat_exists: true/false`
- `de_betriebsrat_members: N` (0 if none)

### 7. Termination policy

Ask:
> "Three quick questions about your termination practice."

Ask separately:
1. "When does legal review a termination? (All terminations / performance/conduct only / RIFs only / exec only)"
   - Record: `termination_review_trigger`

2. "Do you offer standard severance above statutory entitlements? If yes, what formula?"
   - Record: `standard_severance: [none / formula e.g. '1 month per year']`

3. "What's the outside counsel threshold for terminations — at what point do you escalate to external employment counsel?"
   - Record: `outside_counsel_threshold: [description]`

---

## Write the config

Write to `~/.claude/plugins/config/eu-legal/employment.md` (create parent dirs if needed).

Template to fill and write:

```markdown
# Employment Practice Profile
*Written by /eu-legal:employment-cold-start on [DATE]. Run with --redo to update.*

---

## Hiring countries

**Countries:** [comma-separated list]
**Total headcount:** [N]
**Headcount by country:** [FI: N, DE: N, ...]

---

## Threshold checks

| Check | Applies | Notes |
|---|---|---|
| YT-laki (FI ≥20 employees) | [true/false] | Cooperation procedure required before major changes and collective redundancies |
| Betriebsrat possible (DE ≥5 employees) | [true/false] | Works council can be formed; consultation rights apply once formed |
| KSchG applies (DE >10 employees) | [true/false] | Dismissal protection applies to employees with >6 months tenure |
| Whistleblower channel required (total ≥50) | [true/false] | Directive 2019/1937 and Finnish implementing act 1171/2022 |

---

## Collective agreements

**Finland TES:** [name or none]
**Germany Tarifvertrag:** [name or none]

---

## Standard contract terms

**Finland notice periods:** [statutory / contractual description]
**Germany notice periods:** [statutory / contractual description]
**Non-compete clauses used:** [true/false]
**Non-compete max duration:** [N months or N/A]
**Non-compete countries:** [list or N/A]
**IP assignment clauses:** [true/false]

---

## Finland — personnel representation

**YT-laki cooperation active:** [true/false]
**Personnel rep type:** [luottamusmies / henkilostoedustaja / none]
**Union affiliated:** [true/false]

---

## Germany — works council

**Betriebsrat exists:** [true/false]
**Betriebsrat members:** [N]

---

## Termination policy

**Legal reviews terminations:** [all / performance-conduct / RIFs / exec]
**Standard severance:** [none / formula]
**Outside counsel threshold:** [description]

---

*Re-run: `/eu-legal:employment-cold-start --redo`*
```

---

## Offer next steps

After writing the config, say:

> "Employment profile saved. Try:
> - `/eu-legal:works-council-check` — does this decision require works council or YT-laki consultation?
> - `/eu-legal:termination-review` — review a proposed termination for legal risk
> - `/eu-legal:leave-review` — calculate leave entitlements for an employee
> - `/eu-legal:worker-classification` — classify a contractor or gig worker arrangement"
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/employment-cold-start/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/employment-cold-start/SKILL.md
```

Expected: file exists, first line is `---`.

- [ ] **Step 4: Validate with claude plugin validate**

```bash
cd /Users/mattipuhakka/law-for-eu/eu-legal && claude plugin validate . 2>&1 | head -20
```

Expected: no errors for `employment-cold-start`.

- [ ] **Step 5: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/employment-cold-start/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add employment-cold-start skill"
```

---

## Task 2: `worker-classification` skill

**Files:**
- Create: `eu-legal/skills/worker-classification/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/worker-classification
```

- [ ] **Step 2: Write `eu-legal/skills/worker-classification/SKILL.md`**

```markdown
---
name: worker-classification
description: >
  Classify a worker as employee vs. independent contractor under EU, Finnish,
  and German law. Applies the Platform Work Directive (2024/2831) rebuttable
  presumption, TSL (Finland), and §611a BGB (Germany). Use when asked "is this
  person an employee or contractor?", "can we use a freelancer for this?",
  "contractor classification risk", or "platform worker classification".
argument-hint: "[describe the working arrangement or paste the contract]"
---

# /eu-legal:worker-classification

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/employment.md` if it exists. Note hiring countries and KSchG/YT-laki thresholds.
3. Run the workflow below.
4. Output: classification conclusion, risk level, and recommended action.

---

## Purpose

Worker misclassification is a high-exposure risk: tax authorities (Verohallinto in Finland, Finanzamt in Germany), social security bodies, and labour courts all apply their own tests — and the consequences (back taxes, social contributions, fines, retroactive employment rights) are significant. This skill applies the legal tests; a qualified lawyer must review before acting on the conclusion.

---

## Step 1: Gather facts

Ask the user to describe the arrangement. If not already provided, ask:

1. "Who sets the remuneration (rate, schedule, amount) — the hiring company or the worker themselves?"
2. "Is work performance monitored electronically, or are working hours/output tracked by the company's systems?"
3. "Does the company give instructions about how to perform the work — conduct, dress code, tools, methodology?"
4. "Can the worker build their own client base, work for competitors, or subcontract the work to others?"
5. "Does the worker use their own tools and equipment, or the company's?"
6. "Is the worker economically dependent on this one company (>70% of income)?"
7. "Is the worker integrated into the company's organisation — regular team meetings, company email, company systems?"
8. "What jurisdiction(s) are relevant? (Country where work is performed, country of contract governing law)"

Accept answers in free text or as a summary of an existing contract.

---

## Step 2: Apply Platform Work Directive test (EU baseline)

**Directive 2024/2831 Art. 4 — applies where platform work is involved.**

Five indicators. Count how many are present:

| Indicator | Present? |
|---|---|
| (1) Remuneration set or capped by platform/company | [Y/N/Partial] |
| (2) Work performance supervised electronically (real-time tracking, rating systems, algorithmic management) | [Y/N/Partial] |
| (3) Instructions on conduct, appearance, or how to perform the work | [Y/N/Partial] |
| (4) No freedom to build own client base, work for third parties, or subcontract | [Y/N/Partial] |
| (5) No freedom to organise own working time or accept/reject tasks | [Y/N/Partial] |

**Result:**
- **2 or more indicators present** → rebuttable presumption of employment relationship. Employer must rebut presumption (burden is reversed). Flag as 🔴 High risk if presumption applies.
- **1 indicator** → no presumption, but note the indicator(s) present.
- **0 indicators** → Platform Work Directive presumption does not apply.

Note: even if the worker is not on a "platform", courts in FI and DE may apply similar logic (economic dependency, integration). Apply the indicators as a checklist regardless.

---

## Step 3: Apply Finnish test (TSL §1)

**Employment relationship exists under Työsopimuslaki if ALL THREE elements are present:**

| Element | Analysis |
|---|---|
| (1) Work performed for the employer | [describe: is output for employer's benefit?] |
| (2) Under employer's direction and supervision (johto ja valvonta) | [describe: does employer direct how/when work is done?] |
| (3) For remuneration | [describe: is payment for the work, not for a result?] |

Finnish courts also weigh:
- **Economic dependency**: >70% income from one client → strong indicator
- **Integration into organisation**: company email, access to systems, participates in company meetings
- **Tools and equipment**: company-provided → indicator of employment
- **Exclusivity or practical exclusivity**: cannot work for others in practice

**Conclusion (Finland):** Employee / Likely employee / Likely contractor / Contractor — with confidence level.

---

## Step 4: Apply German test (§611a BGB)

**Employee is personally dependent (persönliche Abhängigkeit):**

| Factor | Analysis |
|---|---|
| Follows instructions (weisungsgebunden) — on time, place, content | [Y/N/Partial] |
| Integrated into company organisation (eingliederung) — fixed schedule, company hierarchy | [Y/N/Partial] |
| No entrepreneurial risk — paid regardless of business outcome | [Y/N/Partial] |
| No own client base — works exclusively or predominantly for this one company | [Y/N/Partial] |

If the formal contract says "contractor" but the factual conduct shows the above, German courts treat the relationship as employment regardless of label. This is *faktisches Arbeitsverhältnis* — a de facto employment relationship.

**KSchG note:** If `de_kschg_applies: true` from profile and this worker would be classified as employee, KSchG dismissal protection applies after 6 months.

**Conclusion (Germany):** Employee / Likely employee / Likely contractor / Contractor — with confidence level.

---

## Step 5: Overall risk assessment

Synthesise across all applicable tests:

| Test | Conclusion | Risk |
|---|---|---|
| Platform Work Directive | [Employee presumption / No presumption] | [🔴/🟡/🟢] |
| Finland (TSL) | [if FI applicable] | [🔴/🟡/🟢] |
| Germany (§611a BGB) | [if DE applicable] | [🔴/🟡/🟢] |

**Overall risk:**
- 🔴 **High** — presumption of employment applies OR two or more tests conclude "employee" or "likely employee". Recommend converting to employment contract before engaging or continuing.
- 🟡 **Medium** — one test concludes "likely employee" or indicators are mixed. Legal review required before relying on contractor status.
- 🟢 **Low** — all applicable tests conclude "contractor" with confidence. Document the analysis.

---

## Step 6: Recommended action

Based on risk level:

**🔴 High:**
> "Based on this analysis, there is a significant risk that this arrangement will be treated as an employment relationship — potentially with retroactive effect. Consequences include back social contributions (pension, unemployment, health insurance), income tax liability for the company, and the full suite of employment rights (notice periods, protection against dismissal, leave entitlements). We strongly recommend: (1) do not start or continue this arrangement as a contractor relationship; (2) either convert to employment or fundamentally restructure the arrangement so the worker has genuine independence; (3) obtain advice from an employment lawyer before proceeding."

**🟡 Medium:**
> "The classification is uncertain. The arrangement has characteristics of both employment and contracting. Before relying on contractor status: obtain a written legal opinion from an employment lawyer familiar with [jurisdiction]; document the independence factors (multiple clients, own tools, right to subcontract, no exclusivity); review any existing contract to ensure it reflects actual practice."

**🟢 Low:**
> "The arrangement appears consistent with independent contracting under the applicable tests. Document this analysis. Review annually or whenever the arrangement materially changes — classification can shift if the company increases control or the worker's economic dependency increases."

---

## Guardrail

> **RESEARCH NOTES — NOT LEGAL ADVICE.** Worker classification has significant tax, social security, and employment law consequences. This analysis applies legal tests to the facts as described — it is a working document for attorney review, not a classification opinion. A qualified employment lawyer in the relevant jurisdiction must review before you rely on contractor status or convert an existing arrangement. Misclassification can result in back payments of social contributions, income tax liability, and retroactive employment rights.
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/worker-classification/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/worker-classification/SKILL.md
```

Expected: file exists, first line is `---`.

- [ ] **Step 4: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/worker-classification/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add worker-classification skill"
```

---

## Task 3: `termination-review` skill

**Files:**
- Create: `eu-legal/skills/termination-review/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/termination-review
```

- [ ] **Step 2: Write `eu-legal/skills/termination-review/SKILL.md`**

```markdown
---
name: termination-review
description: >
  Review a proposed termination for legal risk — jurisdiction-specific
  grounds analysis, notice period calculation, protected status flags, and
  required procedural steps. Covers Finland (TSL, YT-laki) and Germany
  (KSchG, BetrVG). Use when asked "can we fire this person", "reviewing a
  termination", "term review", "is this a valid dismissal", or when
  describing a termination scenario.
argument-hint: "[describe the termination — role, reason, tenure, jurisdiction]"
---

# /eu-legal:termination-review

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/employment.md` if it exists. Read: termination review trigger, standard severance, outside counsel threshold.
3. Run the workflow below.
4. Output: risk assessment, required steps checklist, notice period, protected status flags.

---

## Purpose

Most terminations are legally sound. A few carry serious risk — wrongful dismissal claims, compensation orders, reinstatement orders, works council consultation failures that void the dismissal entirely. This skill identifies the risk; a qualified employment lawyer must review before the dismissal is communicated.

---

## Step 1: Gather facts

If not already provided in the argument, ask:

1. "Which country is this employment relationship governed by?" (Finland / Germany / other — if other, note that coverage is limited)
2. "What is the reason for the termination?" (misconduct / performance / redundancy / restructuring / other)
3. "How long has the employee been employed?" (years and months)
4. "What is the employee's role and seniority?"
5. "Any of these apply? (check all that apply): pregnant or on maternity leave / on parental leave / personnel representative or shop steward / recently reported a workplace concern or filed a complaint / on sick leave currently / member of works council (Germany)"

---

## Step 2: Jurisdiction check

Route to the applicable framework below. If both FI and DE apply (e.g., employee works across borders), apply both and note the more protective.

---

## Finland framework (Työsopimuslaki, TSL 55/2001)

### Grounds check

**Individual grounds (henkilöön liittyvä peruste, TSL Ch. 7 §2):**

Applies where the reason is the employee's conduct, performance, or personal circumstances.

Required elements — all must be satisfied:
- [ ] Substantial reason (asiallinen ja painava syy) — the conduct or incapacity is serious enough to justify dismissal
- [ ] Warning given (varoitus) — for conduct dismissals: has the employee received a written warning about this specific behaviour and been given the opportunity to correct it? Exception: conduct so serious that warning is not required (e.g., criminal conduct, gross breach of trust)
- [ ] Proportionality — is dismissal proportionate, or would a less severe measure (warning, demotion, transfer) suffice?
- [ ] No obligation to offer alternative work — has the employer explored reassignment?

Note whether each element is met, partially met, or not met. Any unmet element is a 🔴 risk.

**Financial/production grounds (taloudellinen tai tuotannollinen irtisanomisperuste, TSL Ch. 7 §3):**

Applies where the reason is a genuine reduction in the need for the employee's work.

Required elements:
- [ ] Genuine reduction in work — is the business reason real? (Not personal performance dressed up as redundancy)
- [ ] No suitable alternative work — employer must check: is there any other role this employee could fill, including after reasonable retraining?
- [ ] Re-employment obligation — TSL Ch. 6 §6: if employer rehires within 9 months of notice, former employee has re-employment priority
- [ ] YT-laki check — if ≥10 redundancies in 90 days: cooperation negotiations (YT-neuvottelut) required BEFORE notice. See works-council-check skill.

### Notice period calculation (TSL Ch. 6 §3)

**Employer-side notice periods by service length:**

| Service length | Employer notice period |
|---|---|
| < 1 year | 14 days |
| 1–4 years | 1 month |
| 4–8 years | 2 months |
| 8–12 years | 3 months |
| 12–15 years | 4 months |
| > 15 years | 6 months |

**Employee-side notice periods:**

| Service length | Employee notice period |
|---|---|
| < 5 years | 14 days |
| ≥ 5 years | 1 month |

Calculate and state: "Based on [N years] service, the employer's notice period is [X]. Last day of employment if notice given today ([DATE]): [DATE + notice period]."

Note: collective agreement (TES) may prescribe longer notice periods — check `fi_collective_agreement` from employment.md. If a TES applies and its terms are not known, flag: "Verify notice period against applicable TES `[review]`."

### Protected employees — Finland

Flag if any apply. Each requires heightened justification; some create near-absolute protection:

| Status | Protection |
|---|---|
| Pregnant (raskaus) | TSL Ch. 7 §9: dismissal during pregnancy presumed connected to pregnancy; extremely high risk |
| Maternity / parental leave (äitiys-, isyys-, vanhempainvapaa) | Cannot dismiss due to family leave; burden on employer to prove unconnected |
| Luottamusmies (shop steward) | TSL Ch. 7 §10: can only dismiss with consent of the majority of the workers they represent, or if work has entirely ceased |
| Henkilöstöedustaja (personnel rep) | Same heightened protection as luottamusmies |
| Current sick leave | Not absolute protection but timing creates strong inference of retaliation — document carefully |

---

## Germany framework (KSchG + BetrVG)

### KSchG applicability check

KSchG (Kündigungsschutzgesetz) applies if:
- Establishment has **more than 10 employees** (employees counted in full/part-time equivalents — part-time ≤20h/week counts as 0.5), AND
- Employee has been employed for **more than 6 months** (Wartezeit)

If KSchG does NOT apply: dismissal is freer but still subject to AGG (anti-discrimination) and good faith. Note this explicitly.

If `de_kschg_applies: true` from profile, proceed with full KSchG analysis.

### Grounds check (KSchG §1)

Three valid grounds — must fit at least one:

**Person-related (personenbedingt):**
- Long-term incapacity or frequent short-term illness where no improvement is expected AND continued employment unreasonable — negative prognosis required
- [ ] Is there a documented negative prognosis (arztliches Attest)?
- [ ] Has employer explored all reasonable adjustments (§164 SGB IX for disabilities)?

**Conduct-related (verhaltensbedingt):**
- Deliberate breach of contractual obligations
- [ ] Has a prior warning (Abmahnung) been given for the same type of conduct? (Required except for the most serious breaches)
- [ ] Is the conduct continuing or was it a one-off?

**Operational (betriebsbedingt):**
- Genuine operational decision removing or reducing this type of work
- [ ] Is the business reason genuine and not a pretext?
- [ ] Social selection (Sozialauswahl, KSchG §1(3)): among comparable employees who could have been dismissed, were the correct criteria applied? (Age, tenure, dependants, disabilities — employee with weakest social score dismissed first)
- [ ] Has employer checked for alternative positions?

### Works council consultation (BetrVG §102) — MANDATORY

**Before every dismissal** (individual or collective), the Betriebsrat must be consulted. This is not optional.

If `de_betriebsrat_exists: true` from employment.md:
- [ ] Has written notification been given to the Betriebsrat with: name of employee, type of dismissal (ordinary/extraordinary), timing, reason?
- [ ] Has the consultation period elapsed? (Ordinary dismissal: 1 week; Extraordinary §626 dismissal: 3 days)
- [ ] If Betriebsrat objects: employer may still dismiss but must notify employee of the objection. Employee may seek interim injunction (Weiterbeschäftigung).

**Failure to consult = dismissal is void (nichtig).** This is the single most common reason German dismissals fail. Flag as 🔴 blocking if consultation has not been completed.

If no Betriebsrat exists: note this, but flag that if one is later formed, prior dismissals without consultation are not retroactively void.

### Notice periods (KSchG / §622 BGB)

**Statutory employer notice periods by service length:**

| Service length | Employer notice period |
|---|---|
| < 2 years | 4 weeks (to 15th or end of month) |
| 2–5 years | 1 month (to end of month) |
| 5–8 years | 2 months (to end of month) |
| 8–10 years | 3 months (to end of month) |
| 10–12 years | 4 months (to end of month) |
| 12–15 years | 5 months (to end of month) |
| 15–20 years | 6 months (to end of month) |
| ≥ 20 years | 7 months (to end of month) |

Calculate and state: "Based on [N years] service, the employer's notice period is [X months], running to end of month. Last day of employment if notice given today ([DATE]): [END OF MONTH + N MONTHS]."

### Protected employees — Germany

| Status | Protection |
|---|---|
| Pregnant (§17 MuSchG) | Dismissal void during pregnancy and 4 months post-birth; requires authority approval (Gewerbeaufsicht) |
| Parental leave (§18 BEEG) | Dismissal protection throughout parental leave period (up to 3 years); authority approval required |
| Betriebsrat member | Special protection: only collective dismissal with Betriebsrat consent, or extraordinary dismissal for serious cause |
| Severely disabled (GdB ≥50) | Requires Integrationsamt approval before dismissal |
| Maternity leave (MuSchG) | Same as pregnancy — void without authority approval |

---

## Step 3: Output

### Risk assessment

State overall risk:

- 🔴 **High** — grounds are weak or absent, protected status present, required consultation not done, or warning not given. Do not proceed without legal review.
- 🟠 **Medium-High** — one or more required elements uncertain; needs legal input before proceeding.
- 🟡 **Medium** — grounds appear sound but procedural steps remain; proceed carefully through checklist.
- 🟢 **Low** — grounds clear, procedure followed, no protected status. Still recommend attorney sign-off.

### Required steps checklist

Output as a checklist of what must happen BEFORE notice is given:

- [ ] [Step 1: e.g., "Obtain written warning file — verify warning was given for [specific conduct] and acknowledged"]
- [ ] [Step 2: e.g., "BetrVG §102 consultation — notify Betriebsrat in writing with grounds, wait 1 week"]
- [ ] [Step 3: e.g., "Verify no alternative positions available — document search"]
- [ ] [Step 4: e.g., "Check TES for any enhanced notice period or procedural requirements"]
- [ ] [Step 5: "Attorney sign-off before notice communicated"]

### Outside counsel escalation

If `outside_counsel_threshold` is set in employment.md: compare the facts against the threshold. If met, say: "Your employment profile sets outside counsel escalation when [threshold]. This termination meets that threshold — escalate before proceeding."

---

## Guardrail

> **CONFIDENTIAL — INTERNAL LEGAL ANALYSIS — NOT A SUBSTITUTE FOR EXTERNAL COUNSEL ADVICE.** Wrongful dismissal claims carry significant financial exposure: compensation up to [24 months salary in Finland (TSL Ch. 12 §2)] or [12 months + Sozialplan entitlements in Germany], plus reinstatement orders. This analysis identifies risk based on the facts as described — it is a working document for attorney review, not a legal opinion. An employment lawyer must review before the dismissal is communicated.
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/termination-review/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/termination-review/SKILL.md
```

Expected: file exists, first line is `---`.

- [ ] **Step 4: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/termination-review/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add termination-review skill"
```

---

## Task 4: `works-council-check` skill

**Files:**
- Create: `eu-legal/skills/works-council-check/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/works-council-check
```

- [ ] **Step 2: Write `eu-legal/skills/works-council-check/SKILL.md`**

```markdown
---
name: works-council-check
description: >
  Check whether a planned decision requires works council or employee
  representative body consultation — YT-laki in Finland, BetrVG in Germany,
  and EU collective redundancy rules. Use when asking "do we need to consult
  the works council?", "does YT-laki apply here?", "can we restructure without
  consultation?", "collective redundancy procedure", or before any major
  workforce or organisational change.
argument-hint: "[describe the decision or change — e.g. 'closing the Helsinki office', 'removing 15 roles', 'changing remote work policy']"
---

# /eu-legal:works-council-check

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/employment.md` if it exists. Read: headcount thresholds, fi_yt_laki_applies, fi_personnel_rep_type, de_betriebsrat_exists, hiring countries.
3. Run the workflow below.
4. Output: consultation required (yes/no/recommended), which body, what procedure, timeline, consequence of skipping.

---

## Purpose

Consultation failures are among the most expensive employment law mistakes in Finland and Germany. In Germany, a dismissal without BetrVG §102 consultation is void — the employer must reinstate the employee and pay back salary regardless of the substantive grounds. In Finland, breach of YT-laki cooperation obligations triggers employer liability. This skill determines whether consultation is required before you act.

---

## Step 1: Identify the decision

If not already described, ask:
> "What decision or change are you considering? (e.g., redundancies, office closure, outsourcing, change to working hours, implementation of monitoring technology, change to remote work policy)"

Also ask:
> "How many employees are affected, and in which countries?"

---

## Step 2: Route by jurisdiction

### Finland — YT-laki 1333/2021

**Threshold check:**
If `fi_yt_laki_applies: false` (fewer than 20 employees regularly), YT-laki does not apply. Note: even below 20, employer has good faith consultation obligations under general employment law principles.

If `fi_yt_laki_applies: true`, check the decision type:

**Decisions requiring YT-neuvottelut (cooperation procedure):**

| Decision type | YT-laki provision | Minimum negotiation period |
|---|---|---|
| Collective redundancies (≥10 employees in 90 days) | §51 | 6 weeks (employers ≥30 employees); 6 weeks or shorter period agreed with personnel rep |
| Significant changes to work or working conditions | §16 | Negotiate until agreement or until parties find no agreement possible |
| Outsourcing or acquisition affecting personnel | §16 | Negotiate to completion |
| Changes to use of fixed-term or part-time employment | §16 | Negotiate to completion |
| Introduction of surveillance or monitoring systems | §16 | Negotiate to completion |
| Changes to remote work practices (significant change) | §16 | Negotiate to completion |
| Reduction of fewer than 10 employees (financial/production grounds) | §47 | Recommend consultation even if below threshold |

**Who to consult:**
- If `fi_personnel_rep_type: luottamusmies` → consult the luottamusmies (shop steward)
- If `fi_personnel_rep_type: henkilostoedustaja` → consult the henkilöstöedustaja (personnel rep)
- If `fi_personnel_rep_type: none` → consult directly with the employees affected; document the process

**What the procedure requires:**
1. Employer gives written notice (neuvottelukutsu) to personnel rep stating: what is being considered, why, and what alternatives have been considered
2. Negotiations held in good faith — employer must provide necessary information; employee rep can request additional information
3. Employer must genuinely consider alternatives and employee proposals
4. Employer documents that negotiations occurred and what was discussed
5. After negotiations conclude: employer issues decision; personnel rep has right to be informed of the outcome

**Consequence of skipping:**
- Employer liable for compensation up to 35 days' salary per affected employee (YT-laki §62)
- For collective redundancies: does not invalidate the dismissal, but creates significant liability exposure
- Criminal liability (fine) for employer's responsible person in serious cases (YT-laki §62 §3)

---

### Germany — BetrVG

**Threshold check:**
If `de_betriebsrat_exists: false` from employment.md: no Betriebsrat consultation required. Note: if ≥5 employees, employees have the right to form one — if they do, consultation requirements apply from that point.

If `de_betriebsrat_exists: true`, determine the BetrVG provision:

**Decisions requiring mandatory consultation or co-determination:**

| Decision type | BetrVG provision | Consequence of skipping |
|---|---|---|
| Individual dismissal (ordinary or extraordinary) | §102 | Dismissal is void (nichtig) |
| Mass redundancy (≥20 employees or thresholds below) | §112 + §111 | Interest of balance of interests (Interessenausgleich) required; Sozialplan mandatory; failure = compensation liability |
| Change to working hours, breaks, or overtime | §87(1) No. 2+3 | Co-determination right — measure cannot be implemented without agreement or arbitration board ruling |
| Introduction of technical monitoring/surveillance devices | §87(1) No. 6 | Co-determination right — cannot implement without Betriebsvereinbarung or arbitration |
| Hiring / internal transfer / reclassification | §99 | Cannot implement without Betriebsrat consent or Labour Court order |
| Changes to pay structure or bonuses | §87(1) No. 10+11 | Co-determination right |
| Major operational change (Betriebsänderung) affecting >20 employees | §111 | Balance of interests required; Sozialplan mandatory; failure = §113 compensation per affected employee |
| Introduction of AI-based performance monitoring or scoring | §87(1) No. 6 + emerging case law | Co-determination right; proceed only with Betriebsvereinbarung |

**Major operational change (§111) threshold:**
Applies if establishment has >20 employees AND the change is: closure or relocation of the establishment or significant parts; merger; major changes to organisation, purpose, or equipment; introduction of new working methods or production processes with material effects on workforce.

**Sozialplan requirement:**
When §111 applies and the employer has >20 employees: a Sozialplan (social plan — compensation for affected employees) must be negotiated with the Betriebsrat. If no agreement: either party may go to the arbitration board (Einigungsstelle). The Sozialplan has force of a works agreement (Betriebsvereinbarung).

**§102 consultation procedure:**
1. Employer gives written notice to Betriebsrat: employee name, type and timing of dismissal, grounds
2. Betriebsrat has 1 week to respond (ordinary dismissal) or 3 days (extraordinary dismissal)
3. If Betriebsrat objects: must state grounds; employer must inform employee of objection
4. Employer may dismiss after period elapses even without Betriebsrat agreement
5. **Notice given before period elapses = void dismissal.**

---

### EU — Collective redundancy thresholds (Directive 98/59/EC)

EU collective redundancy rules apply where national law implements the Directive (all EU member states). Thresholds triggering special procedure:

| Establishment size | Redundancy threshold | Consultation period |
|---|---|---|
| 20–99 employees | ≥ 10 redundancies in 30 days | Minimum 30 days advance notice to authority |
| ≥ 100 employees | ≥ 10% of workforce OR ≥ 30 in 30 days | Minimum 30 days advance notice to authority |
| ≥ 300 employees | ≥ 30 in 30 days | Minimum 30 days advance notice to authority |

**Finland implementing act:** TSL §6 + YT-laki §51 — notification to TE-toimisto (Employment and Economic Development Office) required when ≥10 redundancies within 90 days.

**Germany implementing act:** KSchG §§ 17-18 — notification to Bundesagentur für Arbeit required before collective dismissals. Failure: dismissals void.

---

## Step 3: Output

**Consultation required:** [Yes — mandatory / Yes — recommended / No — but document decision / Uncertain — legal review needed]

**Which body:** [Betriebsrat / Luottamusmies / Henkilöstöedustaja / Employees directly / None / Authority notification also required]

**Applicable provision:** [BetrVG §102 / YT-laki §51 / YT-laki §16 / EU Directive 98/59 / KSchG §17]

**Minimum procedure:**
[Numbered list of required steps with sequence and timing]

**Timeline:**
[Earliest date the decision/dismissal can be implemented given notice and consultation periods]

**If skipped:**
[Specific consequence — void dismissal / compensation liability / criminal fine / authority penalty]

---

## Guardrail

> **RESEARCH NOTES — NOT LEGAL ADVICE.** Consultation requirements are jurisdiction-specific and fact-sensitive. A void dismissal (Germany) or compensation liability (Finland) can result from procedural failures even when the substantive grounds are sound. Have an employment lawyer confirm the consultation procedure and timing before the decision is communicated to employees.
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/works-council-check/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/works-council-check/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/works-council-check/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add works-council-check skill"
```

---

## Task 5: `leave-review` skill

**Files:**
- Create: `eu-legal/skills/leave-review/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/leave-review
```

- [ ] **Step 2: Write `eu-legal/skills/leave-review/SKILL.md`**

```markdown
---
name: leave-review
description: >
  Calculate leave entitlements and employer obligations for annual holiday,
  parental leave, maternity/paternity leave, and sick leave under Finnish,
  German, and EU law. Use when asked "how much holiday does this employee get?",
  "maternity leave rules in Finland", "German parental leave", "sick leave pay
  obligation", "can I recall someone from parental leave", or any leave
  entitlement question.
argument-hint: "[leave type] [jurisdiction] [employee tenure and status] — e.g. 'annual leave Finland 3 years' or 'parental leave Germany'"
---

# /eu-legal:leave-review

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/employment.md` if it exists. Read: hiring countries, fi_collective_agreement, de_tarifvertrag.
3. Identify leave type and jurisdiction from the argument.
4. Run the applicable framework below.
5. Output: entitlement, calculation, employer obligations, document requirements.

---

## Purpose

Leave entitlements under Finnish and German law exceed EU directive minimums significantly. The interaction with collective agreements (TES/Tarifvertrag) and individual contract terms can make the actual entitlement higher still. This skill calculates the floor; a lawyer or HR professional must verify against the applicable collective agreement.

---

## Step 1: Identify leave type

From the argument or by asking:
> "What type of leave are we looking at? (Annual holiday / Maternity leave / Paternity leave / Parental leave / Sick leave / Childcare leave / Other)"

Also confirm: jurisdiction (Finland / Germany / other EU) and employee details (tenure, whether collective agreement applies, full-time or part-time).

---

## Annual Holiday

### EU baseline (Working Time Directive 2003/88/EC Art. 7)
- **Minimum:** 4 weeks (20 working days) per year
- Cannot be replaced by pay-in-lieu except on termination
- Carried forward: national law rules; many member states allow limited carry-forward

### Finland — Vuosilomalaki (162/2005)

**Accrual rate:**
- First holiday year (April 1–March 31): 2 days per month employed
- After 1 full holiday year of employment (before end of first March 31 in employment): 2.5 days per month
- **2.5 days/month = 30 days = 6 weeks** after completing 1 full year of employment (holiday year runs April 1–March 31)

**Calculation:** multiply months worked in holiday year × accrual rate (2 or 2.5). Round up to whole days.

Example: employee has 3 years tenure, worked all 12 months of the holiday year → 12 × 2.5 = 30 days holiday entitlement.

**Holiday pay:**
- Salaried employees: full salary continues during holiday
- Hourly employees: holiday pay = 9% of earnings (≥1 year employment: 11.5%) — Vuosilomalaki §12

**Holiday bonus (lomakorvaus / lomaraha):** 
Not a statutory minimum but standard in most collective agreements (TES): typically 50% of holiday pay as a bonus. If `fi_collective_agreement` is set in employment.md: note that the applicable TES likely includes lomaraha — verify with the TES text.

**Employer obligations:**
- Give holiday during the main holiday season (June 2–September 30) unless otherwise agreed
- Give notice of holiday timing at least 1 month before
- Cannot force all holiday below 24 days to be taken outside main season without agreement

### Germany — Bundesurlaubsgesetz (BUrlG)

**Minimum:** 20 working days per year (5-day week)
**Standard in practice:** 25–30 days — most collective agreements and individual contracts exceed the statutory minimum

**Full entitlement vests after:** 6 months employment (§4 BUrlG). Pro-rated in first 6 months: 1/12 per month.

**Calculation:** if contract specifies 25 days and employee has worked 6 months → full 25 days entitlement.

**Holiday pay:** full salary during holiday (§1 BUrlG)

**Carry-forward:** holiday not taken by year-end carries to March 31 of following year (§7(3) BUrlG). After March 31: lapses unless individual contract or TarV says otherwise. Note: CJEU Kreuziger/Max-Planck case — employer must actively request employee to take holiday or entitlement cannot lapse.

If `de_tarifvertrag` is set in employment.md: note that Tarifvertrag likely specifies higher entitlement — verify.

---

## Maternity Leave

### EU baseline (Pregnant Workers Directive 92/85/EEC)
- **Minimum:** 14 weeks maternity leave, at least 6 weeks must be post-birth
- Pay: at least sick pay level or equivalent

### Finland — äitiysvapaa

**Duration:** 40 working days (äitiysvapaa, TSL + sickness insurance act)
- Begins typically 30 working days before expected due date
- Can begin up to 50 working days before due date

**Pay:**
- Kela pays äitiysraha (maternity allowance) for the 40-day period
- Many TES require employer to top up to full salary — check `fi_collective_agreement`

**Employer obligations:**
- Cannot terminate employee due to pregnancy (TSL Ch. 7 §9) — void dismissal
- Must keep position open or offer equivalent role on return
- Employee must notify at least 2 months before leave starts

### Germany — Mutterschutz (MuSchG)

**Duration:**
- 6 weeks before expected due date (Mutterschutzfrist vor der Geburt) — cannot waive
- 8 weeks after birth (Mutterschutzfrist nach der Geburt); 12 weeks for premature birth or multiple birth

**Pay during Mutterschutz:**
- Employer pays Mutterschaftsgeld advance: daily pay up to €13/day
- Kasse (health insurer) pays the rest up to full net salary
- Net salary continues in full (§18 MuSchG)

**Dismissal protection:**
- §17 MuSchG: dismissal void during pregnancy and 4 months post-birth
- Employer must apply to Gewerbeaufsichtsamt for exceptional approval — rarely granted

---

## Paternity / Second-Parent Leave

### EU baseline (Parental Leave Directive 2019/1158 Art. 4)
- **Minimum:** 10 working days paternity leave around the time of birth
- Must be paid at sick pay level at minimum

### Finland — isyysvapaa

**Duration:** up to 54 working days, flexible timing within 2 years of birth
- New 2023 parental leave model: each parent has 160 days (kuukausittainen vanhempainraha). The old isyysvapaa concept merged into the new equal parental leave model.

**Pay:** Kela pays isyysraha at 70–90% of income. Many TES top up.

### Germany — Elternzeit (BEEG)

Paternity leave as a distinct concept is not mandatory in Germany beyond the EU minimum. The primary mechanism is Elternzeit (parental leave) — see below.

**Elterngeld:** payable to either parent; father can take leave simultaneously with mother or after.

---

## Parental Leave

### EU baseline (Parental Leave Directive 2019/1158 Art. 5)
- **Minimum:** 4 months per parent, of which 2 months are non-transferable
- Member states may make payment conditional on length of service (max 1 year)

### Finland — vanhempainvapaa (new 2023 model, Kela reform effective 1.8.2022)

**Duration per parent:** 160 working days (approximately 6.5 months) each
- Non-transferable: cannot give days to the other parent (equal model)
- Can be taken in segments; full flexibility within 2 years of birth
- Single parents: entitled to both parents' quota (320 days)

**Pay:** Kela pays vanhempainraha at 70–90% of earnings. Many TES top up for first portion.

**Hoitovapaa (childcare leave):** after vanhempainvapaa ends, parent may take unpaid leave until child turns 3 years old. Job protection: employer must offer the same or equivalent role on return.

**Employer obligations:**
- Cannot terminate employee on grounds of family leave (TSL)
- Must keep position open or offer equivalent role
- Employee must give 2 months notice of leave and 1 month notice of return

### Germany — Elternzeit (BEEG)

**Duration:** up to 3 years per child (each parent independently)
- Can be split into up to 3 periods before child's 8th birthday (§16 BEEG — 2015 reform)
- Both parents can take Elternzeit simultaneously
- Can work part-time up to 32h/week during Elternzeit (§15(4) BEEG)

**Pay (Elterngeld):**
- Basic Elterngeld (Basiselterngeld): 65–100% of net income, max €1,800/month, for 12 months (14 months if both parents take at least 2 months each)
- ElterngeldPlus: halved monthly amount over double the period
- Partnerschaftsbonus: extra 4 months each if both parents work 25–32h/week simultaneously

**Job protection:**
- §18 BEEG: employer cannot dismiss during Elternzeit; requires Gewerbeaufsicht approval
- Return right: same or equivalent position

**Employer obligations:**
- Employee must notify Elternzeit at least 7 weeks before it begins (§16 BEEG)
- Employer cannot unilaterally reduce notice period or change terms

---

## Sick Leave

### EU baseline
No EU minimum sick leave entitlement. National law governs.

### Finland

**Employer obligation (TSL Ch. 2 §11):**
- Full salary for up to **9 working days** per illness episode (depending on collective agreement — up to 28 days in many TES)
- Statutory minimum: full salary for illness duration up to 1 month if employed ≥1 month; shorter periods for shorter tenure

**After employer sick pay period:**
- Kela pays sairauspäiväraha (sickness allowance) from day 2 of illness (omavastuu day 1): approximately 70% of income, up to 300 working days

**Employer obligations:**
- Request sickness certificate (lääkärintodistus) — after how many days is negotiated in TES or company policy; statutory: after 3 days
- Cannot terminate employment solely due to illness without examining redeployment possibilities

### Germany — EFZG (Entgeltfortzahlungsgesetz)

**Employer obligation:**
- Full salary (Lohnfortzahlung) for up to **6 weeks** (42 calendar days) per illness, per new illness episode
- "New episode" if employee has been healthy for >6 months between episodes of the same illness

**After 6 weeks:**
- Krankengeld from health insurer (Krankenkasse): 70% of gross salary (up to Beitragsbemessungsgrenze), for up to 78 weeks within a rolling 3-year period for the same illness

**Employer obligations:**
- After 4 weeks continuous sick leave: employer must offer bEM (betriebliches Eingliederungsmanagement — return-to-work integration management) before any illness-related dismissal; failure weakens dismissal grounds

---

## Step 3: Output format

For each leave type reviewed, output:

**Entitlement:** [number of days/weeks + basis]
**Pay obligation:** [employer obligation + state benefit + collective agreement note if applicable]
**Job protection:** [yes/no/conditions]
**Employer document requirements:** [what notices, certificates, forms are required and when]
**Collective agreement note:** [if TES/Tarifvertrag applies — may be more generous; verify against applicable agreement]

---

## Guardrail

> **RESEARCH NOTES — NOT LEGAL ADVICE.** Leave entitlements interact with collective agreements, individual contract terms, and Kela/Krankenkasse rules in ways that affect the actual entitlement. This analysis calculates the statutory floor. Verify the applicable TES or Tarifvertrag for enhanced entitlements, and confirm current Kela/Krankenkasse benefit levels before communicating entitlements to employees. [model knowledge — verify current benefit amounts]
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/leave-review/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/leave-review/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/leave-review/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add leave-review skill"
```

---

## Task 6: `international-expansion` skill

**Files:**
- Create: `eu-legal/skills/international-expansion/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/international-expansion
```

- [ ] **Step 2: Write `eu-legal/skills/international-expansion/SKILL.md`**

```markdown
---
name: international-expansion
description: >
  Legal checklist for hiring in a new EU country — presence type, employment
  contract requirements, works council thresholds, collective agreement
  coverage, dismissal rules, and top surprises for Nordic/German employers.
  Covers Finland, Germany, Estonia, Sweden, Netherlands, France in v1; routes
  other EU countries to outside counsel with a question list. Also covers
  posted workers (Directive 2018/957) and social security coordination
  (Reg 883/2004). Use when asked "we want to hire in [country]", "expanding to
  [country]", "posted worker rules", "what do we need to know to hire in [X]".
argument-hint: "[target country] — e.g. 'Netherlands' or 'Estonia' or 'posted worker Finland to Germany'"
---

# /eu-legal:international-expansion

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/employment.md` if it exists. Read: hiring countries already active (for context), headcount, outside counsel.
3. Identify the target country and expansion type from the argument.
4. Route to the relevant country profile below, or to the outside counsel checklist for unlisted countries.
5. Output: checklist of required steps before first hire.

---

## Purpose

Each EU country has different employment law, social security, tax, and registration requirements. What works in Finland does not transfer to the Netherlands, and what works in Germany does not transfer to France. This skill gives the key legal checkpoints for v1 coverage countries; for others, it produces a structured briefing checklist to take to local counsel.

---

## Step 1: Determine presence type

Ask if not clear from the argument:
> "How do you plan to engage this person or team? Select the model:"

1. **Local entity** — incorporate a subsidiary or branch in the target country
2. **Employer of Record (EOR)** — third-party company employs the worker; you direct the work
3. **Posted worker** — send an existing Finnish/German employee temporarily
4. **Genuine contractor** — engage an independent contractor (apply `/eu-legal:worker-classification` first)
5. **Remote employee** — hire someone living in target country as your employee (requires local entity or EOR in most cases)

Note: hiring an employee without local presence or EOR creates "permanent establishment" risk (corporate tax) and employer obligation risk (local payroll tax, social contributions) in most EU countries. Flag this prominently if the user selects "remote employee" without a local entity.

---

## Step 2: Posted workers (if applicable)

If presence type is "posted worker" — an employee of the Finnish/German company sent to work in another EU state:

**Directive 2018/957 (amending 96/71/EC) — host country minimum terms apply:**

The host country's mandatory rules on the following must be applied (regardless of what the employment contract says):
- Maximum work periods and minimum rest periods
- Minimum paid annual leave
- Remuneration (minimum wage of host country, including applicable collective agreement pay scales)
- Health and safety
- Equal treatment (non-discrimination)
- Accommodation and subsistence where provided by employer
- For postings > 12 months (extendable to 18 months by notification): nearly all employment terms of host country apply

**Social security (Reg 883/2004):**
- For postings up to 24 months: employee remains in home country social security; employer must obtain A1 certificate from home country authority (Kela in Finland, Deutsche Rentenversicherung in Germany)
- Apply for A1 before the posting begins — retroactive applications are possible but create gaps
- After 24 months: local social security of host country applies unless extension agreed between competent authorities

**Employer obligations for postings:**
- [ ] Notify host country labour authority before posting begins (most EU member states require online notification)
- [ ] Obtain A1 certificate
- [ ] Apply host country minimum wage and working time rules
- [ ] Keep employment documentation accessible in host country during posting

---

## Step 3: Country profiles

### Finland (FI) — for employers expanding into Finland

**Legal entity requirement:**
- Branch (sivuliike) or subsidiary (tytäryhtiö) registered with Patentti- ja rekisterihallitus (PRH)
- No local entity needed if using EOR, but Finnish tax authority (Verohallinto) may deem permanent establishment if Finnish employee exercises authority to conclude contracts

**Employment contract:**
- Must be in writing (or written statement of terms within 1 month, TSL Ch. 2 §4)
- Finnish as language standard if employee is Finnish-speaking; bilingual acceptable
- Must include: parties, start date, workplace, role, salary, notice period
- Probation period: max 6 months (TSL Ch. 1 §4); 6 months if training obligation

**Works council / YT-laki:**
- Applies at ≥20 employees regularly in Finland (see works-council-check skill)
- Personnel representation: luottamusmies if employees are unionised; henkilöstöedustaja otherwise
- 🟡 Top surprise for German employers: Finnish cooperation procedure (YT-neuvottelut) looks similar to BetrVG §111 but has different timing and liability rules — the obligation does not freeze the decision, it requires genuine prior negotiation

**Collective agreement coverage:**
- Erga omnes system: if a general binding (yleissitova) TES exists for the industry, it applies to ALL employers in that industry — even non-members. Most major industries have erga omnes TES.
- Check: tyosuojelu.fi or SAK/EK/STTK databases for applicable TES
- 🔴 Top surprise: employer may be bound by a TES it didn't sign and may not know about

**Dismissal:**
- TSL grounds required (individual or financial/production)
- Re-employment obligation 9 months post-notice
- See termination-review skill for full framework

**Top 3 surprises for German employers in Finland:**
1. No Betriebsrat equivalent for daily decisions — YT-laki only applies to major changes. Day-to-day management decisions do not require prior consultation.
2. Employer is bound by the erga omnes TES for the industry even without signing it — check the applicable TES before setting any employment terms.
3. Finnish termination law requires a "substantial and weighty reason" (asiallinen ja painava syy) — similar standard to KSchG but with a different procedural cadence: warning before conduct dismissal is required but there is no §102 BetrVG equivalent for individual dismissals.

---

### Germany (DE) — for employers expanding into Germany

**Legal entity requirement:**
- GmbH (most common) or branch (Zweigniederlassung) registered at Handelsregister
- Registration at Finanzamt for payroll tax (Lohnsteuer) and social insurance registration (Deutsche Rentenversicherung, Krankenkasse, Berufsgenossenschaft)

**Employment contract:**
- No statutory written contract requirement, but NachwG (Nachweisgesetz) requires written statement of terms on day 1 since 2022
- Required contents: parties, start date, place of work, role, probation, salary, working hours, holiday, notice period, applicable Tarifvertrag
- Probation: max 6 months (§622(3) BGB) — notice during probation: 2 weeks

**Works council / BetrVG:**
- At ≥5 permanent employees: employees have the right to form a Betriebsrat — employer cannot prevent this
- Once formed: mandatory consultation for dismissals, working hours changes, surveillance, hiring (see works-council-check skill)
- 🔴 Top surprise for Finnish employers: a Betriebsrat can be formed at any time by the employees themselves. A Finnish company with 6 German employees has no Betriebsrat today and can have one tomorrow — and once formed, all future dismissals require §102 consultation.

**Collective agreement coverage:**
- NOT erga omnes by default: Tarifvertrag binds only employer association members (tarifrechtlich) or individually signed agreements
- But: allgemeinverbindlich (generally binding) declarations can extend some TarVs — check BMAS database
- 🟡 Advantage over Finland: German employer can often choose not to be bound by TarV if not joining employer association; but must then still compete for talent against TarV-paying employers

**Dismissal:**
- KSchG requires valid grounds once >10 employees and >6 months tenure
- §102 BetrVG consultation mandatory if Betriebsrat exists
- See termination-review skill for full framework

**Top 3 surprises for Finnish employers in Germany:**
1. The Betriebsrat has veto rights on hiring, transfers, and dismissals — not just a consultation right. Failure to consult makes the dismissal void, not just risky.
2. Collective redundancy notification to Bundesagentur für Arbeit (§17 KSchG) has strict formal requirements — missing the form or timing makes the dismissals void, regardless of substantive grounds.
3. The 6-week employer sick pay obligation (EFZG) is higher than Finnish employers typically expect — build this into employment cost modelling.

---

### Estonia (EE)

**Employment contract:**
- TLS (Töölepinguseadus) — employment contracts act
- Written contract required; probation max 4 months
- Minimum wage: set annually by government (check current rate [model knowledge — verify])

**Works council:**
- Töötajate usaldusisik (employee trustee) system — employees can elect a representative if ≥10 employees
- No BetrVG equivalent — consultation rights narrower than DE/FI

**Dismissal:**
- TLS requires grounds for dismissal after probation
- Notice periods: employee-side 30 days; employer-side 15–90 days by service length
- Severance: 1 month salary for financial/production dismissals

**Collective agreement:**
- No erga omnes system; collective agreements bind only signatories
- Collective agreements less prevalent than FI/DE

**Top 3 surprises for FI/DE employers:**
1. Estonia has a flat income tax system — payroll administration is significantly simpler than Finland or Germany.
2. Employment protection is less strong than FI/DE but not absent — financial/production dismissal still requires a genuine reason.
3. E-Residency does NOT create an employment law nexus — an Estonian e-resident working in Germany is still subject to German employment law.

---

### Sweden (SE)

**Employment contract:**
- LAS (Lagen om anställningsskydd) — employment protection act; strong protection
- Main rule: indefinite employment. Fixed-term employment heavily restricted — only for: vikariat (substitute), säsongsarbete (seasonal), överenskommen visstid (agreed fixed-term, ≤2yr in 5yr period)
- Written contract required; probation max 6 months (provanställning)

**Works council / MBL:**
- MBL (Medbestämmandelagen) — applies to all employers with ≥10 employees
- Primary negotiation obligation before major changes (§11 MBL)
- Collective agreements very widely used (~90% coverage erga omnes through industrywide)

**Dismissal:**
- Saklig grund (objective grounds) required — similar to but stricter than Finnish "substantial reason"
- Turordning (last-in-first-out) rule for redundancies: cannot cherry-pick who to make redundant unless fewer than 10 employees
- Re-employment priority: 9 months

**Top 3 surprises for FI/DE employers:**
1. The turordning (last-in-first-out) rule for redundancies is stronger than in Germany or Finland — employer cannot bypass it without collective agreement exception. Select carefully who you hire first.
2. Collective agreement coverage is ~90% and erga omnes for many sectors. The employer is practically always bound by a TES equivalent (kollektivavtal).
3. Discrimination claims are administered by DO (Diskrimineringsombudsmannen) and are actively enforced — budget for this in HR processes.

---

### Netherlands (NL)

**Employment contract:**
- Burgerlijk Wetboek Art. 7:610 — employment law in the Civil Code
- Written contract standard; probation max 1 month (up to 2 months for contracts >2 years)
- Transitievergoeding (transition payment) on termination: 1/3 month salary per year, regardless of reason — payable from day 1

**Works council / WOR:**
- WOR (Wet op de ondernemingsraden) — applies at ≥50 employees
- Ondernemingsraad (OR) has consent and advisory rights
- At 10–49 employees: personeelsvertegenwoordiging (PVT) — lighter consultation body
- 🔴 Consent right: OR must consent before: profit sharing, pension changes, working time changes, salary systems, disciplinary procedures, personnel monitoring

**Dismissal:**
- Requires UWV (social security authority) or Kantonrechter (court) approval before dismissal — employer cannot simply give notice
- Two routes: UWV procedure for business reasons (typically 4 weeks+); Kantonrechter for personal grounds (1-3 months)
- Transitievergoeding always payable; additional vergoeding (compensation) if dismissal unfair

**Top 3 surprises for FI/DE employers:**
1. Dutch dismissal law requires prior government (UWV) or court approval — you cannot dismiss with notice alone. Plan for 4–12 weeks procedure time.
2. The transitievergoeding (transition payment of 1/3 month per year) is payable on every termination including redundancy — budget for it from day 1.
3. Dutch employment law distinguishes carefully between ZZP (zelfstandige zonder personeel — independent contractor) and employee. The same Platform Work Directive indicators apply; Dutch authorities actively audit this.

---

### France (FR)

**Employment contract:**
- Code du travail — comprehensive labour code; highly protective of employees
- CDI (contrat à durée indéterminée) is the norm; CDD (fixed-term) strictly limited
- Written contract required; probation: 2 months (workers), 3 months (supervisors), 4 months (cadres)

**Works council / CSE:**
- CSE (Comité Social et Économique) mandatory from ≥11 employees
- At ≥11 employees: basic consultation rights
- At ≥50 employees: full CSE with broader powers; BDES (economic database) obligation
- 🔴 Strong co-determination: CSE must be consulted before major changes; consultation failure can result in offence d'entrave (criminal obstruction charge against management)

**Dismissal:**
- Cause réelle et sérieuse (real and serious ground) required
- Mandatory prior meeting (entretien préalable) with employee before any dismissal
- Notice period by collective agreement or statutory minimums
- Prud'hommes (labour tribunal): very active; statute of limitations 1 year (personal) / 12 months (discrimination)

**Top 3 surprises for FI/DE employers:**
1. The criminal offence d'entrave (obstruction) for failing to consult the CSE is not theoretical — managers can be personally fined or imprisoned. Consultation is not optional.
2. French collective agreements (conventions collectives) are complex, layered (national + sectoral + company level), and often more restrictive than the Code du travail. The applicable convention collective depends on the company's IDCC code (classification code).
3. Licenciement économique (economic dismissal) for companies ≥50 requires a Plan de Sauvegarde de l'Emploi (PSE) — a negotiated document covering redeployment, retraining, and outplacement. This is a multi-month process and requires validation by the DREETS (labour authority).

---

## Other EU countries

For EU member states not covered above (Austria, Belgium, Poland, Czech Republic, Spain, Italy, Portugal, etc.):

> "This skill covers Finland, Germany, Estonia, Sweden, Netherlands, and France in v1. For [target country], I can produce a structured question list to take to local employment counsel."

Generate the following checklist for outside counsel briefing:

**Questions for [country] employment counsel:**
1. What written contract requirements apply (statutory form, language, required clauses)?
2. What is the maximum probation period?
3. Is there a works council or employee representation body? At what headcount threshold? What are its consent vs. advisory rights?
4. What collective agreement system applies — erga omnes or opt-in only? Is there an applicable sectoral agreement?
5. What are the valid grounds for dismissal? Is prior authority approval required?
6. What are the notice periods by service length?
7. Is statutory severance payable on dismissal? If so, what formula?
8. What is the employer's sick pay obligation (days and percentage)?
9. What are the parental and maternity leave entitlements and the associated job protection period?
10. What are the top 3 surprises for employers coming from Finland or Germany?

---

## Guardrail

> **RESEARCH NOTES — NOT LEGAL ADVICE.** Employment law requirements in each EU member state vary significantly from the general principles described here. This checklist identifies key legal checkpoints — it does not substitute for advice from a qualified employment lawyer in the target country. Before the first hire: engage local counsel, confirm current minimum wage and benefit rates, and verify that the applicable collective agreement has been identified. [model knowledge — verify current thresholds and benefit amounts for each country]
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/international-expansion/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/international-expansion/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/international-expansion/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add international-expansion skill"
```

---

## Task 7: `policy-drafting` skill

**Files:**
- Create: `eu-legal/skills/policy-drafting/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/mattipuhakka/law-for-eu/eu-legal/skills/policy-drafting
```

- [ ] **Step 2: Write `eu-legal/skills/policy-drafting/SKILL.md`**

```markdown
---
name: policy-drafting
description: >
  Draft employment policies in EU-compliant form — remote work, employee
  monitoring, AI use (employee-facing), whistleblower reporting, and equal
  treatment. Includes works council consultation requirements, GDPR Art. 88
  limits on employee data, Finnish privacy-in-working-life rules, and German
  BetrVG co-determination rights. Use when asked to "draft a remote work
  policy", "write an employee monitoring policy", "whistleblower policy",
  "AI use policy for employees", or "equal treatment policy".
argument-hint: "[policy type] — e.g. 'remote work policy', 'employee monitoring', 'whistleblower', 'AI use policy', 'equal treatment'"
---

# /eu-legal:policy-drafting

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/employment.md` if it exists. Read: hiring countries, de_betriebsrat_exists, fi_yt_laki_applies, total_headcount, whistleblower_channel_required.
3. Identify the policy type from the argument.
4. Run the applicable framework below.
5. Output: draft policy for attorney review, with a pre-rollout checklist of consultation requirements.

---

## Purpose

Employment policies create legal obligations as well as rights. A monitoring policy that doesn't comply with GDPR Art. 88 or the Finnish privacy-in-working-life act exposes the employer to DPA enforcement. A whistleblower policy that doesn't meet Directive 2019/1937 requirements exposes the employer to national authority fines. A policy rolled out in Germany without Betriebsrat approval under §87 BetrVG may be void. This skill drafts compliant first drafts and identifies what must happen before the policy is communicated.

---

## Step 1: Identify policy type and collect context

From the argument, identify which policy type to draft. If not specified, ask:
> "Which policy would you like to draft? (1) Remote work / (2) Employee monitoring / (3) AI use for employees / (4) Whistleblower reporting / (5) Equal treatment"

Also collect (if not in employment.md):
- Primary jurisdiction for the policy (Finland / Germany / both)
- Approximate headcount (for whistleblower threshold check)
- Applicable collective agreement (TES/Tarifvertrag) — may have policy-specific requirements

---

## Policy type 1: Remote Work Policy

### Legal basis
- EU: Transparent and Predictable Working Conditions Directive (2019/1152) — employee has right to request flexible arrangements; employer must respond within reasonable time
- Finland: TSL Ch. 2 §13a — right to request remote work; employer must give reasoned refusal
- Germany: no statutory right to remote work (Homeoffice-Gesetz proposal did not pass), but request right exists in some Tarifverträge; BetrVG §87(1) No. 14 — co-determination right on "mobile working" since 2021

### Works council consultation (before rollout)
- **Finland (≥20 employees):** If policy changes existing practice or introduces new monitoring requirements → YT-laki §16 procedure required before implementation
- **Germany (Betriebsrat exists):** BetrVG §87(1) No. 14 — employer must negotiate a Betriebsvereinbarung on mobile work before implementing a company-wide remote work policy. Cannot act unilaterally.

### GDPR / Data protection requirements
- If remote work involves monitoring home office equipment: GDPR Art. 88 applies (employee data processing)
- Proportionality: monitoring must be necessary and proportionate; no keystroke logging or continuous camera surveillance
- Finland: Laki yksityisyyden suojasta työelämässä (759/2004) — employer can only process employee data for purposes directly related to employment; must inform employees of monitoring

### Draft policy template

```
REMOTE WORK POLICY
[Company name] — effective [DATE]
[Version: DRAFT — FOR ATTORNEY REVIEW AND WORKS COUNCIL CONSULTATION BEFORE ROLLOUT]

1. SCOPE
This policy applies to all employees of [Company] working remotely on a regular or occasional basis. "Remote work" means work performed at a location other than the company's designated workplace, including home, co-working spaces, and other locations.

2. ELIGIBILITY
Remote work is available to employees in roles where work can be performed effectively outside the office, as determined by the employee's manager. Remote work availability depends on role requirements, performance, and operational needs.

3. REQUESTING REMOTE WORK
Employees may request regular remote work arrangements. Requests should be made in writing to the employee's manager and HR. The company will respond within [30 days] with either approval, a modified proposal, or a reasoned refusal. A reasoned refusal must state the business reason why the request cannot be accommodated.

4. WORKING HOURS AND AVAILABILITY
Employees working remotely remain subject to their contracted working hours and the [Finnish/German] Working Hours Act / Arbeitszeitsgesetz. Core hours of availability: [e.g., 09:00–15:00 local time, Monday–Friday]. Outside core hours, employees may organise their time flexibly, subject to applicable working time limits.

5. HEALTH AND SAFETY
The employee is responsible for ensuring their remote workspace meets basic ergonomic and safety requirements. The company will provide guidance on workspace requirements. Employer occupational health and safety obligations (Finnish occupational safety and health act / German ArbSchG) apply to remote work locations to the extent practicable.

6. DATA SECURITY
Employees must follow the company's information security policy when working remotely: use VPN, lock screens when unattended, do not use public Wi-Fi without VPN, and store work documents on company-approved systems only.

7. EQUIPMENT
[Employer-provided equipment]: The company provides [laptop, headset, monitor — list]. Employees are responsible for maintaining equipment in working order.
[Employee-provided equipment (BYOD)]: If using personal equipment, the employee must ensure it meets the company's security requirements.

8. MONITORING
[Employer note: any monitoring tools must be described here. Apply the monitoring policy framework — see Policy type 2 — before inserting monitoring terms. Do not include monitoring without GDPR Art. 88 / privacy-in-working-life compliance review.]

9. COSTS AND EXPENSES
[Finland: employer typically reimburses reasonable additional costs per TES or agreement]
[Germany: employer pays Aufwendungsersatz per §670 BGB for expenses caused by remote work; internet costs: employer pays a reasonable share if employee uses home internet for work]

10. CHANGE OR WITHDRAWAL
Either party may request a change to a regular remote work arrangement with [1 month] notice. The company may withdraw remote work access for business reasons with reasonable notice, except where a permanent arrangement has been agreed in the employment contract.

11. APPLICABLE LAW
This policy is governed by [Finnish / German] employment law and the employee's employment contract.

---
[REVIEWER NOTE: This policy must be reviewed by employment counsel before rollout. In Germany: Betriebsvereinbarung required under BetrVG §87(1) No. 14 — do not communicate this policy to employees until works council agreement is in place. In Finland: if changing existing practice for ≥20 employees, YT-laki §16 consultation required before implementation.]
```

---

## Policy type 2: Employee Monitoring Policy

### Legal basis
- GDPR Art. 88 — member states may provide specific rules on employee data processing; proportionality and necessity required
- Finland: Laki yksityisyyden suojasta työelämässä (759/2004) §4 — employer may only process personal data that is directly necessary for the employment relationship; employee must be informed of monitoring; most monitoring requires prior notification and YT-laki consultation
- Germany: BetrVG §87(1) No. 6 — introduction and use of technical devices capable of monitoring employee performance or behaviour requires Betriebsrat co-determination

### Works council consultation (before rollout)
- **Germany:** BetrVG §87(1) No. 6 is a mandatory co-determination right. No monitoring system — email logs, access logs, key card records, screen monitoring, GPS tracking, call recording, productivity tools — may be introduced without a Betriebsvereinbarung. This is not a consultation requirement; it is a consent requirement. Unilateral introduction = void measure.
- **Finland (≥20 employees):** YT-laki §16 — introduction of monitoring systems that affect working conditions requires YT-neuvottelut before implementation.

### What monitoring is permissible under Finnish privacy-in-working-life act (759/2004)

| Monitoring type | Permissible? | Requirements |
|---|---|---|
| Access logs (who entered building, when) | Yes, if proportionate | Must inform employees; YT consultation if ≥20 |
| Email content monitoring | Only if specific grounds (security investigation, work completion) | Must inform; cannot monitor content routinely |
| Internet access logs (which sites visited) | Yes, if proportionate and work-related | Must inform employees |
| Keystroke logging / screen recording | No (disproportionate as routine monitoring) | Not permitted |
| GPS tracking (company vehicles) | Yes, if work-related | Inform employees; cannot track off-duty |
| Productivity software (output metrics) | Yes, if non-invasive | Inform employees; YT consultation |
| AI-based performance scoring | Permitted only with safeguards | GDPR Art. 22 profiling restrictions; must inform; YT consultation |

### Draft policy template

```
EMPLOYEE MONITORING POLICY
[Company name] — effective [DATE]
[Version: DRAFT — FOR ATTORNEY REVIEW AND WORKS COUNCIL CONSULTATION BEFORE ROLLOUT]

1. PURPOSE AND LEGAL BASIS
[Company] monitors certain employee activities to ensure the security of company systems, compliance with company policies, and the effective operation of the business. This policy describes what is monitored, why, and how information is used.

Legal basis for processing: [Article 6(1)(b) GDPR — necessary for performance of the employment contract; Article 6(1)(f) GDPR — legitimate interests in IT security and compliance]. Processing of sensitive data categories requires Article 9 basis — state if applicable.

2. WHAT WE MONITOR

We monitor the following:
- IT system access logs: which systems were accessed, when, and from which device. Purpose: IT security, incident investigation.
- Company email metadata: sender, recipient, subject line, timestamps. Purpose: security, compliance. [Note: email content is NOT routinely monitored. Content may be accessed only for specific security or legal investigation purposes, following the procedure in section 6.]
- Internet access logs: which domains were accessed from company networks or devices. Purpose: security, policy compliance.
- Building access logs: entry/exit records from key card systems. Purpose: site security.
- [Company vehicles only:] GPS location during working hours. Purpose: fleet management, safety.

We do NOT monitor:
- Keystroke logging or screen recording during normal work
- Personal devices used for private purposes
- Personal email or personal social media accounts
- Location outside working hours

3. HOW MONITORING DATA IS USED
Monitoring data is used only for the purposes stated above. Access to monitoring data is restricted to IT security team, HR (in HR-related investigations), and management (for business purposes). Monitoring data is not used for automated individual decision-making (GDPR Art. 22) without human review.

4. RETENTION
Monitoring data is retained for [90 days] for IT logs; [30 days] for access logs; unless longer retention is required for an ongoing investigation.

5. EMPLOYEE RIGHTS
Employees have the right to access their own monitoring data (GDPR Art. 15), to request correction of inaccurate data (Art. 16), and to object to processing where it affects their fundamental rights (Art. 21). Contact [data protection contact] to exercise these rights.

6. INVESTIGATIONS
If monitoring data suggests a security incident, policy violation, or other concern, the company may initiate an investigation. Any investigation involving review of personal communications (e.g., email content) will be conducted with prior authorisation from [HR lead / Legal] and with notice to the employee unless legal proceedings require confidentiality.

7. WORKS COUNCIL
[Germany: This policy was agreed with the Betriebsrat by Betriebsvereinbarung dated [DATE]. [Leave blank until agreement in place — do not rollout without.]]
[Finland: This policy was developed following YT-laki §16 consultation with employee representatives on [DATE].]

---
[REVIEWER NOTE: Before rollout — Germany: Betriebsvereinbarung required under BetrVG §87(1) No. 6. Finland: YT-laki §16 consultation required for ≥20 employees. Remove any monitoring type not explicitly stated here before communicating to employees — adding monitoring later requires a new consultation cycle.]
```

---

## Policy type 3: AI Use Policy (Employee-Facing)

### Legal basis
- EU AI Act (Regulation 2024/1689) — transparency and information obligations for AI systems interacting with employees (Art. 50); high-risk AI for employment, HR, and worker management subject to conformity assessment (Annex III)
- GDPR Art. 22 — right not to be subject to solely automated decisions with significant effect; applies to AI-based performance evaluation, promotion decisions, disciplinary AI systems
- Finland: Laki yksityisyyden suojasta työelämässä (759/2004) — employee data processing; AI systems are subject to same proportionality standard
- Germany: BetrVG §87(1) No. 6 — AI performance monitoring tools trigger co-determination right. §90+91 BetrVG — employer must consult Betriebsrat before introducing new work processes (including AI tools)

### Works council consultation
- **Germany:** Any AI tool that monitors performance, schedules work, or evaluates output requires Betriebsvereinbarung under §87(1) No. 6 before deployment to employees.
- **Finland (≥20 employees):** Introduction of AI tools that materially affect how work is done or monitored → YT-laki §16 procedure.

### Draft policy template

```
AI TOOLS POLICY FOR EMPLOYEES
[Company name] — effective [DATE]
[Version: DRAFT — FOR ATTORNEY REVIEW AND WORKS COUNCIL CONSULTATION BEFORE ROLLOUT]

1. SCOPE
This policy applies to all employees using AI tools provided by [Company], or using externally available AI tools for work purposes.

2. APPROVED AI TOOLS
The following AI tools are approved for use at [Company]:
[List approved tools, e.g.: GitHub Copilot (coding only), Claude / ChatGPT (drafting and research), internal AI assistant (describe scope)]

Using AI tools not on this list for work purposes requires prior approval from [IT/Legal/manager — specify].

3. WHAT YOU MUST NOT ENTER INTO AI TOOLS
Do not enter into any AI tool (including approved tools):
- Personal data of clients, employees, or third parties, unless the tool is explicitly approved for this and data processing terms are in place
- Confidential company information (financial data, unreleased products, M&A discussions) into consumer-grade AI tools
- Source code that is not already public, unless using a tool with code privacy guarantees (e.g., GitHub Copilot with enterprise data protection)
- Passwords, API keys, or credentials

4. ACCURACY AND VERIFICATION
AI tools can produce incorrect, outdated, or fabricated output. You are responsible for verifying AI-generated content before using it in customer communications, legal documents, financial calculations, or other consequential outputs. Treating AI output as final without review is a policy violation.

5. AI-BASED PERFORMANCE TOOLS
[If company uses AI for performance assessment or work allocation]: [Company] uses [tool name] for [purpose]. This tool processes [what data] for the purpose of [what decisions]. Human review is applied to all significant decisions affecting individual employees. You have the right to request human review of any individual decision made with significant AI input — contact [HR contact].

The use of AI for employment-related decisions that significantly affect you (performance ratings used for pay, promotion, or disciplinary action) is subject to GDPR Art. 22 safeguards. You have the right to: (a) know that an automated system was used, (b) obtain an explanation, (c) request human review of the decision.

6. INTELLECTUAL PROPERTY
AI-generated output created in the course of your employment belongs to [Company] under standard IP assignment terms. Verify before publishing or using AI-generated work that you hold or the company holds the necessary rights.

7. WORKS COUNCIL
[Germany: The introduction of AI performance monitoring tools described in section 5 was agreed with the Betriebsrat by Betriebsvereinbarung dated [DATE].]
[Finland: This policy was developed following YT-laki §16 consultation with employee representatives on [DATE].]

---
[REVIEWER NOTE: Before rollout — Germany: any AI tool that monitors performance or outputs requires Betriebsvereinbarung. Finland: YT-laki consultation for ≥20 employees before rollout. Check whether any AI tool used for employment decisions (performance, scheduling, candidate screening) qualifies as high-risk under EU AI Act Annex III — if so, conformity assessment required before deployment.]
```

---

## Policy type 4: Whistleblower Reporting Policy

### Legal basis
- EU Whistleblower Directive 2019/1937 — mandatory internal reporting channel for:
  - Private sector employers ≥50 employees
  - All public sector employers
- Finland implementing act: Laki ilmoittajansuojelusta (1171/2022) — in force from 1 January 2023 for employers ≥50 employees
- Germany implementing act: Hinweisgeberschutzgesetz (HinSchG) — in force from 2 July 2023 for employers ≥50 employees (private sector ≥250 from 2 July 2023; ≥50 from 17 December 2023)

### Mandatory requirements

**Reporting channel requirements (both Directive and implementing acts):**
- Secure, confidential channel that allows both written and verbal reporting
- Channel must allow anonymous reporting (employer must accept anonymous reports, though cannot compel reporter to identify themselves)
- Designated impartial person or department to handle reports
- Acknowledgement within 7 days of receiving report
- Feedback to reporter within 3 months (6 months with extension)
- No retaliation against reporter — null and void

**Scope of reports covered:**
- Violations of EU law in: financial services, anti-money laundering, product safety, transport safety, food safety, environmental law, radiation, nuclear safety, public health, consumer protection, privacy and data protection, network and information security, competition law, corporate tax, and EU institutional law
- Finland: also covers serious Finnish employment law violations
- Germany: also covers serious German law violations

**Threshold check:** if `whistleblower_channel_required: true` from employment.md, mandatory. If `total_headcount` < 50 but ≥ 10: recommendation only (Directive permits member states to exempt <50; check current FI and DE rules).

### Draft policy template

```
WHISTLEBLOWER REPORTING POLICY
[Company name] — effective [DATE]
[Version: DRAFT — FOR ATTORNEY REVIEW BEFORE ROLLOUT]

1. PURPOSE
[Company] is committed to lawful and ethical conduct. This policy establishes a safe, confidential channel for employees and other persons to report concerns about violations of law or serious breaches of company policy, without fear of retaliation.

This policy implements the requirements of EU Directive 2019/1937 on the protection of persons who report breaches of Union law, [Finnish Act on the Protection of Whistleblowers 1171/2022 / German Hinweisgeberschutzgesetz (HinSchG)].

2. WHO CAN REPORT
Any person associated with [Company] may make a report under this policy, including:
- Current and former employees
- Job applicants
- Contractors, suppliers, and business partners
- Shareholders and board members

3. WHAT CAN BE REPORTED
Reports should concern suspected violations of:
- EU law in the areas of financial services, anti-money laundering, data protection, competition, consumer protection, environmental law, product safety, and public procurement
- [Finland: serious violations of Finnish employment law, financial markets regulation, or other significant regulatory requirements]
- [Germany: serious violations of German law, including employment law violations, criminal conduct, or serious regulatory breaches]
- Serious violations of company policy or internal controls, where there is a public interest concern

This policy is not designed for general HR grievances (employment disputes, salary questions, holiday entitlements) — those should be raised through normal HR channels.

4. HOW TO REPORT

**Internal reporting channel:**
[Channel method: e.g., confidential online reporting portal at [URL], encrypted email at [address], or dedicated phone line at [number]]

Reports may be made anonymously. Identifying yourself is not required, though it makes follow-up easier. The company accepts anonymous reports.

**External reporting channel:**
You may also report directly to the relevant authority without making an internal report first. In Finland: the Occupational Safety and Health Administration (aluehallintovirasto) and sector-specific authorities. In Germany: the Bundesamt für Justiz (BfJ) operates a federal external reporting channel.

5. CONFIDENTIALITY
The identity of the reporter will be kept confidential to the maximum extent possible. Information that could identify the reporter will not be disclosed except where required by law. The person handling reports will not disclose the reporter's identity without their consent.

6. HANDLING PROCESS
After a report is received:
- **Within 7 days:** Acknowledgement of receipt
- **Within 3 months:** Feedback on the action taken (or planned), including whether the matter was investigated and its outcome — subject to confidentiality obligations regarding others involved
- Reports are handled by [designated person / function, e.g., "the Legal team" or "the Independent Ethics Officer"]
- Reports may be investigated by an internal or external investigator

7. PROTECTION FROM RETALIATION
Retaliation against a reporter, facilitator of a report, or any person associated with a report is prohibited and will result in disciplinary action. Retaliation includes dismissal, demotion, salary reduction, harassment, or any other detrimental treatment connected to a report. Any person who believes they have been retaliated against should contact [HR / Legal contact] immediately.

Reports made in good faith are protected even if the reported concern turns out to be unfounded. False reports made knowingly and maliciously are not protected and may result in disciplinary action.

8. RECORD-KEEPING
[Company] maintains a confidential register of reports received. This register records the date of report, the subject matter (not the reporter's identity), the action taken, and the outcome. Records are retained for [5 years] after the close of the matter.

9. REVIEW
This policy is reviewed annually and updated as required by law or best practice.

---
[REVIEWER NOTE: Before rollout — verify that the reporting channel meets technical requirements (secure, confidential, anonymous-capable) before communicating this policy. Germany: report this policy to Betriebsrat — §87 BetrVG co-determination may apply to the implementation of the reporting system. Finland: YT-laki §16 consultation recommended before rollout for ≥20 employees. Confirm implementing act applicability threshold for current headcount. [model knowledge — verify current HinSchG applicability thresholds for employers 50–249 employees]]
```

---

## Policy type 5: Equal Treatment Policy

### Legal basis
- Finland: Tasa-arvolaki (Laki naisten ja miesten välisestä tasa-arvosta, 609/1986) — gender equality; employers ≥30 employees must have tasa-arvosuunnitelma (equality plan) updated every 2 years
- Finland: Yhdenvertaisuuslaki (1325/2014) — equal treatment across all grounds; employers ≥30 employees must have yhdenvertaisuussuunnitelma (equality and non-discrimination plan)
- Germany: AGG (Allgemeines Gleichbehandlungsgesetz) — anti-discrimination; applies to all employers regardless of size
- EU: Employment Equality Directive (2000/78/EC), Race Equality Directive (2000/43/EC)

### Protected characteristics (all EU/FI/DE)
| Ground | FI (Tasa-arvo/YV) | DE (AGG) | EU Directives |
|---|---|---|---|
| Sex / gender | ✓ | ✓ | ✓ |
| Pregnancy / parental status | ✓ | ✓ | ✓ |
| Ethnic origin / nationality | ✓ | ✓ | ✓ |
| Disability | ✓ | ✓ | ✓ |
| Age | ✓ | ✓ | ✓ |
| Sexual orientation | ✓ | ✓ | ✓ |
| Religion or belief | ✓ | ✓ | ✓ |
| Language | ✓ (Finnish specific) | — | — |

### Draft policy template

```
EQUAL TREATMENT AND NON-DISCRIMINATION POLICY
[Company name] — effective [DATE]
[Version: DRAFT — FOR ATTORNEY REVIEW BEFORE ROLLOUT]

1. COMMITMENT
[Company] is committed to equal treatment of all employees, candidates, and persons associated with the company. Discrimination, harassment, and victimisation based on any protected characteristic are prohibited.

2. SCOPE
This policy applies to all aspects of employment including: recruitment and selection, terms and conditions, pay, access to training and development, promotion, and termination. It applies to all employees, managers, contractors, and third parties acting on the company's behalf.

3. PROTECTED CHARACTERISTICS
No person will be treated less favourably because of: sex or gender; pregnancy or parental leave status; marital or family status; ethnic or national origin; race; colour; language; religion or belief; disability; age; sexual orientation; gender identity; or political opinion.

4. WHAT CONSTITUTES DISCRIMINATION

**Direct discrimination:** treating a person less favourably because of a protected characteristic.

**Indirect discrimination:** applying a practice, criterion, or provision that, while apparently neutral, puts persons with a protected characteristic at a particular disadvantage — unless the practice is objectively justified.

**Harassment:** unwanted conduct related to a protected characteristic that has the purpose or effect of violating a person's dignity or creating an intimidating, hostile, degrading, or offensive environment.

**Victimisation:** treating a person unfairly because they have raised a concern under this policy or assisted another person in doing so.

5. OBLIGATIONS OF MANAGERS
Managers are responsible for ensuring equal treatment within their teams, including in: hiring decisions, performance reviews, access to development opportunities, and disciplinary processes. Managers who become aware of potential discrimination must report it to HR promptly.

6. REASONABLE ADJUSTMENTS (DISABILITY)
[Company] will make reasonable adjustments for employees and candidates with disabilities to ensure they are not placed at a substantial disadvantage. Employees may request an adjustment at any time by contacting HR. Adjustments will be documented and reviewed regularly.

[Finland: Employers must take proportionate measures to enable a disabled person to access, perform, and advance in employment — Yhdenvertaisuuslaki §15]
[Germany: §164 SGB IX — employer with ≥20 employees is subject to the Schwerbehindertenrecht; quota obligation (5% of positions for severely disabled persons)]

7. PAY EQUITY
[Finland: required for employers ≥30 employees — tasa-arvosuunnitelma must include a pay survey (palkkakartoitus) every 3 years examining whether pay differences between men and women performing equivalent work are justified]
[Germany: Entgelttransparenzgesetz — employees in companies ≥200 have the right to request information about the pay of comparators of the opposite sex]

8. REPORTING A CONCERN
Any employee who believes they have experienced or witnessed discrimination, harassment, or victimisation should report it to [HR contact] or their manager (or, if the manager is involved, to [alternate contact or whistleblower channel]).

All reports will be treated confidentially to the extent possible and investigated promptly. Victimisation of a person who raises a concern in good faith is a disciplinary offence.

9. EQUALITY PLAN [Finland — required for ≥30 employees]
In accordance with Finnish equality legislation, [Company] maintains:
- A tasa-arvosuunnitelma (gender equality plan) updated every 2 years, including a palkkakartoitus (pay survey)
- A yhdenvertaisuussuunnitelma (non-discrimination and equality plan) updated every 2 years

These plans are developed in cooperation with employee representatives and are available to all employees. [Link to current plans or note location]

10. REVIEW
This policy is reviewed every 2 years, or sooner if required by changes in law. [Finland: equality plans updated in conjunction with this review.]

---
[REVIEWER NOTE: Finland ≥30 employees: tasa-arvosuunnitelma and yhdenvertaisuussuunnitelma are legally required — this policy template does not substitute for them. They must be prepared in cooperation with employee representatives. Germany: AGG §12 requires employers to take measures to prevent discrimination — this policy satisfies that obligation, but consider adding AGG-specific complaint procedures (Beschwerdestelle designation required under §13 AGG). Check that pay survey has been or will be conducted within required cycle.]
```

---

## Pre-rollout checklist

After drafting any policy, output this checklist:

**Before communicating this policy to employees:**

- [ ] Attorney review complete — policy reflects current law and applicable collective agreement
- [ ] Finland (≥20 employees): YT-laki §16 consultation complete — personnel rep has been given opportunity to comment and discussion has concluded
- [ ] Germany (Betriebsrat exists): applicable BetrVG co-determination process complete — Betriebsvereinbarung signed or arbitration board ruling obtained
- [ ] Monitoring policies: GDPR Art. 88 / Finnish privacy-in-working-life act requirements confirmed with DPO or privacy counsel
- [ ] Whistleblower policy: reporting channel is operational and tested before policy communicated
- [ ] Equal treatment policy: equality plan development process initiated (Finland ≥30 employees); AGG Beschwerdestelle designated (Germany)
- [ ] Policy distributed and acknowledged — record employee acknowledgment date

---

## Guardrail

> **CONFIDENTIAL — INTERNAL LEGAL ANALYSIS — DRAFT FOR ATTORNEY REVIEW.** Employment policies create legal obligations as well as rights. Policies involving employee monitoring, performance assessment, or disciplinary procedures must be reviewed by an employment lawyer and — where required — agreed with the Betriebsrat (Germany) or developed through YT-laki consultation (Finland) before they are communicated to employees. A policy rolled out without required works council consultation may be void and expose the employer to liability. These drafts are starting points, not final documents.
```

- [ ] **Step 3: Verify file was written**

```bash
ls -la /Users/mattipuhakka/law-for-eu/eu-legal/skills/policy-drafting/
head -5 /Users/mattipuhakka/law-for-eu/eu-legal/skills/policy-drafting/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git -C /Users/mattipuhakka/law-for-eu add eu-legal/skills/policy-drafting/SKILL.md
git -C /Users/mattipuhakka/law-for-eu commit -m "feat: add policy-drafting skill"
```

---

## Task 8: Final validation and skill count check

**Files:** no new files; validate all 7 skills and confirm count.

- [ ] **Step 1: Validate all employment skills**

```bash
cd /Users/mattipuhakka/law-for-eu/eu-legal && claude plugin validate . 2>&1
```

Expected: no errors across all 7 new skills (`employment-cold-start`, `worker-classification`, `termination-review`, `works-council-check`, `leave-review`, `international-expansion`, `policy-drafting`).

- [ ] **Step 2: Verify all skill directories exist**

```bash
ls /Users/mattipuhakka/law-for-eu/eu-legal/skills/
```

Expected output includes all 7 new skills plus the 23 from Plans 1–3:
```
cold-start-interview
customize
deadlines
employment-cold-start
enforcement-intelligence
international-expansion
leave-review
obligations
policy-drafting
reg-gap-analysis
reg-watch
termination-review
worker-classification
works-council-check
[...plus others from Plans 2–3]
```

- [ ] **Step 3: Verify each SKILL.md has correct frontmatter**

```bash
for dir in employment-cold-start worker-classification termination-review works-council-check leave-review international-expansion policy-drafting; do
  echo "=== $dir ==="
  head -8 /Users/mattipuhakka/law-for-eu/eu-legal/skills/$dir/SKILL.md
done
```

Expected: each file starts with `---`, has `name:`, `description:`, and `argument-hint:` fields.

- [ ] **Step 4: Verify no US law references in any employment skill**

```bash
grep -r "FLSA\|FMLA\|EEOC\|OWBPA\|COBRA\|at-will\|W-2\|1099\|WARN Act\|state law" \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/employment-cold-start/ \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/worker-classification/ \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/termination-review/ \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/works-council-check/ \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/leave-review/ \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/international-expansion/ \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/policy-drafting/
```

Expected: no output (zero matches).

- [ ] **Step 5: Verify each skill has a guardrail**

```bash
grep -l "RESEARCH NOTES\|NOT LEGAL ADVICE\|attorney review\|DRAFT FOR ATTORNEY REVIEW" \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/employment-cold-start/SKILL.md \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/worker-classification/SKILL.md \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/termination-review/SKILL.md \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/works-council-check/SKILL.md \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/leave-review/SKILL.md \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/international-expansion/SKILL.md \
  /Users/mattipuhakka/law-for-eu/eu-legal/skills/policy-drafting/SKILL.md
```

Expected: all 7 file paths listed (every skill has a guardrail).

- [ ] **Step 6: Final batch commit for the plan file**

```bash
git -C /Users/mattipuhakka/law-for-eu add docs/superpowers/plans/2026-05-29-eu-legal-plan-4-employment.md
git -C /Users/mattipuhakka/law-for-eu commit -m "docs: add Plan 4 employment skills implementation plan"
```

---

## Self-review against spec

### Spec coverage check

| Spec requirement | Task covering it |
|---|---|
| `employment-cold-start` — interview, config write, offer next steps | Task 1 |
| `worker-classification` — Platform Work Directive 5 indicators, TSL test, §611a BGB test, risk level | Task 2 |
| `termination-review` — FI notice period table by tenure, individual/production grounds, YT-laki check, DE KSchG grounds, §102 BetrVG, protected status tables | Task 3 |
| `works-council-check` — YT-laki §16 + §51, BetrVG §102 + §87 + §111, EU Directive 98/59 thresholds, consequence of skipping | Task 4 |
| `leave-review` — EU directive minimums, FI Vuosilomalaki, FI parental leave new 2023 model, DE BUrlG, DE EFZG, DE BEEG/Elterngeld, DE Mutterschutz | Task 5 |
| `international-expansion` — FI/DE/EE/SE/NL/FR profiles, posted workers Directive, A1 certificate, social security coordination, EOR PE risk, outside counsel checklist | Task 6 |
| `policy-drafting` — 5 policy types, works council consultation before rollout, GDPR Art. 88, FI 759/2004, DE §87 BetrVG, whistleblower Directive + FI 1171/2022 + DE HinSchG, equal treatment FI tasa-arvo + DE AGG | Task 7 |
| `employment.md` template — all fields from spec (headcount, collective agreements, notice policies, non-compete, rep body, Betriebsrat, termination policy) | Task 1 |
| No US law references | Tasks 1–7 (verified in Task 8) |
| Guardrail on all employment skills | Tasks 1–7 (verified in Task 8) |
| No Co-Authored-By in commits | All tasks |
| Final skill count 30 | Task 8 |

No gaps found.

### Placeholder scan

No "TBD", "TODO", "implement later", or "fill in details" strings present. All policy templates contain complete draft text with section-level placeholders (e.g., `[Company name]`, `[DATE]`) that are standard in policy templates and not implementation gaps.

### Type / name consistency check

- Config key `fi_yt_laki_applies` used consistently across: Task 1 (written), Task 4 (read), Task 6 (referenced)
- Config key `de_betriebsrat_exists` used consistently across: Task 1 (written), Task 3 (read), Task 4 (read), Task 7 (read)
- Config key `whistleblower_channel_required` used consistently across: Task 1 (written), Task 7 (read)
- Config key `total_headcount` used consistently across: Task 1 (written), Task 7 (read)
- Config path `~/.claude/plugins/config/eu-legal/employment.md` consistent across all 7 skills

All consistent. Plan is ready for execution.
