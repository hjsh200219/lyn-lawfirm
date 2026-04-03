# Reliability

## Error Handling

### generate.js (criminal-settlement)
- JSON parsing: supports both inline JSON and file path input
- Fallback values: all fields default to `○○○` / `○○` if missing
- Amount formatting: `formatAmount()` strips duplicate `금` prefix
- Exit codes: 0 on success, 1 on parse failure or generation error

### MCP Server Dependency
- Remote server: `https://korean-law.up.railway.app/mcp`
- If MCP is unreachable, skills should note that law/case verification could not be performed
- Skills should never cite unverified law articles -- mark as `[법령 확인 필요]` if MCP is down

### Document Generation
- Post-generation validation: read back .docx to check for text duplication
- Evidence number consistency: `증 제N호증` in body must match evidence table
- Party info consistency: names must be identical throughout the document
- File save location: always confirm with user before saving; never use `/tmp/` as final destination

## Escalation Criteria

| Skill | Condition | Action |
|-------|-----------|--------|
| complaint-drafter | SOL < 3 months | Warning banner at document top |
| complaint-drafter | 친고죄 deadline imminent | Warning banner at document top |
| complaint-drafter | Amount > 5억원 | Note 특정경제범죄법 applicability |
| complaint-drafter | Weak evidence | Flag 무고 risk |
| certified-letter | Amount > 5,000만원 | Recommend attorney review |
| certified-letter | SOL < 6 months | Recommend attorney review |
| certified-letter | Responding to opponent's letter | Recommend attorney review |
| All skills | Incomplete info | Mark `[확인 필요]` -- never guess |

## Deployment

- Plugin distributed via Claude Code marketplace (`/plugin marketplace add hjsh200219/lyn-lawfirm`)
- MCP server independently hosted on Railway
- No CI/CD pipeline in this repo -- changes are pushed directly to `main`
- Plugin version tracked in `.claude-plugin/plugin.json` (currently 1.1.0)

## Monitoring

- No runtime monitoring (plugin is prompt-based, not a running service)
- MCP server health is managed externally (Railway)
- Post-push: verify marketplace listing reflects latest changes
