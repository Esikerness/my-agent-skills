# My Agent Skills

A public collection of reusable agent skills for Tilda development, commerce integrations, and reliable problem-solving workflows.

These skills help an AI agent inspect existing work carefully, preserve working behavior, make focused changes, and report what was actually verified.

## Included skills

### `tilda-ui-engineering`

Reviews, debugs, and extends custom Tilda Vibe Block HTML, CSS, and JavaScript. It focuses on responsive layouts, accessibility, safe global scripts, visual interactions, motion, and evidence-based browser verification.

### `tilda-commerce-integration`

Audits and improves custom Tilda catalog, cart, pricing, inventory, forms, and B2B checkout integrations. It covers TildaCatalogSDK, `tcart`, ST100, form processing, client/server trust boundaries, data flow, and safe compatibility with native Tilda behavior.

### `adaptive-work-reasoning`

This is a work-focused derivative of [`arc-skill`](https://github.com/pbshgthm/arc-skill). It was adapted at the user's request after asking an AI to rework the original approach for practical work tasks.

The skill applies a compact hypothesis-driven loop: define the expected outcome, separate facts from assumptions, run the smallest useful test, compare evidence, update the working model, and report verified results separately from remaining uncertainty.

## Installation

Install all skills from this repository:

```bash
npx skills add Esikerness/my-agent-skills
```

Install one specific skill globally for Codex:

```bash
npx skills add Esikerness/my-agent-skills \
  --skill tilda-ui-engineering \
  -g \
  -a codex
```

Replace `tilda-ui-engineering` with `tilda-commerce-integration` or `adaptive-work-reasoning` when needed.

## Repository structure

```text
skills/
├── tilda-ui-engineering/
│   └── SKILL.md
├── tilda-commerce-integration/
│   └── SKILL.md
└── adaptive-work-reasoning/
    └── SKILL.md
```

Each skill directory contains one `SKILL.md` file with the required `name` and `description` metadata.

## License

No license has been added yet. Add a license file before treating this repository as a library for redistribution.
