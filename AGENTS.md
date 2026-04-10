# AGENTS.md -- lyn-lawfirm

Korean law search & legal document drafting plugin for Claude Code.

## Quick Start

```bash
/plugin marketplace add hjsh200219/lyn-lawfirm
/plugin install lyn-lawfirm@lyn-lawfirm
```

## Quick Orientation

This is a **Claude Code plugin** -- NOT a traditional application.
No servers, no databases, no frontend.
Skills are prompt-based (SKILL.md files) with one Node.js script for .docx generation.

## Repository Map

```
.claude-plugin/           Plugin manifest & marketplace config
.mcp.json                 MCP server connection (korean-law @ Railway)
skills/                   Legal document drafting & analysis skills (6 skills)
  complaint-drafter/      Criminal complaint (고소장) -- SKILL.md only
  certified-letter/       Certified letter (내용증명) -- SKILL.md only
  criminal-settlement/    Settlement agreement (형사합의서) -- SKILL.md + generate.js
  fact-confirmation/      Fact confirmation (사실확인서) -- SKILL.md only
  sentencing-predictor/   Sentencing prediction (양형 예측) -- SKILL.md only
  criminal-procedure-simulator/ Criminal procedure simulation (형사절차 시뮬레이션) -- SKILL.md only
references/               Shared resources
  cta-config.md           Attorney CTA info (single source of truth)
docs/                     Knowledge base and documentation
```

## Skills (L2)

| Skill | Triggers | Output |
|-------|----------|--------|
| `complaint-drafter` | 고소장, 형사고소 | .docx criminal complaint |
| `certified-letter` | 내용증명, 통지서, 최고서 | .docx certified letter |
| `criminal-settlement` | 합의서, 형사합의 | .docx settlement (has generate.js) |
| `fact-confirmation` | 사실확인서, 진술서 | .docx fact confirmation |
| `sentencing-predictor` | 양형, 양형예측, 형량 | sentencing prediction with IRAC analysis |
| `criminal-procedure-simulator` | 형사절차, 수사절차 | interactive procedure simulation |

## Documentation Index

### Architecture & Design
- [ARCHITECTURE.md](ARCHITECTURE.md) -- System overview, component map, data flow
- [docs/DESIGN.md](docs/DESIGN.md) -- Design philosophy and key decisions
- [docs/design-docs/core-beliefs.md](docs/design-docs/core-beliefs.md) -- Fundamental principles
- [docs/design-docs/layer-rules.md](docs/design-docs/layer-rules.md) -- L0-L4 dependency rules
- [docs/design-docs/index.md](docs/design-docs/index.md) -- Design docs index

### Quality & Reliability
- [docs/QUALITY.md](docs/QUALITY.md) -- Legal accuracy and formatting standards
- [docs/QUALITY_SCORE.md](docs/QUALITY_SCORE.md) -- Scoring rubric for generated documents
- [docs/RELIABILITY.md](docs/RELIABILITY.md) -- Error handling, escalation, deployment
- [docs/SECURITY.md](docs/SECURITY.md) -- Threat model and data handling

### Product & Planning
- [docs/PRODUCT_SENSE.md](docs/PRODUCT_SENSE.md) -- Users, problems, constraints
- [docs/product-specs/index.md](docs/product-specs/index.md) -- Skills and resources overview
- [docs/PLANS.md](docs/PLANS.md) -- Roadmap and improvement plans
- [docs/FRONTEND.md](docs/FRONTEND.md) -- No frontend; .docx output formatting

### Data & Tech Debt
- [docs/generated/db-schema.md](docs/generated/db-schema.md) -- generate.js input schema (no DB)
- [docs/exec-plans/tech-debt-tracker.md](docs/exec-plans/tech-debt-tracker.md) -- Known debt items

### References
- [references/cta-config.md](references/cta-config.md) -- Attorney info and CTA templates
- [skills/criminal-settlement/references/legal-notes.md](skills/criminal-settlement/references/legal-notes.md) -- Case-type legal notes

## MCP Server

- URL: `https://korean-law.up.railway.app/mcp`
- Source: 국가법령정보센터 Open API

| Tool | Purpose |
|------|---------|
| `search_laws` | Search Korean statutes by name or text |
| `search_cases` | Search court decisions |
| `get_law_detail` | Get full text of a specific law |
| `get_case_detail` | Get full court decision text |
| `search_interpretations` | Search legal interpretations |

## Key Rules for Agents

1. **Never fabricate** -- use `[확인 필요]` for unknown information
2. **Always include CTA** -- in both chat response and .docx last page
3. **Verify law citations** via MCP before including them
4. **Respect layer rules** -- L0-L4 dependency constraints (see layer-rules.md)
5. **Single source of truth** -- attorney info comes from `references/cta-config.md`
6. **Korean legal formatting** -- Batang 12pt, A4, `0000. 00. 00.` dates, `증 제N호증`
7. **Escalate edge cases** -- SOL < 3 months, amount > 5 billion KRW, public officials

## Conventions

- Font: Batang 12pt, A4, margins 25/20/25/25mm
- Dates: `0000. 00. 00.` (Korean legal standard)
- Evidence: `증 제N호증` numbering system
- Unknown info: `[확인 필요]` -- never fabricate
- CTA: mandatory on every document (chat + .docx last page)
- Source of truth for attorney info: `references/cta-config.md`

## Layer Rules

```
L0 Manifest    (.claude-plugin/, .mcp.json)    -> nothing
L1 References  (references/)                    -> nothing
L2 Skills      (skills/*/SKILL.md)              -> L1, L0(MCP)
L3 Scripts     (skills/*/scripts/)              -> L1, npm:docx
L4 Docs        (docs/, README.md, AGENTS.md)    -> any
```

See [docs/design-docs/layer-rules.md](docs/design-docs/layer-rules.md) for full rules.

## Build & Deploy

- No build step (plugin is prompt-based SKILL.md files + 1 Node.js script)
- `criminal-settlement/scripts/generate.js` requires `npm install -g docx`
- Push to `main` updates the marketplace listing
- Plugin version: `.claude-plugin/plugin.json` (currently 1.1.0)

> Be concise. No filler. Straight to the point. Use fewer words.
