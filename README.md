# Image Analysis Skill Collection

A focused collection centered on one practical Agent Skill: **image-analysis**.

This repository is designed for scenarios where you need to analyze images and make structured judgments, such as:

- identifying objects/scenes
- extracting visible facts from images
- producing evidence-based conclusions
- reporting confidence and uncertainty
- deciding whether human review is required

## Installation

Install this skill collection:

```/dev/null/install.sh#L1-1
pnpx skills add CrombastiC/skills --skill='image-analysis'
```

Or install globally:

```/dev/null/install-global.sh#L1-1
pnpx skills add CrombastiC/skills --skill='image-analysis' -g
```

Learn more about the CLI at [skills](https://github.com/vercel-labs/skills).

## Included Skill

| Skill | Description |
|-------|-------------|
| [image-analysis](skills/image-analysis) | Structured workflow for image information judgment (facts → hypotheses → confidence → final decision). |

## What This Skill Helps With

The `image-analysis` skill emphasizes **reliable decision-making**, not just image captioning:

- Task definition and acceptance thresholds
- Image quality gating (`usable` / `borderline` / `unusable`)
- Fact-only visual inventory before inference
- Hypothesis scoring with supporting/contradicting evidence
- Confidence-based output and escalation rules
- Task-fit evaluation (OCR/moderation/classification/review pipelines)

## Repository Structure

```mySkill/skills/README.md#L47-58
.
├── skills/
│   └── image-analysis/
│       ├── SKILL.md
│       └── references/
│           └── core-image-information-judgment.md
├── scripts/
│   └── cli.ts
├── meta.ts
└── AGENTS.md
```

## Usage Notes

When applying this skill in production-like flows:

1. Separate **observations** from **inferences**.
2. Attach evidence for each key conclusion.
3. Include confidence and uncertainty constraints.
4. Escalate to human review when confidence is low or risk is high.

## Development

Install dependencies:

```/dev/null/dev-install.sh#L1-1
pnpm install
```

Useful commands:

```/dev/null/dev-commands.sh#L1-4
pnpm start check
pnpm start cleanup
pnpm lint
pnpm start
```

## License

This repository is licensed under [MIT](LICENSE.md).
