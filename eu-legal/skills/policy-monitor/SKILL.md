---
name: policy-monitor
description: >
  Detect drift between your privacy policy commitments and actual practice.
  Two modes: (1) query — check whether a specific new practice is covered by
  the current policy; (2) sweep — review recent privacy activities for policy
  drift. Use when asking "does our policy cover this", "we want to start doing
  X — does the policy need updating", or "run the policy monitor".
argument-hint: "[describe new practice, or 'sweep' for a full review]"
---

# /eu-legal:policy-monitor

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Load `~/.claude/plugins/config/eu-legal/privacy.md` for privacy policy commitments. If missing, note: "Run `/eu-legal:privacy-cold-start` to record your privacy policy commitments."
3. Determine mode: query (specific new practice) or sweep (general drift check).
4. Run the workflow below.

---

## Purpose

Privacy policies are legal commitments under GDPR Art. 13/14 (transparency obligations). If practice diverges from policy without updating the policy, the controller is in breach. This skill finds that gap.

## Mode 1: Query — specific new practice

The user describes something new they want to start doing.

Step 1: Identify what GDPR Art. 13/14 would require in the privacy notice for this processing (purposes, legal basis, recipients, transfers, retention, rights).

Step 2: Compare against the privacy policy commitments in privacy.md. Does the policy already cover this? Specifically:
- Is the purpose disclosed?
- Is the legal basis stated?
- Are any new recipients or categories of recipients disclosed?
- Are new transfer destinations covered (adequacy / SCC)?
- Is the retention period consistent with what's stated?

Step 3: Verdict:
- **Covered** — practice is within current policy; proceed with standard controls
- **Gap** — policy needs updating before the new practice begins; draft the additional language
- **Major gap** — significant new processing category; full policy review required before proceeding

## Mode 2: Sweep

Review the last significant privacy activities (from context or by asking the user) for drift.

For each activity: was the privacy notice updated? Is the processing documented in a record of processing activities (RoPA)? Is the legal basis still valid (particularly for consent-based processing: was consent obtained; can users withdraw)?

GDPR Art. 30 RoPA check: if more than 250 employees, or high-risk processing — is a current RoPA maintained?

## Guardrail

Identified gaps require attorney review and privacy notice update before the new processing begins. Publishing an inaccurate privacy notice is a violation of GDPR Art. 13/14; supervisory authorities regularly fine for transparency failures.
