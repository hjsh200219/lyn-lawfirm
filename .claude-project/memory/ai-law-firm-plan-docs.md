---
name: ai-law-firm-plan-docs
description: AI 법무법인 기획 문서 위치 및 MCP 엔드포인트 매핑
metadata:
  type: reference
created: 2026-05-12
---

AI 법무법인 멀티에이전트 아키텍처 작업의 권위 있는 참조 위치.

**플랜 문서 (정본)**:
- `docs/plans/ai-law-firm.md` — RALPLAN-DR 2라운드 합의안 (v2, ~1700 lines). Phase 0~3 로드맵, Agent Tree, Pre-mortem 4건, Test plan, ADR
- `docs/plans/ai-law-firm-open-questions.md` — 후속 의사결정 포인트

> 주의: `.omc/plans/ai-law-firm.md`에도 동일 내용이 존재하나 `.omc/`는 `.gitignore`에 등록되어 추적되지 않음. **수정·참조는 항상 `docs/plans/`** 기준.

**MCP 엔드포인트**:
| Server | URL | 등록 위치 |
|--------|-----|-----------|
| korean-law | `https://korean-law.up.railway.app/mcp` | `.mcp.json` (플러그인 설치자 자동 사용) |
| K-Data | `https://public-data.up.railway.app/mcp` | claude.ai 커넥터만 (플러그인 미등록) |

**Phase 2 미결 사항**: K-Data MCP를 플러그인 `.mcp.json`에 번들할지 vs 커넥터 전용 유지할지.

**How to apply:**
- 에이전트/스킬 관련 작업 시 위 plan 문서를 먼저 참조
- MCP 호출 시 두 서버의 가용성을 분리해서 다룰 것 (korean-law는 항상 가용, K-Data는 사용자 환경 의존)
- Plan 수정 후에는 git push 필수 (다른 PC 동기화)

관련: [[ai-law-firm-architecture]]
