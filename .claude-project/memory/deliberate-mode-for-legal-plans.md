---
name: deliberate-mode-for-legal-plans
description: 법률 도메인 고이슈 기획에는 RALPLAN --interactive --deliberate 모드를 기본값으로 사용
metadata:
  type: feedback
created: 2026-05-12
---

법률 도메인 (변호사법 영향, 컴플라이언스 게이트 설계, 사용자 노출 명칭, 변호사 검토 게이트 우회 가능성이 있는 기능)의 기획은 RALPLAN을 `--interactive --deliberate` 모드로 실행한다.

**Why:**
- 단일 변호사 운영체제에서 변호사법 §109/§23 같은 형사·행정 리스크 라인이 걸린 결정은 자동 합의보다 사람-개입 단계가 필요. 잘못된 자동 결정의 책임은 변호사에게 귀속.
- Deliberate 모드는 (1) 각 합의 에이전트(Planner/Architect/Critic)가 충분히 사유하고, (2) 사용자에게 분기점(draft review, final approval)에서 확인을 요청. Pre-mortem과 expanded test plan을 의무화.
- 이번 세션에서 사용자가 `--interactive --deliberate` 명시 선택 → 결과물 품질에 만족 (Architect APPROVE_WITH_MINOR, Critic 35/40, 2 round로 수렴).

**How to apply:**
다음 트리거에서는 fast 모드 대신 `--interactive --deliberate` 제안 (또는 자동 적용):
- 변호사법·형법 관련 아키텍처 결정
- 사용자 노출 명칭/마케팅 카피
- 변호사 검토 게이트 우회 가능성이 있는 기능 추가
- skill MANDATORY TRIGGERS 변경
- CTA 정책 변경
- MCP 서버 추가/제거 (법령·판례 데이터 흐름 변경)

빠른 결정으로 충분한 경우 (오타 수정, 문서 정리, 단일 파일 리팩터)에는 fast 모드 유지.

관련: [[lyn-operator-context]], [[ai-law-firm-architecture]]
