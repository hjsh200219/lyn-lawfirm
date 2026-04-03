# Frontend

> This project has no frontend. It is a Claude Code plugin that operates within the Claude Code CLI environment.

## User Interface

Users interact with lyn-lawfirm through Claude Code's conversational interface:

1. **Input**: Natural language requests in Korean or English (e.g., "고소장 작성해줘")
2. **Triggers**: SKILL.md frontmatter defines keyword triggers for each skill
3. **Output**: Chat response with guidance notes + `.docx` file saved to user's filesystem

## Document Output Formatting

Generated `.docx` documents serve as the visual output. Formatting standards:

- Paper: A4 (210mm x 297mm)
- Margins: Top 25mm, Bottom 20mm, Left 25mm, Right 25mm
- Font: Batang (바탕체), 12pt base
- Line spacing: 160-180%
- Title: 22-24pt bold centered
- CTA page: separate last page, blue (#0055CC) text

See [QUALITY.md](QUALITY.md) for full formatting specification.
