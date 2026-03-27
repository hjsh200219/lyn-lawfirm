# Product Sense

## Who Uses This

### Primary Users
- **Korean attorneys and legal professionals** who need to draft legal documents quickly
- Use Claude Code with this plugin to generate first-draft legal documents from consultation notes, evidence files, and client conversations

### Secondary Users
- **Individuals** handling simple legal matters (e.g., certified letters for lease disputes, fact confirmations)
- Non-lawyers who need properly formatted legal documents but will have them reviewed by a professional

## What Problems It Solves

1. **Time-consuming drafting**: Criminal complaints, certified letters, and settlement agreements follow predictable patterns. The plugin automates the boilerplate while preserving legal precision.
2. **Format compliance**: Korean legal documents have strict formatting requirements (paper size, margins, font, evidence numbering). The plugin enforces these automatically.
3. **Law verification**: Citing the wrong statute or missing an aggravated punishment provision is a serious error. The plugin verifies legal references through MCP before including them.
4. **Evidence organization**: Automatically numbers and cross-references evidence items (`증 제N호증`).

## Key Product Constraints

- **AI disclaimer is mandatory**: Every document clearly states it is an AI-generated draft. The CTA page cannot be omitted.
- **No legal advice**: The plugin drafts documents; it does not provide legal counsel.
- **Korean law only**: The MCP server covers Korean statutes, case law, and legal terminology. Other jurisdictions are not supported.
- **Verification-first**: If a law citation cannot be verified via MCP, it is marked as unverified rather than included as-is.

## Success Metrics

- Document passes attorney review with minimal edits
- All law/case citations are accurate and current
- Formatting matches law firm standards (no manual corrections needed)
- CTA is always present and correctly formatted
- No fabricated information in generated documents

## Competitive Landscape

- Generic AI document generators lack Korean legal formatting knowledge
- Traditional legal document software requires manual input for every field
- This plugin combines conversational input collection with automated formatting and real-time law verification
