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
- [docs/plans/ai-law-firm.md](docs/plans/ai-law-firm.md) -- AI 법무법인 multi-agent architecture (RALPLAN consensus)
- [docs/plans/ai-law-firm-open-questions.md](docs/plans/ai-law-firm-open-questions.md) -- AI 법무법인 open decision points
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


## TDD 필수

모든 새 기능/로직 변경은 반드시 TDD로 개발한다.
1. Red: 실패하는 테스트 먼저 작성
2. Green: 테스트를 통과하는 최소 코드 작성
3. Refactor: 코드 정리
테스트 없는 코드 변경은 허용하지 않는다.

---

## Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

Tradeoff: These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

These guidelines are working if: fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## 세션 시작 시 Handoff 강제

세션을 시작할 때 프로젝트 루트에 `handoff.md` 파일이 있는지 먼저 확인한다.
- `handoff.md`가 존재하면 다른 어떤 작업보다 먼저 **반드시 전체를 읽고 인수인계 컨텍스트를 파악한 뒤 시작**한다.
- 파일이 없으면 정상 진행한다.

이 규칙은 이전 세션의 미완료 작업·결정 사항·주의사항을 놓치지 않기 위한 강제 사항이다.

**이 프로젝트의 handoff 위치**: 없음 (생성 시 `.claude-project/HANDOFF.md` 권장)
