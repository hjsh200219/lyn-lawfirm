# Data Schema

> This project has no database. It is a Claude Code plugin that generates `.docx` legal documents from structured prompts and user input.

## Runtime Data Structures

### generate.js Input Schema (criminal-settlement)

The `criminal-settlement` skill's `generate.js` script accepts a JSON object via `--data` flag. Below is the effective schema derived from the code:

```json
{
  "사건유형": "string (상해·폭행 | 재물손괴 | 명예훼손 | 사기·횡령 | 교통사고 | 성범죄 | ...)",
  "피해자": {
    "name": "string",
    "dob": "string (YYYY. MM. DD.)",
    "address": "string"
  },
  "가해자": {
    "name": "string",
    "dob": "string (YYYY. MM. DD.)",
    "address": "string"
  },
  "합의금": "string (numeric, e.g. '5000000')",
  "합의금한글": "string (e.g. '오백만')",
  "지급방법": "string (default: '계좌이체')",
  "계좌정보": "string (optional)",
  "지급기한": "string (YYYY. MM. DD.)",
  "고소사건번호": "string (optional)",
  "작성일": "string (YYYY년 MM월 DD일)",
  "분할지급": [
    { "금액": "string", "기한": "string" }
  ],
  "특약": {
    "고소취하": "boolean (default: true)",
    "처벌불원": "boolean (default: true)",
    "접근금지": "boolean",
    "게시물삭제": "boolean",
    "공개사과": "boolean",
    "추가게시금지": "boolean",
    "위약금": "boolean",
    "위약금액": "string",
    "기한이익상실": "boolean (auto-added for 분할지급)",
    "합의금미지급효력상실": "boolean",
    "기타": "string"
  }
}
```

All fields fall back to placeholder values (`○○○`, `○○○○. ○○. ○○.`) when not provided.

## MCP Server Data (External)

The MCP server at `korean-law.up.railway.app` provides law and case data from the Korean National Law Information Center. This repo does not own or manage that data.
