# Tech Debt Tracker

## Active Debt

| ID | Description | Severity | Location | Notes |
|----|-------------|----------|----------|-------|
| TD-001 | CTA info hardcoded in generate.js instead of reading from references/cta-config.md | Medium | `skills/criminal-settlement/scripts/generate.js` L13-21 | CTA object is duplicated; should read from shared config |
| TD-002 | generate.js uses Malgun Gothic (맑은 고딕) instead of Batang (바탕체) | Low | `skills/criminal-settlement/scripts/generate.js` L45 | Quality standard specifies Batang 12pt for all documents |
| TD-003 | No input validation schema for generate.js JSON data | Low | `skills/criminal-settlement/scripts/generate.js` | Relies on fallback `○○○` values; no schema validation |
| TD-004 | SKILL.md structure differs across skills | Low | `skills/*/SKILL.md` | Frontmatter format, section ordering, and depth vary between skills |

## Resolved Debt

| ID | Description | Resolved Date | Resolution |
|----|-------------|---------------|------------|
| (none yet) | | | |
