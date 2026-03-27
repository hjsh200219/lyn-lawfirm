# Quality Score

## Scoring Dimensions

Each generated document is evaluated against these dimensions:

### 1. Legal Accuracy (Critical)
- [ ] All statute citations verified via MCP (`search_laws` -> `get_law_detail`)
- [ ] Case citations include full case number format
- [ ] Aggravated punishment provisions checked where applicable
- [ ] Statute of limitations verified; near-expiry warnings shown
- [ ] Constitutional requirements (구성요건) are logically complete

### 2. Document Formatting (High)
- [ ] A4, Batang 12pt, correct margins (25/20/25/25mm)
- [ ] Title: 22-24pt bold centered
- [ ] Section headers follow size hierarchy (16pt > 14pt > 13pt > 12pt)
- [ ] Line spacing: 160-180%
- [ ] Evidence numbered as `증 제N호증`

### 3. Information Completeness (High)
- [ ] All user-provided information incorporated
- [ ] Missing information marked as `[확인 필요]` (not fabricated)
- [ ] Party names consistent throughout document
- [ ] Dates in `0000. 00. 00.` format

### 4. CTA Compliance (Required)
- [ ] CTA present in chat response
- [ ] CTA present as separate last page in .docx
- [ ] CTA uses correct attorney info from `references/cta-config.md`
- [ ] CTA styling: Batang 12pt, blue (#0055CC)
- [ ] `{{문서유형}}` placeholder correctly replaced

### 5. Escalation Handling (High)
- [ ] Escalation criteria checked per skill-specific thresholds
- [ ] Warning banners displayed for flagged conditions
- [ ] No silent pass-through of edge cases

## Score Table

| Dimension | Weight | Pass Criteria |
|-----------|--------|---------------|
| Legal Accuracy | 30% | All citations verified; no fabrication |
| Document Formatting | 25% | Matches specification exactly |
| Information Completeness | 20% | All provided info used; gaps marked |
| CTA Compliance | 15% | Present in both outputs with correct content |
| Escalation Handling | 10% | All applicable thresholds checked |

## Anti-Patterns (Auto-Fail)

- Fabricating any information (names, dates, amounts, law articles)
- Omitting CTA from either chat or .docx output
- Citing unverified law articles without `[법령 확인 필요]` marker
- Saving final files to `/tmp/` only
- Duplicate Paragraph objects in docx-js output
