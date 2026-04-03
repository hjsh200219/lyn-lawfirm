# Frontend

## 공통 금지 사항

- **이모지를 UI 아이콘으로 사용 금지.** OS/브라우저마다 렌더링이 다르고, 텍스트와 간격이 맞지 않음. SVG 아이콘 또는 Remixicon 사용.
- **미구현 페이지로 링크 금지.** 페이지가 없으면 disabled 처리 + "준비 중" 태그 표시.
- **E2E 테스트는 로그인/비로그인 두 상태 모두 검증.**
- **디자인 리뷰 시 모든 상태의 스크린샷 확인 필수.**


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
