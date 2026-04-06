---
name: core-image-information-judgment
description: Practical workflow for judging image content, quality, authenticity signals, and task-fit with concise, evidence-based output.
---

# Image Information Judgment Workflow

Use this workflow when you need to determine **what an image shows**, **how reliable that interpretation is**, and **whether the image is suitable for a downstream task** (classification, moderation, OCR, design review, etc.).

## When to Use

- You need a structured decision instead of a casual caption.
- You must separate **observable facts** from **inference**.
- You need confidence estimates and uncertainty handling.
- You need an auditable output format for product or ops pipelines.

## Core Principles

1. **Evidence first**: Describe visible facts before interpretation.
2. **Layered certainty**: Assign confidence per claim, not once globally.
3. **Task-aware judgment**: “Good image” depends on the target task.
4. **Fail gracefully**: Return “insufficient evidence” when needed.
5. **No hallucination**: Never invent text, logos, identities, or events.

## Step-by-Step Workflow

## 1) Define Task and Decision Criteria

Before looking at details, define the target:

- Goal: e.g. object presence, scene type, safety risk, text extraction readiness.
- Required output: label, boolean, ranked candidates, or report.
- Acceptance thresholds:
  - Minimum confidence (e.g. >= 0.8)
  - Required evidence count
  - Escalation rule for ambiguous cases

Template:

- **Task**:
- **Must decide**:
- **Pass threshold**:
- **Escalate if**:

## 2) Quick Technical Quality Check

Judge whether the image can support reliable analysis:

- Resolution sufficient for target?
- Blur / motion blur?
- Exposure (too dark/bright)?
- Noise / compression artifacts?
- Occlusion / cropping?
- Perspective distortion?

Output:

- `quality_status`: `usable` | `borderline` | `unusable`
- `quality_issues`: list
- `impact_on_task`: short explanation

## 3) Structured Visual Inventory (Facts Only)

Extract only what is directly visible:

- Objects/entities (with approximate locations)
- Human presence (count/pose only; avoid identity claims unless explicit requirement + strong evidence)
- Scene/background
- Text regions (do not assume unreadable text)
- Colors, symbols, logos, marks
- Interactions (holding, touching, overlap)

Good fact style:

- “A red rectangular sign appears in the upper-right area.”
- “Two people are visible; one is seated.”

Bad fact style:

- “They are coworkers in a meeting.” (inference)

## 4) Build Hypotheses and Score Evidence

Convert facts into candidate conclusions.

For each hypothesis:

- Supporting evidence (bullets)
- Contradicting evidence (bullets)
- Missing evidence
- Confidence score (0.0–1.0)
- Risk of misclassification

Example confidence rubric:

- `0.9–1.0`: explicit, multiple clear cues
- `0.7–0.89`: strong cues, minor ambiguity
- `0.4–0.69`: partial cues, significant ambiguity
- `<0.4`: weak support; avoid firm claims

## 5) Manipulation & Authenticity Signals (If Relevant)

Check for non-conclusive but useful signals:

- Inconsistent lighting/shadows
- Edge halos / cutout artifacts
- Repeated texture patterns
- Geometric inconsistencies
- Metadata mismatch (if metadata is available externally)
- Unnatural text rendering

Important: treat these as **signals**, not proof.  
Output as:

- `manipulation_signals`: `[signal, strength, note]`
- `authenticity_conclusion`: `no strong signal` | `possible editing` | `likely edited` (with rationale)

## 6) Task-Fit Judgment

Map analysis to the business/application task:

- For OCR: is text region large/clear enough?
- For moderation: are policy-relevant elements unambiguous?
- For classification: does dominant subject fill enough frame?
- For UX/design: is key UI legible and hierarchy visible?

Return:

- `task_fit`: `fit` | `partial_fit` | `not_fit`
- `blocking_reasons`: list
- `recommended_next_action`: retake, crop, zoom, request metadata, human review

## 7) Final Decision with Uncertainty

Make a bounded decision:

- Primary conclusion
- Confidence
- Top alternatives (if ambiguity exists)
- What additional evidence would change the result

## Output Format (Recommended)

Use a stable schema for automation and auditability.

```/dev/null/image-analysis-output.json#L1-33
{
  "task": "detect whether image contains a traffic stop sign",
  "quality": {
    "status": "usable",
    "issues": ["mild compression artifacts"],
    "impact_on_task": "does not block sign-shape recognition"
  },
  "observations": [
    "Octagonal red sign appears near center-left",
    "White border and white central text-like region visible",
    "Urban roadside background"
  ],
  "hypotheses": [
    {
      "label": "stop_sign_present",
      "confidence": 0.86,
      "support": [
        "octagonal shape",
        "red color with white border",
        "roadside context"
      ],
      "contradictions": [],
      "missing_evidence": ["full text not perfectly legible"]
    }
  ],
  "authenticity": {
    "signals": [],
    "conclusion": "no strong signal"
  },
  "task_fit": {
    "status": "fit",
    "blocking_reasons": [],
    "recommended_next_action": "accept"
  },
  "final": {
    "decision": "yes",
    "confidence": 0.86,
    "alternatives": [
      {"label": "other_red_sign", "confidence": 0.12}
    ],
    "needs_human_review": false
  }
}
```

## Practical Heuristics

- Prefer **specific** wording over generic:
  - Better: “logo-like circular mark on lower-right corner”
  - Worse: “some branding”
- Use **bounded language**:
  - “likely”, “possibly”, “insufficient evidence”
- Separate:
  - `observed_text`: what can be read directly
  - `inferred_meaning`: interpretation of that text
- If faces are present and identity is not required, avoid identity speculation.
- If critical decision + low confidence, force human review.

## Common Failure Modes

- Overconfident conclusions from low-quality images
- Mixing inference into observation layer
- Ignoring contradictory cues
- Treating editing signals as definitive proof
- Using one global confidence for all claims

## Minimal Checklist (Fast Path)

- [ ] Task + threshold defined
- [ ] Quality status set
- [ ] Facts listed without inference
- [ ] At least one alternative hypothesis considered
- [ ] Confidence assigned with rationale
- [ ] Task-fit and next action provided

## Suggested Confidence Threshold Policy

- `>= 0.85`: auto-accept
- `0.60–0.84`: accept with caution or secondary check
- `< 0.60`: escalate to human review

Tune thresholds by domain risk (safety/legal/medical should be stricter).

<!--
Source references:
- Internal best-practice synthesis for image information judgment workflows
-->