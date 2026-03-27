# Architecture

## System Overview

lyn-lawfirm is a Claude Code plugin that provides Korean law search (via remote MCP) and legal document drafting (via prompt-based skills). It is a **plugin distribution package**, not a traditional application with runtime servers or databases.

## Component Map

```
lyn-lawfirm/
├── .claude-plugin/            # L0: Plugin & marketplace metadata
│   ├── plugin.json            #   Plugin manifest (name, version, keywords)
│   └── marketplace.json       #   Marketplace listing config
├── .mcp.json                  # L0: MCP server connection (korean-law @ Railway)
├── skills/                    # L2/L3: Legal document drafting skills
│   ├── complaint-drafter/     #   L2: SKILL.md -- criminal complaint (고소장)
│   ├── certified-letter/      #   L2: SKILL.md -- certified letter (내용증명)
│   ├── criminal-settlement/   #   L2: SKILL.md + L3: scripts/generate.js
│   │   ├── SKILL.md
│   │   ├── scripts/generate.js
│   │   └── references/legal-notes.md
│   └── fact-confirmation/     #   L2: SKILL.md -- fact confirmation (사실확인서)
├── references/                # L1: Shared references for all skills
│   └── cta-config.md          #   Attorney CTA info (이영남 변호사)
├── docs/                      # L4: Knowledge base
├── AGENTS.md                  # L4: Agent navigation map
├── CLAUDE.md -> agent.md      # L4: Agent instructions (symlink)
├── agent.md                   # L4: Agent instructions (source)
├── README.md                  # L4: Public-facing documentation
└── LICENSE                    # MIT + legal disclaimer
```

## Data Flow

```
User request (e.g. "고소장 써줘")
  └─> Claude Code matches SKILL.md trigger keywords
        ├─> Skill reads attached files (transcripts, evidence)
        ├─> Skill calls MCP tools (search_laws, search_cases, etc.)
        │     └─> korean-law MCP server (Railway)
        │           └─> 국가법령정보센터 Open API
        ├─> Skill drafts document per SKILL.md template
        └─> Skill generates .docx output
              ├─> complaint-drafter, certified-letter, fact-confirmation:
              │     Claude Code built-in docx-js
              └─> criminal-settlement:
                    scripts/generate.js (Node.js + npm:docx)
              └─> CTA page appended (references/cta-config.md)
```

## Key Boundaries

| Boundary | Owner | Technology | Note |
|----------|-------|------------|------|
| MCP Server | External (Railway) | Node.js HTTP | `https://korean-law.up.railway.app/mcp` |
| Skills | This repo | Prompt (SKILL.md) | 4 skills, keyword-triggered |
| Script | This repo | Node.js | Only `criminal-settlement/scripts/generate.js` |
| CTA Config | This repo | Markdown | `references/cta-config.md` -- single source of truth |
| Plugin Manifest | This repo | JSON | `.claude-plugin/` -- marketplace listing |

## Layer Architecture

```
L0  Plugin Manifest    .claude-plugin/, .mcp.json       -> (nothing)
L1  Shared References  references/                       -> (nothing)
L2  Skills             skills/*/SKILL.md                 -> L1, L0(MCP)
L3  Scripts            skills/*/scripts/                 -> L1, npm:docx
L4  Documentation      docs/, README.md, AGENTS.md       -> any
```

See [docs/design-docs/layer-rules.md](docs/design-docs/layer-rules.md) for full dependency rules.

## Runtime Dependencies

| Dependency | Type | Used By | Purpose |
|-----------|------|---------|---------|
| `docx` (npm) | External | `criminal-settlement/scripts/generate.js` | .docx file generation |
| Node.js | Runtime | `generate.js` | Script execution |
| Claude Code | Host | All skills | Plugin host and docx-js provider |
| korean-law MCP | Remote service | All skills | Law/case data lookup |

## Conventions

- All legal documents: A4, Batang font, 12pt base
- Dates: `0000. 00. 00.` format (Korean legal standard)
- Evidence numbering: `증 제N호증` system
- CTA mandatory on every document (chat + .docx last page)
- Unknown information: `[확인 필요]` -- never fabricated
