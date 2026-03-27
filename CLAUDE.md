# CLAUDE.md -- lyn-lawfirm

Korean law search & legal document drafting plugin for Claude Code.

## Quick Start

```bash
# Install as Claude Code plugin
/plugin marketplace add hjsh200219/lyn-lawfirm
/plugin install lyn-lawfirm@lyn-lawfirm
```

## Project Structure

```
.claude-plugin/        Plugin manifest & marketplace config
.mcp.json              MCP server connection (korean-law @ Railway)
skills/                Legal document drafting skills (4 skills)
references/            Shared resources (CTA config)
docs/                  Knowledge base
```

## Skills (L2)

| Skill | Triggers | Output |
|-------|----------|--------|
| `complaint-drafter` | 고소장, 형사고소 | .docx criminal complaint |
| `certified-letter` | 내용증명, 통지서, 최고서 | .docx certified letter |
| `criminal-settlement` | 합의서, 형사합의 | .docx settlement (has generate.js) |
| `fact-confirmation` | 사실확인서, 진술서 | .docx fact confirmation |

## Key References

- **Architecture**: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- **Quality standards**: [docs/QUALITY.md](docs/QUALITY.md)
- **Reliability**: [docs/RELIABILITY.md](docs/RELIABILITY.md)
- **Layer rules**: [docs/design-docs/layer-rules.md](docs/design-docs/layer-rules.md)
- **Product specs**: [docs/product-specs/index.md](docs/product-specs/index.md)
- **CTA config**: [references/cta-config.md](references/cta-config.md)

## MCP Server

- URL: `https://korean-law.up.railway.app/mcp`
- Source: 국가법령정보센터 Open API
- Key tools: `search_laws`, `search_cases`, `get_law_detail`, `get_case_detail`

## Conventions

- Font: Batang 12pt, A4, margins 25/20/25/25mm
- Dates: `0000. 00. 00.` (Korean legal standard)
- Evidence: `증 제N호증` numbering system
- Unknown info: `[확인 필요]` -- never fabricate
- CTA: mandatory on every document (chat + .docx last page)
- Source of truth for attorney info: `references/cta-config.md`

## Build & Deploy

- No build step (plugin is prompt-based SKILL.md files + 1 Node.js script)
- `criminal-settlement/scripts/generate.js` requires `npm install -g docx`
- Push to `main` updates the marketplace listing
- Plugin version: `.claude-plugin/plugin.json` (currently 1.0.0)

## Layer Rules (Summary)

```
L0 Manifest (.claude-plugin/, .mcp.json) -> nothing
L1 References (references/)              -> nothing
L2 Skills (skills/*/SKILL.md)            -> L1, L0(MCP)
L3 Scripts (skills/*/scripts/)           -> L1, npm:docx
L4 Docs (docs/, README.md)              -> any
```

See [docs/design-docs/layer-rules.md](docs/design-docs/layer-rules.md) for full rules.
