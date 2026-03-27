# Architecture

## System Overview

lyn-lawfirm is a Claude Code plugin that provides Korean law search (via remote MCP) and legal document drafting (via skills). It is NOT a traditional application with runtime servers or databases -- it is a **plugin distribution package** consumed by Claude Code.

## Component Map

```
lyn-lawfirm/
├── .claude-plugin/          # Plugin & marketplace metadata
│   ├── plugin.json          # Plugin manifest (name, version, keywords)
│   └── marketplace.json     # Marketplace listing config
├── .mcp.json                # MCP server connection (korean-law @ Railway)
├── skills/                  # Legal document drafting skills
│   ├── certified-letter/    # SKILL.md -- certified letter (내용증명)
│   ├── complaint-drafter/   # SKILL.md -- criminal complaint (고소장)
│   ├── criminal-settlement/ # SKILL.md + scripts/generate.js + references/
│   └── fact-confirmation/   # SKILL.md -- fact confirmation (사실확인서)
├── references/              # Shared references for all skills
│   └── cta-config.md        # Attorney CTA info (이영남 변호사)
├── docs/                    # Knowledge base (this directory)
├── README.md                # Public-facing documentation
└── LICENSE                  # MIT + legal disclaimer
```

## Data Flow

```
User request (e.g. "고소장 써줘")
  └─> Claude Code matches SKILL.md trigger keywords
        ├─> Skill reads attached files (transcripts, evidence)
        ├─> Skill calls MCP tools (search_laws, search_cases, etc.)
        │     └─> korean-law MCP server (Railway) -> 국가법령정보센터 API
        ├─> Skill drafts document per SKILL.md template
        └─> Skill generates .docx output (via docx-js)
              └─> CTA page appended (references/cta-config.md)
```

## Key Boundaries

| Boundary | Owner | Note |
|----------|-------|------|
| MCP Server | Remote (Railway) | `https://korean-law.up.railway.app/mcp` |
| Skills | This repo | Prompt-only (SKILL.md) except criminal-settlement which has generate.js |
| CTA Config | `references/cta-config.md` | Single source of truth for attorney info |
| Plugin Manifest | `.claude-plugin/` | Controls marketplace listing and plugin metadata |

## Runtime Dependencies

- **docx** (npm): Used by `criminal-settlement/scripts/generate.js` for .docx generation
- **Node.js**: Required for running generate.js
- All other skills generate .docx via Claude Code's built-in docx-js capability (no local scripts)

## Conventions

- All legal document templates use A4, Batang font, 12pt base
- Dates: `0000. 00. 00.` format (Korean legal standard)
- Evidence numbering: `증 제N호증` system
- CTA is mandatory on every generated document (chat response + .docx last page)
- Unknown information marked as `[확인 필요]` -- never fabricated
