---
name: adversarial-qc
description: "Adversarial quality control for AI deliverables. Run structured parallel verification using one or two models before delivering reports, plans, analyses, scripts, or any substantive work. Use when: (1) reviewing a deliverable before sending to a human, (2) user asks to 'QC this', 'review this', 'check this', 'verify this', (3) user wants a quality gate on AI output, (4) validating factual claims, numbers, logic, or completeness. Configurable: single-model (two agents, same model) or cross-model (two different providers). Outputs a QC certificate with pass/fail/review items and evidence."
---

# Adversarial QC

Structured quality control for AI deliverables. Two agents independently verify a deliverable against a checklist, then results are compared. Agreements = high confidence. Disagreements = flagged for human review.

## Quick Start

1. Read the deliverable to be reviewed
2. Read `references/checklist.md` for the verification checklist
3. Read `references/config.md` for model configuration
4. Run the QC process (see Workflow below)
5. Generate the QC certificate

## Configuration

The user controls these settings. Ask if not specified:

- **Mode**: `single` (one model, two agent personas) or `cross` (two different models)
- **Models**: Which model(s) to use. Defaults: primary = current session model, secondary = user's choice
- **Checklist**: `standard` (general-purpose) or a custom checklist path
- **Output**: `certificate` (PDF), `inline` (text summary), or `both`
- **Depth**: `quick` (5-item core checklist, ~30s), `standard` (full checklist, ~2min), `deep` (full checklist + source re-verification, ~5min)

## Workflow

### Step 1: Ingest

Read the deliverable. Identify its type (report, script, plan, analysis, email, legal document, other). This determines which checklist items are relevant.

### Step 2: Run Agent A (Verifier)

Spawn a sub-agent with these instructions:

```
You are QC Agent A — a verification specialist. Your job is to check a deliverable against a structured checklist. You must be thorough, skeptical, and evidence-based.

RULES:
- Every finding must include EVIDENCE (command output, source quote, or specific reasoning)
- "I think" is not evidence — verify or mark UNVERIFIED
- Do not assume the deliverable is correct — assume it contains errors until proven otherwise
- Be specific: "Line 14 claims X but source says Y" not "some numbers seem off"

DELIVERABLE:
[Insert deliverable text]

CHECKLIST:
[Insert from references/checklist.md — only items relevant to deliverable type]

For each checklist item, output:
- PASS: [item] — [evidence it's correct]
- FAIL: [item] — [what's wrong + evidence]
- REVIEW: [item] — [unable to verify, reason, suggested human check]

End with a summary: X pass, Y fail, Z review.
```

### Step 3: Run Agent B (Challenger)

Spawn a second sub-agent (different model if cross-mode). Same instructions but with this addition:

```
You are QC Agent B — an independent challenger. You are running the same checklist as Agent A but you have NOT seen Agent A's results. Your job is to find what Agent A might miss.

Pay special attention to:
- Claims that SOUND correct but aren't verified
- Numbers that are plausible but unchecked
- Logic that flows well but has gaps
- Things that would embarrass the author if a domain expert read them
```

If single-model mode: use different system prompts to create genuine perspective difference. Agent A is methodical and checklist-driven. Agent B is adversarial and tries to break things.

### Step 4: Compare Results

For each checklist item:
- **Both PASS** → High confidence. Mark GREEN.
- **Both FAIL** (same issue) → Confirmed error. Mark RED.
- **One PASS, one FAIL** → Disagreement. Mark YELLOW. Include both agents' evidence. Flag for human review.
- **Either REVIEW** → Unverifiable. Mark YELLOW. Flag for human review.

### Step 5: Generate Certificate

Use the template in `references/certificate-template.md`. Output as:
- **Inline**: Text summary in chat (for quick checks)
- **PDF**: Full certificate with evidence (for formal deliverables)
- **Both**: If user wants both

## Checklist Customization

Users can create custom checklists for specific domains. See `references/custom-checklists.md` for examples (legal, financial, technical, editorial).

To use a custom checklist: place a `.qc-checklist.md` file in the project directory, or specify `--checklist path/to/checklist.md`.

## Cost Guidance

| Depth | Single-model | Cross-model |
|-------|-------------|-------------|
| Quick | ~$0.02-0.05 | ~$0.04-0.10 |
| Standard | ~$0.10-0.30 | ~$0.20-0.50 |
| Deep | ~$0.30-1.00 | ~$0.50-1.50 |

Costs assume Sonnet-tier models. Using Opus increases ~5-10x.

## Tips

- For routine checks, `quick` + `single` is fine
- For client-facing deliverables, `standard` + `cross` catches more
- For high-stakes work (legal filings, financial reports), `deep` + `cross` + human review of all YELLOW items
- The certificate is the audit trail — save it alongside the deliverable
