# Design Overview

## Philosophy

lyn-lawfirm is designed as a **prompt-first plugin** for Claude Code. Rather than building a traditional application with servers, databases, and UI, it leverages Claude Code's plugin system to deliver legal document drafting through structured prompts (SKILL.md files) and a remote MCP server for Korean law data.

## Architecture Decisions

### Why Prompt-Based Skills?

Legal document templates are inherently text-heavy with complex conditional logic (escalation criteria, special clauses, formatting rules). SKILL.md files encode this logic as structured instructions that Claude follows, rather than imperative code. This makes templates auditable by legal professionals who are not programmers.

**Exception**: `criminal-settlement` has a `generate.js` script because settlement agreements require dynamic clause numbering, installment tables, and signature tables that benefit from programmatic `.docx` generation.

### Why Remote MCP?

Korean law data (statutes, case law, legal terms) is large and frequently updated. Hosting a dedicated MCP server on Railway allows the plugin to access current legal data without bundling it. The MCP server wraps the National Law Information Center (국가법령정보센터) Open API.

### Why Centralized CTA?

Attorney contact information and disclaimers appear in every generated document. Storing this in a single file (`references/cta-config.md`) prevents drift and makes updates atomic.

## Key Design Patterns

| Pattern | Where | Why |
|---------|-------|-----|
| Template Method | Each SKILL.md follows a multi-step workflow | Ensures consistent process across document types |
| Single Source of Truth | `references/cta-config.md` | Prevents attorney info duplication |
| Escalation Rules | Each SKILL.md has explicit thresholds | Legal safety net for edge cases |
| Layered Architecture | L0-L4 layers with strict dependency rules | Modularity and isolation |
| Fallback Values | `generate.js` defaults to `○○○` | Documents are always renderable even with missing info |

## Document Lifecycle

```
1. User triggers skill (keyword match)
2. Skill collects information (from user + attached files)
3. Skill verifies legal references (MCP: search_laws, search_cases)
4. Skill drafts document (following SKILL.md template)
5. Skill generates .docx (docx-js or generate.js)
6. CTA appended (from references/cta-config.md)
7. User receives: chat guidance + .docx file
```

## References

- [Architecture](ARCHITECTURE.md) -- Component map and data flow
- [Core Beliefs](design-docs/core-beliefs.md) -- Fundamental principles
- [Layer Rules](design-docs/layer-rules.md) -- Dependency constraints
- [Quality Standards](QUALITY.md) -- Formatting and accuracy requirements
