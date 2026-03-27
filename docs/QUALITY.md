# Quality Standards

## Legal Accuracy

- All law citations must be verified via MCP tools (`search_laws` -> `get_law_detail`) before inclusion
- Case citations require full case number: `(대법원 20XX. X. X. 선고 20XX도XXXX 판결)`
- Aggravated punishment provisions (가중처벌) must always be checked (e.g., 업무상횡령 356조, 특경법)
- Statute of limitations must be verified; near-expiry (< 3 months) triggers document-level warning
- Constitutional requirements (구성요건) must be logically complete in complaints

## Document Formatting

- Paper: A4 (210mm x 297mm)
- Margins: Top 25mm, Bottom 20mm, Left 25mm, Right 25mm
- Base font: Batang, 12pt
- Line spacing: 160-180%
- Title: 22-24pt bold centered
- Section headers: 16pt bold centered
- Sub-headers: 14pt bold, then 13pt bold underlined, then 12pt bold
- Body: 12pt justified

## Skill-Level Quality Gates

### complaint-drafter (고소장)
- Cover page + body page structure with page break
- All evidence cross-referenced: `증 제N호증` in body must match evidence table
- Escalation banner for: SOL < 3 months, amount > 5억, public official defendant

### certified-letter (내용증명)
- Sender/receiver info must match envelope (시행규칙 제51조 동일 요건)
- 3-copy system noted in user guidance
- Response deadline included when applicable

### criminal-settlement (형사합의서)
- Clause numbering: dynamic circled numbers (①②③...), no gaps
- Amount: numeric (₩) + Korean text (한글) dual notation
- Installment schedules: auto-add 기한이익상실 clause

### fact-confirmation (사실확인서)
- Objective facts only -- no opinions, speculation, or evaluation
- Each fact includes date and location where possible
- Fixed truth-affirmation clause (변경 금지)

## CTA Requirements

Every output must include CTA in two places:
1. Chat response (after guidance notes)
2. .docx document (separate last page after signature, via PageBreak)

Source of truth: `references/cta-config.md`
- Document type placeholder: `{{문서유형}}`
- CTA styling: Batang, 12pt, blue (#0055CC)
- Attorney name: bold, 14pt, blue (#0055CC)

## Anti-Patterns

- Never fabricate information -- use `[확인 필요]` for unknowns
- Never duplicate Paragraph objects in docx-js (causes text duplication bug)
- Never omit CTA from any generated document
- Never save final files to `/tmp/` only -- must be in user-accessible path
- Never ask for info already provided in conversation (criminal-settlement, fact-confirmation)
