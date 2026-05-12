---
name: plan-artifact-location-and-pull-first
description: 지속성이 필요한 plan 파일은 docs/plans/ (gitignored .omc/ 금지). Pack 실행 전 git pull --ff-only 먼저
metadata:
  type: feedback
created: 2026-05-12
---

이번 세션에서 사용자가 명시적으로 선택한 워크플로우 두 가지.

1. **Plan artifact location**: RALPLAN/CEO 리뷰/계획 산출물은 `docs/plans/` (리포지터리 추적 대상)에 저장. `.omc/plans/`는 `.gitignore`되어 세션 간 유실 위험.
2. **Pull-first Pack**: `/sh:git-push` Pack 실행 전 origin이 ahead이면 `git pull --ff-only` 먼저 수행. Pack을 ahead 상태에서 단독 실행하지 않음.

**Why:**
- Plan은 다음 세션과 다른 PC가 참조할 durable artifact → 리포지터리 일급 시민. `.omc/`는 transient state (스킬 내부 상태) 전용.
- Pack 전 동기화는 머지 충돌 방지, 다른 PC에서 작업한 변경과의 일관성 보장, CI 트리거 일관성 보장.
- 사용자가 명시적으로 "git pull 후 Pack 실행 (Recommended)"과 "docs/plans/로 이동 후 커밋"을 선택 — 검증된 판단.

**How to apply:**
- 새로 생성하는 RALPLAN/DR/CEO 리뷰/long-form planning 산출물은 처음부터 `docs/plans/`에 저장
- `.omc/plans/`에서 발견된 기존 산출물이 durable 가치를 가지면 `docs/plans/`로 이동 제안
- `/sh:git-push` 시작 시 `git status`로 origin ahead 여부 확인. ahead면 `git pull --ff-only` 후 Pack 진행
- 리베이스 충돌·non-fast-forward 발생 시 자동 머지 금지, 사용자 확인 요청

관련: [[ai-law-firm-plan-docs]]
