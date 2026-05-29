# fi-startup-legal + MCP Servers — Design Spec

**Date:** 2026-05-29
**Author:** Matti Puhakka / Silly Pilot Oy
**Status:** Approved for implementation planning

---

## Problem

Finnish startup founders and early ops hires lack practical, Finnish-law-specific legal tooling. Generic tools (Docue, Lexly) provide templates but no reasoning layer. General AI gives plausible-sounding but potentially outdated or hallucinated Finnish statute references. The 2026 TVL §66 option tax reform, the seriesseed.fi SHA standard, and Business Finland IP clawback rules require current, verified Finnish legal data — not training-data guesses.

Additionally, eu-legal's employment and corporate skills currently rely on model training data for Finnish statute citations (TSL notice periods, OYL provisions), which introduces hallucination risk for specific article numbers and thresholds.

## Goal

Build `fi-startup-legal`: a Claude Code plugin for Finnish startup founders and first ops hires, covering the pre-seed through Series A journey. Backed by two new MCP servers (`finlex-mcp`, `prh-mcp`) that provide live Finnish statute and company register data. Retrofit the same MCP servers into eu-legal's employment and corporate skills to eliminate training-data dependency for Finnish law citations.

---

## Target user

**Primary:** Finnish startup founder (pre-legal hire, A/B profile) — practical output, plain language, generous guardrails, clear "talk to a lawyer" gates. Not a lawyer, but smart. Needs to understand what they're doing, not just receive a template.

**Secondary:** Startup's first ops/finance hire handling legal admin.

**Stage:** Pre-seed through Series A. Pre-incorporation through post-round entity management.

**Not the target:** Law firms (eu-legal + future law-firm plugin handles them), EU financial institutions (eu-legal).

---

## Architecture

### New MCP servers (prerequisites for fi-startup-legal)

Both hosted at `mcp.velvoite.eu`. No API key required — public Finnish data APIs. Velvoite hosts and manages rate limiting.

#### `finlex-mcp`

Wraps the Finlex API (finlex.fi/api/) — Finland's official statute database.

**Tools:**

| Tool | Parameters | Returns |
|---|---|---|
| `search_statutes` | `query: string, language?: "fi"\|"en"` | Array of matching statute sections: `{law_name, law_code, section, text_excerpt, url}` |
| `get_statute` | `law_code: string, section?: string` | Full text of statute or specific section: `{law_name, law_code, section, full_text, last_amended, url}` |

**Key law codes used by plugins:**

| Code | Act | Used by |
|---|---|---|
| `TSL` | Employment Contracts Act (Työsopimuslaki 55/2001) | eu-legal employment, fi-startup first-hire |
| `OYL` | Companies Act (Osakeyhtiölaki 624/2006) | fi-startup incorporation, eu-legal corporate |
| `YTL` | Co-operation Act (YT-laki 1333/2021) | eu-legal employment |
| `TVL` | Income Tax Act (Tuloverolaki 1535/1992) | fi-startup ESOP (§66, §66a) |
| `LSL` | Business Secrets Act (Liikesalaisuuslaki 595/2018) | fi-startup NDA, eu-legal commercial |
| `VLL` | Annual Holidays Act (Vuosilomalaki 162/2005) | eu-legal employment |
| `TaL` | Working Hours Act (Työaikalaki 872/2019) | eu-legal employment |
| `HeTiL` | Data Protection Act (Tietosuojalaki 1050/2018) | fi-startup GDPR basics |

**Deployment:** Python FastAPI server wrapping finlex.fi/api/. Hosted at `https://finlex-mcp.velvoite.eu/mcp`. HTTP transport. No auth.

#### `prh-mcp`

Wraps PRH OpenData API (opendata.prh.fi) — Finnish Trade Register.

**Tools:**

| Tool | Parameters | Returns |
|---|---|---|
| `lookup_company` | `business_id: string` | `{name, business_id, status, registered, address, board_members[], auditor, share_capital, fiscal_year_end}` |
| `search_companies` | `name: string, city?: string` | Array of `{name, business_id, status, municipality}` |

**Deployment:** Python FastAPI server wrapping opendata.prh.fi. Hosted at `https://prh-mcp.velvoite.eu/mcp`. HTTP transport. No auth.

---

### `fi-startup-legal` plugin

Single plugin. Same architecture as eu-legal. Config at `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md`.

**`.mcp.json` servers:**
- `finlex` — `https://finlex-mcp.velvoite.eu/mcp` (no auth)
- `prh` — `https://prh-mcp.velvoite.eu/mcp` (no auth)
- `velvoite` — `https://mcp.velvoite.eu/mcp` (optional, `VELVOITE_API_KEY`)
- `Slack` — for notifications
- `Google Drive` — for document storage

**MCP tool prefixes in skills:**
- `mcp__finlex__search_statutes`, `mcp__finlex__get_statute`
- `mcp__prh__lookup_company`, `mcp__prh__search_companies`
- `mcp__velvoite__*` (optional depth)

---

## Plugin profile

Written by `startup-cold-start` to `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md`.

Captures:
- **Company name** + **business ID** (if incorporated) — PRH lookup triggered on setup
- **Stage:** pre-incorporation / incorporated (pre-seed) / post-seed / Series A
- **Founders:** names, roles, equity split (or "not decided yet")
- **Primary jurisdiction:** Finland (fixed) + any operating countries
- **Business Finland grants active:** yes/no — triggers BF compliance awareness in relevant skills
- **AI/tech startup flag:** yes/no — triggers AI Act + GDPR depth via Velvoite (if key present)
- **Velvoite API key status:** ✓/✗ — for GDPR/AI Act depth
- **Outside counsel:** law firm name if any (Fondia / Nordic Law / Dottir / Lexia / other / none)

---

## Skills inventory (15)

### Onboarding
| Skill | What it does |
|---|---|
| `startup-cold-start` | Setup interview → writes profile. PRH lookup if business ID provided. Probes Finlex/PRH/Velvoite connectivity. Offers first actions based on stage. |

### Incorporation
| Skill | What it does | Key data |
|---|---|---|
| `oy-setup` | OY incorporation wizard — yhtiöjärjestys (articles) + osakassopimus (SHA) checklist + PRH registration steps | `mcp__prh__search_companies` to check name availability; `mcp__finlex__get_statute("OYL", "2:1")` for share capital rules (€0 min since 2019); board EEA residency requirement |

### Founder mechanics
| Skill | What it does | Key data |
|---|---|---|
| `founder-agreement` | Co-founder agreement wizard — equity split, roles, vesting (Finnish good/bad/early leaver), IP assignment, non-compete validity check | `mcp__finlex__get_statute("TSL", "3:5")` for non-compete max 12 months + compensation requirement |
| `ip-assignment` | IP assignment audit — checks all founders, employees, and contractors have assigned IP to the company. Flags gaps. Business Finland IP restriction check if BF grants active. | `mcp__finlex__get_statute("TSL", "7")` for employee invention rules (Laki oikeudesta työntekijän tekemiin keksintöihin) |

### Investment & cap table
| Skill | What it does | Key data |
|---|---|---|
| `sha-review` | SHA review against seriesseed.fi three-tier leaver standard — good/early/bad leaver definitions, pre-emption rights, drag-along, tag-along, ROFR, anti-dilution, board composition | `mcp__finlex__get_statute("OYL", "3")` for share transfer restrictions |
| `term-sheet-review` | Term sheet review — valuation mechanics, liquidation preference (1× non-participating standard), pro-rata rights, information rights, board seat, protective provisions, ESOP pool size | Plain language explanation of each term for non-lawyer founder |

### ESOP
| Skill | What it does | Key data |
|---|---|---|
| `esop-designer` | Option scheme design — §66 vs §66a decision tool (2026 reform: §66 now taxed at sale not exercise), AGM approval requirements, typical Finnish vesting (4-year / 1-year cliff), option pool sizing | `mcp__finlex__get_statute("TVL", "66")` + `mcp__finlex__get_statute("TVL", "66a")` for live tax rules |
| `vesting-calculator` | Vesting schedule generator — cliff + monthly vesting, good/early/bad leaver treatment, acceleration (single/double trigger), exercise window (Finnish standard: 10 years) | Finnish-specific: share subscription vs. option mechanics under OYL |

### Grants & public funding
| Skill | What it does | Key data |
|---|---|---|
| `bf-grant-check` | Business Finland IP compliance check — 5-year EEA control transfer restriction, IP retention requirement (same Business ID), clawback trigger assessment for M&A or restructuring | BF funding terms reference; flags if planned transaction triggers clawback |

### Employment
| Skill | What it does | Key data |
|---|---|---|
| `first-hire-contract` | First employment contract — mandatory terms under TSL, trial period (max 6 months), notice period (statutory vs contractual), non-compete validity, IP assignment clause | `mcp__finlex__get_statute("TSL", "1:4")` for mandatory terms; `mcp__finlex__get_statute("TSL", "3:5")` for non-compete |
| `non-compete-check` | Validates a non-compete clause against Finnish law — max 12 months, compensation required if >6 months (50% or 100% of salary depending on restriction), reasonableness | `mcp__finlex__get_statute("TSL", "3:5")` live |

### Commercial basics
| Skill | What it does | Key data |
|---|---|---|
| `customer-contract` | First customer contract checklist / review — liability cap, IP ownership (we retain), GDPR data processing clause if handling customer data, governing law (Finnish law default), SLA | Routes to Velvoite GDPR if data processing involved |
| `startup-nda` | NDA review and generation — mutual vs one-way, EU Trade Secrets Directive framing (LSL 595/2018), duration, scope | `mcp__finlex__get_statute("LSL", "2")` for trade secret definition |

### Governance
| Skill | What it does | Key data |
|---|---|---|
| `board-minutes` | Startup board minutes — OYL-compliant format, quorum check, decision types requiring board vs shareholder approval, signature requirements | `mcp__finlex__get_statute("OYL", "6")` for board rules |

### Privacy & AI (Velvoite-optional)
| Skill | What it does | Key data |
|---|---|---|
| `gdpr-basics` | Minimum GDPR compliance for startups — privacy notice, cookie consent, first DPA with processors, DSAR process. Routes to Velvoite for DPIA if data-intensive. | Velvoite optional; base guidance from model + HeTiL |
| `ai-act-startup` | AI Act for AI startups — risk classification for the product being built (are you a provider?), high-risk Annex III check, transparency obligations for limited-risk, FRIA if deploying high-risk | Velvoite for full obligation corpus; `mcp__finlex__get_statute("HeTiL")` for Finnish GDPR implementation |

---

## eu-legal retrofit

Add `finlex-mcp` and `prh-mcp` to eu-legal's `.mcp.json`. Update these existing skills to call Finlex instead of relying on training data:

| eu-legal skill | Current data source | After retrofit |
|---|---|---|
| `termination-review` | Training data for TSL notice periods | `mcp__finlex__get_statute("TSL", "6:3")` |
| `works-council-check` | Training data for YT-laki thresholds | `mcp__finlex__get_statute("YTL", "51")` |
| `leave-review` | Training data for Vuosilomalaki | `mcp__finlex__get_statute("VLL")` |
| `worker-classification` | Training data for TSL employment test | `mcp__finlex__get_statute("TSL", "1:1")` |
| `entity-compliance` | Training data for OYL/PRH filings | `mcp__prh__lookup_company(business_id)` |
| `nda-review` | Training data for LSL | `mcp__finlex__get_statute("LSL", "2")` |

---

## Implementation phases

| Phase | What | Est. size |
|---|---|---|
| **Phase 1** | `finlex-mcp` server (Python FastAPI) | Small (~3h) |
| **Phase 2** | `prh-mcp` server (Python FastAPI) | Small (~2h) |
| **Phase 3** | `fi-startup-legal` plugin (15 skills) | Large (~1 day) |
| **Phase 4** | eu-legal retrofit (6 skill updates + .mcp.json) | Small (~2h) |

---

## Open questions for legal review (Heidi)

1. **§66 vs §66a decision logic** — does the 2026 TVL reform (options taxed at sale) apply to all Annex-III-type unlisted companies, or only startups meeting specific criteria? The `esop-designer` skill needs to get this exactly right.
2. **BF grant clawback trigger** — does a VC investment that brings a foreign LP to >50% indirect ownership trigger the EEA control transfer restriction, or only direct control changes?
3. **OYL §9:14 and drag-along** — is drag-along valid in Finnish law without explicit yhtiöjärjestys provision, or does it require a specific share class structure?
4. **Non-compete compensation threshold** — TSL §3:5 requires compensation for restrictions >6 months. What is the current correct threshold (50% or 100% of salary) after recent case law?

---

## Out of scope (v1)

- Cross-border IP (PCT patents, EUIPO trademark)
- Convertible note / SAFE mechanics (too instrument-specific for v1)
- Funding round closing checklist (Phase 2)
- Term sheet generation (review only in v1)
- Multi-founder equity dispute resolution
