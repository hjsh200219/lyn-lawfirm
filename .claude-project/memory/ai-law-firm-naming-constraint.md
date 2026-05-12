---
name: ai-law-firm-naming-constraint
description: "AI 법무법인"은 내부 코드네임 전용. 외부 노출 명칭에 "법무법인" 단어 사용 금지 (변호사법 §23)
metadata:
  type: project
created: 2026-05-12
---

"AI 법무법인"이라는 프로젝트 명칭은 **내부 코드네임**으로만 사용한다. 사용자에게 노출되는 모든 텍스트(플러그인 description, README, 마케팅 카피, 출력 .docx CTA, 채팅 응답)에서는 "법무법인" 단어를 사용할 수 없다.

**Why:**
- 변호사법 §23은 법무법인으로 등록되지 않은 조직이 "법무법인" 명칭을 사용하는 것을 금지한다. 위반 시 §109 (무자격 법률사무 — 7년 이하 징역 / 5천만원 이하 벌금) 리스크와는 별도로 명칭 사용 자체가 제재 대상.
- 단일 변호사 운영체제(이영남 변호사 1인)는 법무법인이 아니라 개인 법률사무소. AI 보조시스템이 "법무법인" 명칭을 차용하면 의뢰인에게 조직 규모·구조에 대한 오인을 유발.
- 외부 명칭 후보: "Lyn 법률 어시스턴트 팀" (TBD). Phase 1 출시 전 최종 결정 필요.

**How to apply:**
- 내부 기획 문서 (`docs/plans/`, `.omc/plans/`, 합의 회의록): "AI 법무법인" 사용 가능
- 외부 노출 위치 (`.claude-plugin/plugin.json` description, `.claude-plugin/marketplace.json`, `README.md`, 모든 SKILL.md의 사용자 응답, 모든 .docx 생성물): "법무법인" 단어 검색 후 제거
- 새 텍스트 작성 시 "AI 법무법인" 자동 거부, "법률 어시스턴트" 등 대체
- 외부 명칭은 확정 시 `references/cta-config.md` 또는 별도 단일 source of truth에 등록 후 참조

관련: [[ai-law-firm-architecture]], [[lyn-operator-context]]
