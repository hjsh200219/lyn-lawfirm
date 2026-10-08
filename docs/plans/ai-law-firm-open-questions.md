# Open Questions

## integrate-new-skills - 2026-03-27
- [ ] Should the README section heading be renamed from "법률 문서 작성 스킬" to a broader title (e.g., "법률 스킬"), or should a separate subsection be added for advisory skills? — Affects how users perceive the plugin's capability scope (document drafting only vs. broader legal assistance).
- [ ] Should `.claude/settings.json` unzip permissions be removed entirely after extraction, or replaced with other permissions? — The current permissions (`unzip -l`) were only for inspecting the ZIPs and will be unnecessary once extracted.
- [ ] Should the plugin version in `plugin.json` be bumped from 1.0.0 to 1.1.0 to reflect the addition of 2 new skills? — Semantic versioning convention; minor version bump for additive feature changes.

## ai-law-firm - 2026-10-08 (v3·v3.1 — 9월 자료 정렬)

v3 에서 새로 생긴 질문. 번호는 `ai-law-firm.md` 본문이 참조한다.

- [ ] **Q1 lawfirm-suite 와의 정본 관계** — 이영남 사무소 하네스(구조도 v4 ④층)의 `lawfirm-suite 0.2.7` 은 lyn-lawfirm 6 스킬과 같은 이름의 스킬 6종 + 4종을 갖고 3계층 위임을 이미 운영한다. 그 원본은 이 workspace 에 없다(2026-10-08 확인 — 구조도 1장과 데모 `daily_check/run_draft.py` 26–33행의 스킬 ID 참조뿐: `lawfirm-suite:complaint-drafter`·`certified-letter`·`litigation-brief`·`civil-complaint`, 문체표준 `house-writing-standard.md`). lyn-lawfirm 이 정본이고 사무소가 가져다 쓰는지, 반대인지, 따로 가는지 이영남 변호사와 정해야 한다. 정하기 전에는 에이전트 이름·36명 분장·4개 추가 스킬 내용이 미확인이다. — 7-f 이중 유지 위험, Phase 2 Q5 의 전제.
- [ ] **Q2 외부 명칭과 에이전트 호칭** — v2 의 배포명 질문(아래)에 더해, 사무소 내부 호칭 「매니징 파트너·부서장·전문변호사」를 마켓플레이스 배포물(에이전트 description·화면 문구)에 그대로 쓸지. AI 를 「변호사」로 부르는 표현이 §109·§23 취지와 충돌할 수 있어 배포물에서는 「○○ 분석 담당」 같은 중립 호칭을 검토.
- [ ] **Q3 에이전트 레이어 배포 단위** — 같은 `lyn-lawfirm` 플러그인에 넣으면 일반 사용자에게도 1계층·부서장이 보이고, 별도 플러그인(같은 마켓플레이스)으로 내면 사무소용과 공개용이 분리된다. V7(Resolution A 대체)의 「남는 위험」이 이 결정에 달려 있다. **Phase 1 공개 배포의 선행 조건** — 정하기 전에는 에이전트 레이어를 `main` 에 병합하지 않는다(`main` push = 마켓플레이스 갱신). 권고 기본값: 같은 마켓플레이스의 별도 플러그인. 이유 ① 비변호사 사용자에게 「부서장·전문변호사」 에이전트가 보이지 않는다(§109 취지) ② 주요 서면 게이트·사무소 설정 훅이 일반 사용자의 고소장 작성에 걸리지 않는다 ③ 사무소용 진입점을 `claude --agent` 로 따로 둘 여지가 생긴다(계획 §9 A-1).
- [ ] **Q4 사무소 설정 위치** — 사건 폴더 루트·변호사 정보 등 사무소별 값을 어디에 둘지(플러그인 밖 설정 파일 · 환경 변수 · 프로젝트 `.claude/` 아래). 조건: 플러그인 업데이트로 덮이지 않을 것, 비밀값을 담지 않을 것(V10).
- [ ] **Q5 민사 문서 스킬: 이식 vs 신규** — 사무소는 민사소장(`civil-complaint`)·준비서면/의견서(`litigation-brief`)·불송치 이의신청을 이미 쓴다(구조도 ④층, 데모 `run_draft.py`). Q1 이 정해지면 이식할지 새로 만들지 결정. v2 의 「민사 문서 스킬 우선순위」 질문을 대체한다.
- [ ] **Q6 서브에이전트 중첩 한도 버전 의존** — 2026-10-08 문서·Claude Code 2.1.294 기준 기본 3단. 2.1.172–2.1.216 은 5단 고정, **2.1.217–2.1.218 은 기본 1단**(서브에이전트가 위임 못 함), 2.1.219 부터 3단. 3계층 위임의 최소 조건은 2.1.219 이상 또는 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` ≥ 2. 플러그인 설명·설치 안내에 최소 버전을 적을지, 1계층이 버전을 확인하고 3계층 없이 도는 대체 경로를 둘지 결정.
- [ ] **Q7 v3 결정 V1–V11 Round 3 리뷰** — 특히 V1(운영자 = 변호사)·V5(사건 DB 폐지)·V7(Resolution A 대체)은 v2 합의를 뒤집는다. Round 3 Critic 1차는 ITERATE 30/40(높음 2건) → v3.1 반영. Architect 리뷰와 Critic 재리뷰 후 Status 를 바꾼다.
- [ ] **Q8 사무소 서식·문체·금지 문구의 형식** — 9월 자료의 사무소는 서식 SSOT(바탕 12pt·1.5줄·양쪽정렬·레터헤드·꼬리말 4종)·문체표준(`house-writing-standard.md`)·금지항목 목록(데모 run_draft.py 127행 「§2 금지항목」 — 옛 사무소명류로 추정)을 쓴다. lyn-lawfirm 은 기본값(바탕 12pt·A4·여백 25/20/25/25)만 있다. 사무소가 이 셋을 어떤 파일 형식으로 주면 docx 훅 ②⑥과 4단계가 읽을지(Phase 2, Q4 와 같이 결정).

## ai-law-firm - 2026-10-08 (v3.2 — AI OS 목표)

목표 「AI 법무법인을 위한 AI OS = Claude Code + Obsidian + 에이전트 + Slack, 필요시 Buzz」(계획 §0.4·§12)에서 생긴 질문.

- [ ] **Q9 Slack 을 지나는 정보의 범위 — Slack 승인·인테이크 기능의 선행 조건** — 나가는 것(브리지 → Slack: 승인할 diff·구성안·초안 요지)과 들어오는 것(사람이 붙여 넣는 상담 원문·파일) 모두. Q9 전에는 사건 코드·건수·기한·결함 개수만(계획 §12.4 규칙 2). 승인 버튼은 승인할 내용이 보여야 의미가 있으므로 Q9 없이 켜지 않는다. [확인 필요] 쟁점(판단은 변호사): 변호사법 제26조(비밀유지의무), 개인정보 보호법 제28조의8(국외로 제공 — 「조회되는 경우를 포함」 — ·처리위탁·보관), 제23조(민감정보 — 형사 사건 정보가 시행령상 범위에 드는지 미확인). 같은 쟁점이 엔진 자체(사건 내용이 Anthropic API 로 감)에도 있다 — 9월 사무소도 같은 구조다.
- [ ] **Q10 Obsidian 볼트 동기화** — 없음(기본) / Obsidian Sync / iCloud·OneDrive 등. 휴대폰에서 사건메모를 직접 보고 싶다면 필요하지만 의뢰인 비밀이 클라우드로 나간다. 각 서비스의 암호화 방식은 이 계획에서 확인하지 않았다(미확인). Q9 와 같은 법 쟁점(제26조·개인정보 보호법 제28조의8·제23조)이 있다.
- [ ] **Q11 Slack 브리지의 배포 형태** — lyn-lawfirm repo 하위 폴더 / 별도 repo / Q3 의 사무소용 플러그인과 한 묶음. 플러그인은 상주 프로세스를 설치하지 못하므로 설치 절차가 따로 필요하다: 사무소 워크스페이스에 Slack 앱 직접 생성(Socket Mode 앱은 공개 Marketplace 배포 불가 — v3.2 Critic 인용, AC16 에서 확인) · 앱 수준 토큰·봇 토큰 2종 발급 · 키체인 등록 · OS 서비스 등록 · Anthropic API 키(Q14). 9월 사무소가 Windows 이므로 Windows 지원 여부도 같이 정한다.
- [ ] **Q12 9월 봇 예약 8종의 내용** — 구조도에는 「예약 8」 개수만 있고 구조도·요약에서 확인된 것은 매일 9시 브리핑·기한 스캔뿐이다. 데모에 `run_daily.py`·`run_weekly.py`·`run_harvest.py` 등이 있으나 파일 이름만 봤고 내용은 확인하지 않았다. 8종 목록을 사무소에 확인해 Slack 브리지 Phase 2 범위를 정한다.
- [ ] **Q13 Buzz 재평가 시점** — 채택 조건 3개(모바일 클라이언트 · 릴레이 운영 수용 · 에이전트 논의 실시간 관찰 수요, 계획 §12.5)를 누가 언제 점검할지. 이 Mac 에서는 2026-09-15 사용자가 Buzz 를 삭제했으므로 설치·복구하지 않는다.

- [ ] **Q14 브리지 엔진 계정 — Slack 브리지 착수의 선행 조건** — Agent SDK 개요: "Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK." 브리지는 여러 사무소에 설치하는 제품이므로 기본은 사무소별 API 키(사용량 과금)다. 구독 로그인을 쓰려면 Anthropic 사전 승인이 필요하다. API 비용을 사무소가 감당할지, 승인을 신청할지 결정. 9월 사무소 자체의 「Claude CLI(구독)」 사용이 이 정책과 어떤 관계인지는 미확인.

## ai-law-firm - 2026-05-12 (v2 업데이트)

v3(2026-10-08) 에서 상태를 갱신했다. 닫힌 항목은 근거를 붙였다.

- [ ] Phase 0 스파이크 결과 — plugin `agents/*/AGENT.md`가 Claude Code에서 subagent로 인식되는지 검증 필요. 미인식 시 Fallback(skills/managing-partner/SKILL.md 메타스킬)으로 전환. Phase 1 착수 전 반드시 확정. → **v3: 범위 축소.** 인식 여부는 공식 문서로 Yes(형식은 `agents/<이름>.md`). 남은 검증은 Phase 0 AC1–AC16(AC4 는 닫힘).
- [x] AGENT.md 포맷 확정 — Claude Code plugin agent 시스템의 공식 스펙이 확정되면 AGENT.md frontmatter 필드(name, description, triggers, delegates_to 등)를 해당 스펙에 맞춰 확정해야 한다. 현재는 SKILL.md 유사 구조를 가정. → **v3 닫힘**: https://code.claude.com/docs/en/sub-agents (2026-10-08 조회). 필수 `name`·`description`, `triggers`·`delegates_to` 는 스펙에 없어 삭제(계획 §4.3).
- [x] K-Data MCP 배포 방식 결정 — `.mcp.json`에 K-Data MCP를 추가하면 마켓플레이스 설치 사용자도 활용 가능하지만, K-Data 서버 가용성에 대한 의존성이 생긴다. claude.ai 커넥터 전용으로 남기면 Cowork/claude.ai 사용자만 활용. Phase 2 시작 전 결정 필요. → **v3 닫힘**: korean-law 서버 중단으로 a77f764(1.2.0)에서 `.mcp.json` 을 `k-data` 로 바꿨다. 가용성 의존은 계획 7-d 로 관리.
- [ ] 외부 명칭 최종 확정 — "AI 법무법인"은 내부 코드네임으로만 사용. 마켓플레이스 배포명은 "Lyn 법률 어시스턴트 팀" / "이영남 변호사 AI 보조시스템" 중 이영남 변호사와 협의하여 확정. 변호사법 §23 명칭 규제 준수 필수. → v3 Q2 로 범위 확대(에이전트 호칭 포함).
- [x] 에이전트 간 상태 공유 메커니즘 — 프롬프트 컨텍스트 전달로 충분한지, 별도 상태 파일(.omc/state/)이 필요한지. 긴 사건에서 컨텍스트 윈도우 압박 가능성 확인 필요. → **v3 닫힘**: 사건 폴더의 `사건메모.md` 가 정본(P9, 계획 §5.3). 하위 에이전트는 제안만, 쓰기는 메인 대화(v3.1).
- [ ] L5↔L5 직접 호출 금지의 강제 수준 — 현재 프롬프트 수준(soft enforcement). Claude Code가 에이전트 간 호출 제한을 플랫폼 수준에서 지원하면 hard enforcement로 전환 가능한지 확인. → **v3.1 결론**: 하위 위임 금지는 `disallowedTools: Agent` 로 하드. 부서장의 위임 대상을 자기 부서로 묶는 것은 **불가** — `Agent(agent_type)` 허용 목록은 `claude --agent` 메인 스레드에만 적용(sub-agents 문서, 2026-10-08). 수평 호출 금지는 soft 로 남는다.
- [ ] 민사 문서 스킬 우선순위 — Phase 2에서 소장 초안 / 답변서 초안 / 준비서면 초안 중 어떤 것을 먼저 개발할지 이영남 변호사의 실무 빈도 기준으로 결정. → v3 Q5(이식 vs 신규)로 대체.
- [x] 사건 DB 구조 — Phase 3에서 사건 히스토리를 저장할 방식. JSON 파일 vs SQLite vs 외부 서비스. 플러그인 아키텍처 제약(서버 없음) 하에서 실현 가능한 방식 조사 필요. → **v3 닫힘**: 사건 DB 없음. 사무소 업무관리시스템과 같은 `사건메모.md` 파일 정본(V5).
