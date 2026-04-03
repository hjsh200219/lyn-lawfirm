# Plans

## Current State (v1.0.0)

The plugin ships with 6 skills (4 document drafting + 2 criminal analysis) and a remote MCP server for Korean law search. All skills are production-ready and available via the Claude Code marketplace.

## Planned Improvements

### Short-Term

- **Skill template standardization**: Unify SKILL.md frontmatter format and section structure across all 6 skills (noted in design-docs/index.md)
- **MCP tool usage patterns**: Document common search strategies for law/case lookup
- **Fix TD-001**: Read CTA info from `references/cta-config.md` in `generate.js` instead of hardcoding
- **Fix TD-002**: Align `generate.js` font to Batang per quality standards

### Medium-Term

- **Additional document types**: Expand skill coverage to civil complaints (민사소송장), appeals (항소장), response briefs (답변서)
- **Input validation**: Add JSON schema validation for `generate.js` input data
- **Programmatic generation for all skills**: Consider adding generate.js scripts for complaint-drafter, certified-letter, and fact-confirmation to improve .docx output consistency

### Long-Term

- **Multi-attorney support**: Allow CTA configuration for multiple attorneys
- **Template versioning**: Track document template versions for audit trails
- **Automated testing**: Validate generated .docx files against formatting standards

## Active Execution Plans

See [exec-plans/active/](exec-plans/active/) for in-progress work.

## Completed Plans

See [exec-plans/completed/](exec-plans/completed/) for finished work.
