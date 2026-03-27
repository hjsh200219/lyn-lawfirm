# Security

## Threat Model

### Data Sensitivity
This plugin handles **sensitive legal information**: party names, addresses, dates of birth, case numbers, financial amounts, and descriptions of criminal incidents. All processing occurs within the user's local Claude Code session -- no data is sent to external services except MCP law search queries.

### MCP Server
- **Endpoint**: `https://korean-law.up.railway.app/mcp` (HTTPS)
- **Data sent**: Law search queries (statute names, case numbers, legal terms)
- **Data NOT sent**: User personal information, party details, document content
- **Risk**: Low. MCP queries contain only public legal reference lookups, not case-specific sensitive data.

### Generated Documents
- `.docx` files are saved locally on the user's machine
- File paths are confirmed with the user before saving
- Files must NOT be saved to `/tmp/` as the final destination (may be cleaned up by OS)

## Access Control

- Plugin is distributed via Claude Code marketplace (public)
- No authentication required for MCP law search (public legal data)
- Attorney contact info in `references/cta-config.md` is intentionally public (marketing CTA)

## Secret Management

- No API keys, tokens, or secrets in this repository
- MCP server authentication (if any) is managed by the server host (Railway)
- The `.gitignore` excludes `.omc/` and `.DS_Store`

## Compliance

- **AI Disclaimer**: Every generated document includes a mandatory disclaimer stating it is AI-generated
- **No Legal Advice**: The plugin explicitly does not provide legal advice; it generates document drafts
- **License**: MIT with additional legal disclaimer in LICENSE file

## Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Incorrect law citation | High | MCP verification before citation; `[법령 확인 필요]` for unverified |
| Fabricated information | High | `[확인 필요]` for unknowns; never guess |
| MCP server downtime | Medium | Skills note when verification could not be performed |
| Sensitive data in MCP queries | Low | Only public law references sent; no personal data |
| Outdated law references | Medium | MCP server sources from 국가법령정보센터 (regularly updated) |
