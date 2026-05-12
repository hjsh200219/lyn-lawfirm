# Open Questions

## integrate-new-skills - 2026-03-27
- [ ] Should the README section heading be renamed from "법률 문서 작성 스킬" to a broader title (e.g., "법률 스킬"), or should a separate subsection be added for advisory skills? — Affects how users perceive the plugin's capability scope (document drafting only vs. broader legal assistance).
- [ ] Should `.claude/settings.json` unzip permissions be removed entirely after extraction, or replaced with other permissions? — The current permissions (`unzip -l`) were only for inspecting the ZIPs and will be unnecessary once extracted.
- [ ] Should the plugin version in `plugin.json` be bumped from 1.0.0 to 1.1.0 to reflect the addition of 2 new skills? — Semantic versioning convention; minor version bump for additive feature changes.

## ai-law-firm - 2026-05-12 (v2 업데이트)
- [ ] Phase 0 스파이크 결과 — plugin `agents/*/AGENT.md`가 Claude Code에서 subagent로 인식되는지 검증 필요. 미인식 시 Fallback(skills/managing-partner/SKILL.md 메타스킬)으로 전환. Phase 1 착수 전 반드시 확정.
- [ ] AGENT.md 포맷 확정 — Claude Code plugin agent 시스템의 공식 스펙이 확정되면 AGENT.md frontmatter 필드(name, description, triggers, delegates_to 등)를 해당 스펙에 맞춰 확정해야 한다. 현재는 SKILL.md 유사 구조를 가정.
- [ ] K-Data MCP 배포 방식 결정 — `.mcp.json`에 K-Data MCP를 추가하면 마켓플레이스 설치 사용자도 활용 가능하지만, K-Data 서버 가용성에 대한 의존성이 생긴다. claude.ai 커넥터 전용으로 남기면 Cowork/claude.ai 사용자만 활용. Phase 2 시작 전 결정 필요.
- [ ] 외부 명칭 최종 확정 — "AI 법무법인"은 내부 코드네임으로만 사용. 마켓플레이스 배포명은 "Lyn 법률 어시스턴트 팀" / "이영남 변호사 AI 보조시스템" 중 이영남 변호사와 협의하여 확정. 변호사법 §23 명칭 규제 준수 필수.
- [ ] 에이전트 간 상태 공유 메커니즘 — 프롬프트 컨텍스트 전달로 충분한지, 별도 상태 파일(.omc/state/)이 필요한지. 긴 사건에서 컨텍스트 윈도우 압박 가능성 확인 필요.
- [ ] L5↔L5 직접 호출 금지의 강제 수준 — 현재 프롬프트 수준(soft enforcement). Claude Code가 에이전트 간 호출 제한을 플랫폼 수준에서 지원하면 hard enforcement로 전환 가능한지 확인.
- [ ] 민사 문서 스킬 우선순위 — Phase 2에서 소장 초안 / 답변서 초안 / 준비서면 초안 중 어떤 것을 먼저 개발할지 이영남 변호사의 실무 빈도 기준으로 결정.
- [ ] 사건 DB 구조 — Phase 3에서 사건 히스토리를 저장할 방식. JSON 파일 vs SQLite vs 외부 서비스. 플러그인 아키텍처 제약(서버 없음) 하에서 실현 가능한 방식 조사 필요.
