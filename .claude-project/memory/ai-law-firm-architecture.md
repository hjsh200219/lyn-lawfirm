---
name: ai-law-firm-architecture
description: AI 법무법인 멀티에이전트 아키텍처는 Claude Code native subagents + MCP (Option A), hub-spoke 구조, specialist 간 직접 호출 금지
metadata:
  type: project
created: 2026-05-12
---

AI 법무법인 멀티에이전트 시스템의 핵심 아키텍처 결정. 산출물은 `docs/plans/ai-law-firm.md` (RALPLAN-DR 2라운드 합의안, Architect APPROVE_WITH_MINOR / Critic 35/40).

**결정**:
- 플랫폼: Claude Code native subagents (`.claude/agents/*.md` 또는 plugin 배포 `agents/*/AGENT.md`) + MCP. 외부 프레임워크(LangGraph/CrewAI/FastAPI 자체빌드) 배제.
- 토폴로지: Hub-Spoke. Managing Partner (대표변호사 AI) → 전문 specialist (형사/민사/가사/행정/지재/노동/회사·M&A) → MCP + 기존 6스킬. Specialist 간 cross-call 금지.
- 레이어: 기존 L0-L4에 **L5 Agents** 추가. L5 → L2(skills), L1(references), L0(MCP) 호출 가능. L5 ↔ L5 직접 호출 금지.
- P2/P3 모순 해소: **Resolution A** 채택 → 경량 모드 폐기. 에이전트 시스템이 활성화된 세션에서 모든 skill 호출은 Managing Partner를 경유. 단축 경로 없음. SKILL.md MANDATORY TRIGGERS는 MP 활성 시 비활성화 필요 (메커니즘은 Phase 0에서 검증).
- Phase 0 spike (1-2일): Phase 1 MVP 진입 전 **플러그인으로 배포된 AGENT.md가 실제 subagent로 동작하는지** 반드시 검증. 실패 시 fallback = Managing Partner를 `skills/managing-partner/SKILL.md` 메타스킬로 구현.

**Why:** 외부 프레임워크는 플러그인 배포 친화적이지 않고 1인 변호사 운영체제에 과도한 복잡도. Native + MCP가 기존 인프라(`korean-law` MCP)와 자연스럽게 결합. Hub-Spoke는 변호사 검토 게이트와 책임 추적성 보장에 최적. Resolution A는 P2(변호사법 §109 준수)를 P3(스킬 재사용)보다 상위 원칙으로 명시.

**How to apply:**
- 새로운 에이전트 관련 작업 시: (1) Option A 전제 유지, (2) Specialist 간 직접 호출 패턴 거부, (3) Phase 0 결과 확인 전에는 플러그인 배포 가정 금지, (4) skill 단축 경로 (Managing Partner 우회) 제안 거부.
- Phase 0 미완 상태에서 `agents/` 디렉토리 생성·구현 시작 거부.
- Critic minor 잔존 사항 (Intake "도구: 없음"→"Read", §5.4 hedging, Phase 0 AC 보강 1건)은 Phase 1 착수 전 정정.

관련: [[ai-law-firm-naming-constraint]], [[ai-law-firm-plan-docs]], [[lyn-operator-context]]
