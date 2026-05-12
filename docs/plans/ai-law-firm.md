# AI 법무법인 (AI Law Firm) — 멀티에이전트 오케스트레이션 설계

**Plan ID**: `ai-law-firm`
**Status**: DRAFT v2 (Round 2 — Architect/Critic 리뷰 대기)
**Date**: 2026-05-12
**Mode**: RALPLAN-DR (Deliberate)

---

## Revision History

| 버전 | 날짜 | 변경 요약 |
|------|------|-----------|
| v1 | 2026-05-12 | 초안 작성. 5개 Principles, 3 Drivers, 5 Options, Agent Tree, Workflow, 3-Phase Roadmap, Pre-mortem 3건, Test Plan, ADR 포함 |
| v2 | 2026-05-12 | Architect/Critic Round 1 피드백 반영 (M1-M6 필수 수정, R1-R5 권장 수정). Phase 0 스파이크 추가 (M1). P2/P3 모순 해결 — Resolution A 채택 (M2). 변호사 검수 게이트 UX 명시 (M3). 테스트 하네스 실행 방법 + copy-paste 프롬프트 3건 추가 (M4). MCP 다운타임 Pre-mortem 추가 (M5). sentencing-predictor WebFetch 의존성 명시 (M6). B/C/D 통합 (R1). Observability 실현 가능성 태그 (R2). 문서 업데이트 범위 명시 (R3). PII 보호 규칙 (R4). 스킬 호출 메커니즘 통일 (R5). §10 Response Matrix 추가 |

---

## 1. Principles (설계 원칙)

### P1. 법률 정확성 > 자동화 속도

모든 법령·판례 인용은 MCP 도구로 검증한 후 포함한다. 검증 불가 시 `[확인 필요]`로 표시한다. 에이전트가 빠르게 답을 내는 것보다 틀리지 않는 것이 우선이다. 이 원칙은 기존 core-beliefs.md(Core Belief #1, #2)에서 계승하며, Managing Partner의 QA 게이트와 모든 specialist의 MCP 호출 의무를 정당화한다.

> 참조: `docs/design-docs/core-beliefs.md:1-9`

### P2. 변호사 검수 게이트 필수 (변호사법 준수)

AI가 생성한 모든 최종 산출물은 의뢰인에게 전달되기 **전에** 이영남 변호사의 검토를 거쳐야 한다. 이것은 편의가 아니라 법적 요건이다:

- **변호사법 제109조**: 변호사 아닌 자의 법률사무 취급 금지
- **변호사법 제23조**: "법무법인" 명칭은 등록 법인만 사용 가능

따라서 "AI 법무법인"은 **내부 코드네임/프로젝트명**으로만 사용하고, 마켓플레이스 배포 시에는 "Lyn 법률 어시스턴트 팀" 또는 "이영남 변호사 AI 보조시스템"처럼 법무법인 명칭을 피한다. AI는 초안 작성 보조 도구이며, 법률자문 주체가 아님을 모든 산출물에서 명확히 한다.

**아키텍처 영향**: 워크플로우의 최종 단계에 반드시 "변호사 검수" 게이트가 존재하며, 이 게이트를 건너뛸 수 없다. 에이전트 시스템이 활성화된 상태에서는 모든 스킬 호출이 Managing Partner를 경유하며, **기존 SKILL.md의 MANDATORY TRIGGERS로 인한 직접 스킬 실행은 Managing Partner 활성 시 비활성화된다** (§5.4 참조). 기존 SKILL.md의 면책 문구("변호사 검토 필수")는 텍스트 수준 면책이고, Managing Partner의 변호사 검수 게이트는 워크플로우 수준의 능동적 게이트이다. 두 메커니즘은 보완적으로 공존한다.

> **v2 변경 (M2)**: Resolution A 채택. 경량 모드(Managing Partner 우회) 제거. P2 엄격성 유지. §5.4에 기존 SKILL.md 트리거 비활성화 메커니즘 추가. §7-c에서 경량 모드 삭제.

### P3. 기존 6개 스킬 재사용 (핵심 로직 무변경)

기존 스킬(complaint-drafter, certified-letter, criminal-settlement, fact-confirmation, criminal-procedure-simulator, sentencing-predictor)의 **핵심 로직은 변경 없이** 에이전트 시스템에서 호출한다. 에이전트는 스킬의 **소비자**이지 스킬의 일부가 아니다.

**최소 변경 허용 범위 (v2 추가)**: 각 SKILL.md에 다음 두 가지 최소 변경만 허용한다. 이는 P2를 보장하기 위한 최소 뮤테이션이다:
1. frontmatter에 `gate: attorney-review` 필드 1줄 추가
2. "5단계: 생성 후 검증" 섹션에 1문장 추가: "Managing Partner가 활성 상태인 경우, 이 스킬의 출력물은 Managing Partner의 QA 게이트로 전달되어 변호사 검수를 거친 후 의뢰인에게 제공된다."

이 외의 스킬 내부 로직(워크플로우, 템플릿, 검증 로직)은 일체 수정하지 않는다.

이 원칙의 아키텍처적 근거: 기존 layer-rules.md의 L2→L2 참조 금지 규칙(`docs/design-docs/layer-rules.md:29`)을 유지하기 위함이다. 에이전트는 새로운 **L5 레이어**로 정의하며, `L5 → L2, L1, L0(MCP)` 의존만 허용한다.

> **v2 변경 (M2)**: "변경 없이" → "핵심 로직 무변경". 최소 변경 허용 범위(frontmatter 1줄 + 검증 섹션 1문장)를 명시적으로 정의.

### P4. L0-L4 레이어 규칙 유지 + L5 Agent 레이어 신설

기존 5-레이어 아키텍처를 보존하면서, 에이전트 오케스트레이션을 위한 L5를 추가한다:

```
L0  Plugin Manifest    .claude-plugin/, .mcp.json       -> (nothing)
L1  Shared References  references/                       -> (nothing)
L2  Skills             skills/*/SKILL.md                 -> L1, L0(MCP)
L3  Scripts            skills/*/scripts/                 -> L1, npm:docx
L4  Documentation      docs/, README.md, AGENTS.md       -> any
L5  Agents             agents/*/AGENT.md                 -> L2, L1, L0(MCP)
```

**L5 규칙**:
- L5 → L2: 에이전트가 스킬을 **스킬명 직접 지정** 방식으로 호출한다. 키워드 매칭이 아닌 명시적 스킬명을 사용한다. 이는 MANDATORY TRIGGERS가 없는 criminal-settlement, fact-confirmation도 일관되게 호출하기 위함이다 (R5).
- L5 → L1: 에이전트가 CTA 등 공유 참조를 읽을 수 있다
- L5 → L0(MCP): 에이전트가 MCP 도구를 직접 호출할 수 있다
- L5 ↔ L5: 에이전트 간 직접 호출 **금지**. Managing Partner만 specialist를 위임/호출할 수 있다 (hub-spoke 토폴로지)
- L2 → L5: **금지**. 스킬이 에이전트를 참조하면 안 된다 (역방향 의존성)

> **v2 변경 (R5)**: L5 → L2 호출 방식을 "트리거 키워드 또는 직접 지정" → "스킬명 직접 지정"으로 통일.

### P5. Never Fabricate (계승)

기존 Core Belief #2를 그대로 계승한다. 에이전트 체인에서 한 단계라도 팩트를 생성하면 체인 전체의 신뢰가 무너진다. 모든 에이전트는 `[확인 필요]` 규칙을 준수한다. 특히 Managing Partner가 specialist 결과를 통합할 때 정보를 보간(interpolate)하지 않는다.

### P6. PII 보호 (MCP 쿼리 마스킹)

> **v2 추가 (R4)**

에이전트가 MCP를 호출할 때, 쿼리에 당사자 실명·주민등록번호·연락처를 포함하지 않는다. MCP 쿼리는 법령명·조문번호·판례 키워드 등 공개 법률 정보 검색에만 사용한다. 사건 메타데이터 JSON은 로컬 컨텍스트에서만 유지하며 외부로 전송하지 않는다.

이 원칙은 기존 `docs/SECURITY.md:18`의 "Data NOT sent: User personal information, party details, document content" 약속을 AGENT 레이어에서도 준수하는 것이다.

AGENT.md 프롬프트에 다음 규칙을 명시한다:
- MCP 호출 시 당사자 실명 → 일반명사("피해자", "피고소인")로 대체
- 주민등록번호, 연락처 등 식별정보는 MCP 쿼리에 절대 포함 금지
- 법령·판례 검색에 필요한 최소 키워드(죄명, 법조항명, 피해액 구간)만 전송

---

## 2. Decision Drivers (의사결정 동인) — Top 3

### D1. 변호사법 위반 리스크 (법적 생존)

한국에서 변호사 아닌 자가 법률사무를 취급하면 형사처벌 대상이다(변호사법 §109, 7년 이하 징역 또는 5천만원 이하 벌금). AI 시스템이 "법률자문을 제공한다"는 인상을 주면 안 된다. 이 드라이버는 모든 아키텍처 결정의 최상위 제약이다.

**영향**: 변호사 검수 게이트 필수(P2), 외부 명칭 제한(P2), CTA 비생략(기존 규칙), 면책조항 필수.

### D2. 단일 변호사 운영 현실 (운영 부담 최소화)

이영남 변호사 1인이 운영한다. 별도 서버, 별도 인프라, 별도 배포 파이프라인을 요구하는 플랫폼은 채택 불가. Claude Code 플러그인 구조(`/plugin install lyn-lawfirm`)로 배포할 수 있어야 하고, `npm install`과 `.mcp.json` 설정만으로 동작해야 한다.

**영향**: 자체 빌드(FastAPI/LangGraph) 탈락, 외부 플랫폼(검증 불가능한 것들) 탈락, Claude Code native 구조 채택.

### D3. 한국어 법률 도메인 정밀도

양형기준표, 공소시효, 증거번호 체계("증 제N호증"), 한국 법원 양식(바탕체 12pt, A4) 등 한국 법률 실무에 특화된 규칙이 이미 6개 스킬에 깊이 녹아 있다. 플랫폼 선택은 이 스킬들을 **변경 없이 호출**할 수 있어야 한다.

**영향**: SKILL.md 프롬프트 기반 아키텍처 유지(P3), L5 레이어가 L2를 호출하는 구조(P4).

---

## 3. Viable Options — 플랫폼 선택

### Option A: Claude Code Native Subagents + MCP ★ 채택

현재 플러그인 구조(`skills/*/SKILL.md`) 위에 `agents/*/AGENT.md`를 추가한다. **Phase 0 스파이크에서 검증된 메커니즘**(plugin `agents/*/AGENT.md` vs `.claude/agents/*.md`)을 사용하여 에이전트를 정의한다. Managing Partner가 Claude Code의 subagent 위임 기능을 사용해 specialist를 호출한다.

**Fallback (Phase 0에서 plugin agents가 동작하지 않을 경우)**: Managing Partner를 `skills/managing-partner/SKILL.md` (메타스킬)로 구현한다. 이 경우 L5가 L2로 축소되며, layer-rules.md 업데이트는 L5 신설이 아닌 "메타스킬" 규칙 추가로 제한된다. 상세는 §6 Phase 0 참조.

> **v2 변경 (M1)**: "또는" 제거. Phase 0 스파이크에서 메커니즘을 확정한 후 단일 경로를 선택. Fallback 명시.

| Driver | Score | 근거 |
|--------|-------|------|
| D1 변호사법 | **High** | 기존 CTA/면책 체계 그대로 유지, 변호사 검수 게이트 삽입 용이 |
| D2 운영 부담 | **High** | `git push main` = 배포 완료. 별도 서버 불필요. 1인 운영 가능 |
| D3 도메인 정밀도 | **High** | 기존 6개 SKILL.md를 변경 없이 트리거/호출 |

**Pros**:
1. 기존 배포 경로(`/plugin install`) 유지 — 사용자 설치 경험 동일
2. SKILL.md 프롬프트 기반 아키텍처와 자연스러운 호환
3. MCP 도구(korean-law, K-Data)를 에이전트가 직접 호출 가능
4. L5 레이어 추가만으로 기존 L0-L4 구조 보존

**Cons**:
1. Claude Code subagent API/플러그인 에이전트 시스템의 정확한 스펙은 Claude Code 버전에 따라 변동 가능 → **Phase 0에서 검증**
2. 에이전트 간 상태 공유(사건 ID, 컨텍스트)를 위한 명시적 메커니즘이 프롬프트 수준에서 관리됨
3. 병렬 에이전트 실행의 안정성은 Claude Code 런타임에 의존

---

### Option B-D: 외부 에이전트 프레임워크 (OpenClaw / Hermes / Claude OS) — 검증 불가로 일괄 탈락

> **v2 변경 (R1)**: B/C/D를 동일 사유로 통합.

**탈락 사유 (공통)**: 2026년 5월 기준 검증 가능한 1차 소스(공식 문서, API 스펙, 배포 가이드)를 확인할 수 없다. 검증 불가능한 플랫폼에 법률 도메인 시스템을 구축하는 것은 D1(법적 리스크)과 D2(운영 부담) 양쪽에서 허용 불가.

| 후보 | 구체적 미확인 사항 |
|------|-------------------|
| OpenClaw (오픈클로) | 플러그인 배포 경로 호환성, SKILL.md 호출 방식, MCP 연동 방식 미확인 |
| Hermes (에르메스) | 동일. 한국어 법률 도메인 동작 미확인 |
| Claude OS | 구체적 제품 존재 여부 자체가 미확인 |

**향후 검토 트리거 (공통)**: 해당 플랫폼의 공식 문서가 공개되고, Claude Code 플러그인과의 호환성이 검증된 시점에 재평가.

---

### Option E: 자체 빌드 (FastAPI + LangGraph/CrewAI) — 탈락

**탈락 사유 (D2 위반)**:

기술적으로는 실현 가능하나, 운영 현실과 충돌한다:

1. **별도 서버 필요**: FastAPI 서버를 호스팅·유지보수해야 함. Railway/Vercel/Fly.io 비용 + 모니터링 부담
2. **이중 배포 경로**: 플러그인(`/plugin install`) + 서버 배포를 모두 관리해야 함
3. **SKILL.md 재구현**: LangGraph 에이전트에서 SKILL.md 프롬프트를 호출하려면 래퍼 코드 필요 — P3 위반
4. **MCP 이중 연결**: 서버 측에서 MCP를 호출하려면 별도 클라이언트 구현 필요
5. **1인 변호사가 Python/FastAPI 서버를 운영**: D2와 정면 충돌

| Driver | Score | 근거 |
|--------|-------|------|
| D1 변호사법 | Med | 자체 서버에서 법률 처리 = 추가 리스크 포인트 |
| D2 운영 부담 | **Low** | 서버 운영·모니터링·비용 부담 과다 |
| D3 도메인 정밀도 | Med | SKILL.md 재구현/래핑 필요 |

---

## 4. Agent Tree (에이전트 아키텍처)

### 4.1 전체 구조

```
대표변호사 AI (Managing Partner)
│
├── 사건접수·분류 (Intake & Triage)
│   책임: 의뢰인 자유 텍스트 → 구조화된 사건 메타데이터
│   입력: 자연어 사건 설명, 첨부 파일
│   출력: 사건 메타데이터 JSON (분야, 당사자, 쟁점, 긴급도)
│   도구: 없음 (순수 텍스트 분석)
│   위임: Managing Partner에게 분류 결과 반환
│
├── 형사팀 (Criminal) ← Phase 1 MVP, 문서 생성 가능
│   ├── 수사·기소 대응
│   │   책임: 피의자/피고인 입장의 형사절차 안내 및 대응 전략
│   │   입력: 사건 메타데이터 + 혐의 정보
│   │   출력: 절차 안내, 대응 전략 보고서
│   │   스킬: criminal-procedure-simulator (스킬명 직접 지정)
│   │   MCP: search_laws, search_cases, get_case_detail
│   │
│   ├── 고소·고발 담당
│   │   책임: 피해자 입장의 고소장 작성 지원
│   │   입력: 상담 녹취록, 증거자료, 당사자 정보
│   │   출력: 고소장 .docx
│   │   스킬: complaint-drafter (스킬명 직접 지정)
│   │   MCP: search_laws, get_law_detail, search_cases, get_case_detail
│   │
│   └── 양형·합의 담당
│       책임: 양형 예측 + 합의서 작성
│       입력: 사건 정보, 피해액, 전과, 합의 조건
│       출력: 양형 예측 보고서, 합의서 .docx
│       스킬: sentencing-predictor (스킬명 직접 지정), criminal-settlement (스킬명 직접 지정)
│       MCP: search_cases, get_case_detail, search_laws
│       참고 (v2 M6): sentencing-predictor는 양형기준표를 WebFetch로
│         대법원 양형위원회 사이트(sc.scourt.go.kr)에서 직접 조회한다.
│         이 데이터는 MCP 검증 경로를 거치지 않으므로, §7-b의 MCP-only
│         인용 규칙은 양형기준표 데이터에 적용되지 않는다.
│         WebFetch 실패 시 sentencing-predictor는 기본 형량 범위만
│         제시하고 사용자에게 양형위원회 사이트 수동 확인을 안내한다.
│
├── 민사팀 (Civil) ← Phase 2, 리서치 전용 (문서 = 내용증명만)
│   책임: 계약 분쟁, 손해배상, 부동산, 대여금 등
│   입력: 사건 메타데이터 + 계약서/증거
│   출력: 법률 분석 보고서, 내용증명 .docx
│   스킬: certified-letter (스킬명 직접 지정), fact-confirmation (스킬명 직접 지정)
│   MCP: search_laws, get_law_detail, search_cases
│   제한: 소장/준비서면 등 민사 고유 문서 템플릿은 미구현
│
├── 가사팀 (Family) ← Phase 2, 리서치 전용
│   책임: 이혼, 양육권, 상속, 후견
│   입력: 사건 메타데이터
│   출력: 법률 분석 보고서, 사실확인서 .docx
│   스킬: fact-confirmation (스킬명 직접 지정)
│   MCP: search_laws, search_cases
│   제한: 이혼소장, 양육권신청서 등 가사 고유 문서 미구현
│
├── 행정팀 (Administrative) ← Phase 3
├── 지재팀 (IP) ← Phase 3
├── 노동팀 (Labor) ← Phase 3
├── 회사·M&A팀 (Corporate) ← Phase 3, K-Data MCP 필요
│
└── 공통 지원 (Shared Services)
    ├── 법령·판례 리서치
    │   책임: 모든 specialist에게 법령/판례 검색 결과 제공
    │   MCP: korean-law (search_laws, search_cases, get_law_detail,
    │         get_case_detail, search_interpretations)
    │   참고: 각 specialist가 직접 MCP를 호출하므로, 이 서비스는
    │         복잡한 cross-domain 리서치가 필요할 때만 독립 호출
    │
    ├── 공공데이터 리서치
    │   책임: 기업 공시, 부동산 등기, 수출입 등 비법률 데이터
    │   MCP: K-Data (legal_research, corporate_disclosure 등)
    │   참고: K-Data MCP는 현재 .mcp.json에 미포함 (claude.ai 커넥터로만 연결).
    │         Phase 2에서 .mcp.json 추가 또는 선택적 의존으로 처리
    │
    ├── 문서작성 (기존 6개 스킬)
    │   스킬 목록:
    │   - complaint-drafter (고소장)
    │   - certified-letter (내용증명)
    │   - criminal-settlement (형사합의서, generate.js 포함)
    │   - fact-confirmation (사실확인서)
    │   - criminal-procedure-simulator (형사절차 시뮬레이션)
    │   - sentencing-predictor (양형 예측, WebFetch 의존)
    │   규칙: L5 → L2 호출. 스킬명 직접 지정. 스킬 핵심 로직은 변경하지 않는다.
    │   호출 방식 (v2 R5): AGENT가 SKILL을 호출할 때는 키워드 매칭이 아닌
    │     **스킬명 직접 지정** 방식을 사용한다. 이는 MANDATORY TRIGGERS가
    │     없는 criminal-settlement, fact-confirmation도 일관되게 호출하기
    │     위함이다. AGENT.md 프롬프트에서 "complaint-drafter 스킬을 실행하라"와
    │     같이 스킬명을 명시적으로 지정한다.
    │
    └── 품질검수·CTA
        책임: 최종 산출물의 품질 검증 + CTA 포함 확인
        입력: specialist가 생성한 문서/보고서
        출력: QA 결과 (pass/fail + 수정사항)
        참조: references/cta-config.md, docs/QUALITY.md, docs/QUALITY_SCORE.md
        규칙: CTA 미포함 시 반드시 fail. [확인 필요] 항목 개수 보고.
               [MCP 미확인] 항목 개수 보고 (v2 M5 추가).
```

### 4.2 Hub-Spoke 토폴로지

```
                    ┌─────────────┐
                    │  의뢰인     │
                    └──────┬──────┘
                           │ 사건 접수 (자유 텍스트)
                           ▼
                    ┌─────────────┐
                    │  Intake &   │
                    │  Triage     │
                    └──────┬──────┘
                           │ 사건 메타데이터 JSON
                           ▼
              ┌────────────────────────┐
              │   Managing Partner     │
              │   (대표변호사 AI)       │
              │                        │
              │  - 위임 결정           │
              │  - 결과 통합           │
              │  - QA 검수             │
              │  - 변호사 게이트 연결   │
              └──┬────┬────┬────┬──┘
                 │    │    │    │
         ┌───────┘    │    │    └───────┐
         ▼            ▼    ▼            ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐  ...
    │ 형사팀  │ │ 민사팀  │ │ 가사팀  │
    │(Phase1) │ │(Phase2) │ │(Phase2) │
    └────┬────┘ └────┬────┘ └────┬────┘
         │           │           │
         ▼           ▼           ▼
    ┌─────────────────────────────────┐
    │        MCP + Skills (L0-L2)     │
    │  korean-law / K-Data / 6 skills │
    └─────────────────────────────────┘
```

**핵심 규칙**: Specialist 간 직접 통신 금지. 형사팀이 민사팀의 분석이 필요하면 Managing Partner를 통해 요청한다. 이는 L5↔L5 금지 규칙(P4)의 운영적 표현이다.

### 4.3 AGENT.md 작성 가이드라인

**파일 구조 (예상)**:

```
agents/
├── managing-partner/
│   └── AGENT.md          # 대표변호사: 트리아지, 위임, 통합, QA
├── intake-triage/
│   └── AGENT.md          # 사건 접수·분류
├── criminal/
│   └── AGENT.md          # 형사팀 (하위 역할은 프롬프트 내 분기)
├── civil/
│   └── AGENT.md          # 민사팀
├── family/
│   └── AGENT.md          # 가사팀
└── shared/
    ├── quality-gate.md   # 품질검수 규칙
    └── research-guide.md # 리서치 가이드 (MCP 사용법)
```

각 AGENT.md는 SKILL.md와 유사한 프롬프트 기반 파일이며, frontmatter에 `name`, `description`, `triggers`, `delegates_to`(호출 가능한 스킬/MCP) 필드를 포함한다.

**AGENT.md 공통 규칙** (v2 R4, R5 추가):
- **PII 마스킹**: MCP 호출 시 쿼리에 당사자 실명·주민번호·연락처 포함 금지. "피해자", "피고소인" 등 일반명사로 대체 (P6 준수).
- **스킬 호출 방식**: 키워드 매칭 아닌 스킬명 직접 지정 ("complaint-drafter 스킬을 실행하라"). 이는 MANDATORY TRIGGERS가 없는 스킬도 일관되게 호출하기 위함.
- **사건 메타데이터**: 로컬 컨텍스트에서만 유지. 외부 전송 금지.

---

## 5. Workflow (사건 의뢰 → 보고)

### 5.1 전체 시퀀스

```
Step 1  의뢰인 사건 접수
        ├── 입력: 자유 텍스트 설명 + 첨부 파일 (계약서, 녹취록, 증거 등)
        └── 트리거: "사건 의뢰", "상담", "도와주세요" 등

Step 2  Intake & Triage — 메타데이터 추출
        ├── 처리:
        │   ① 첨부 파일 전수 읽기 (Read 도구)
        │   ② 당사자 정보 추출 (성명, 관계, 연락처)
        │   ③ 사건 유형 분류 (형사/민사/가사/행정/지재/노동/회사)
        │   ④ 쟁점 식별 (죄명, 청구원인, 분쟁 포인트)
        │   ⑤ 긴급도 판단 (공소시효 임박? SOL < 3개월?)
        │   ⑥ 에스컬레이션 기준 체크 (피해액 5억+, 공무원, 약한 증거)
        ├── 출력: 사건 메타데이터 JSON
        │   {
        │     "case_id": "LYN-2026-0512-001",
        │     "domain": "criminal",
        │     "sub_type": "사기",
        │     "parties": { "victim": {...}, "suspect": {...} },
        │     "issues": ["피해액 2억", "업무상 횡령 가중 검토"],
        │     "urgency": "normal",
        │     "escalation_flags": [],
        │     "attachments": ["녹취록.txt", "계약서.pdf"]
        │   }
        └── 부족한 정보: 의뢰인에게 1회 보충 질문 (최대 3개)

Step 3  Managing Partner — 위임 결정
        ├── 입력: 사건 메타데이터 JSON
        ├── 처리:
        │   ① domain 필드로 1차 팀 선택
        │   ② 복합 사건(형사+민사 등) 판단 → 다수 팀 동시 위임
        │   ③ 각 팀에 필요한 컨텍스트(첨부 파일, 쟁점) 분배
        │   ④ 실행 순서 결정 (병렬 vs 순차)
        └── 출력: 위임 지시서 (어떤 팀에 무엇을 요청하는지)

        위임 규칙:
        - 단일 분야 → 해당 팀 1개에 위임
        - 복합 분야 → 주 분야 팀 + 보조 분야 팀 (최대 3팀)
        - 형사 사건 중 고소장이 필요하면 → 형사팀(고소·고발)에 위임
        - 양형 예측이 필요하면 → 형사팀(양형·합의)에 위임
        - 내용증명이 필요하면 → 해당 분야 팀 + certified-letter 스킬 지정

Step 4  Specialist 실행 — MCP 리서치 + 스킬 호출
        ├── 리서치 단계:
        │   ① search_laws → 관련 법령 확인
        │   ② search_cases → 유사 판례 검색 (2-3건)
        │   ③ get_law_detail / get_case_detail → 상세 조문·판결요지 확인
        │   ④ search_interpretations → 법령해석례 확인 (필요시)
        │   ⑤ K-Data MCP → 기업공시, 부동산 등 (해당시, Phase 2+)
        │   ⑥ MCP 호출 실패 감지 (v2 M5):
        │      - timeout 또는 5xx 응답 시, 사용자에게 통보:
        │        "MCP 서버 일시 장애로 법령·판례 검색이 불가합니다.
        │         복구 후 재시도를 권장합니다."
        │      - 문서 생성을 진행하되, 인용 부분을 [MCP 미확인]으로 태깅
        │
        ├── 문서 생성 단계 (스킬명 직접 지정으로 호출):
        │   - 고소장 필요 → complaint-drafter 스킬 실행
        │   - 합의서 필요 → criminal-settlement 스킬 실행
        │   - 내용증명 필요 → certified-letter 스킬 실행
        │   - 사실확인서 필요 → fact-confirmation 스킬 실행
        │   - 절차 안내 필요 → criminal-procedure-simulator 스킬 실행
        │   - 양형 예측 필요 → sentencing-predictor 스킬 실행
        │
        ├── 분석 보고서 작성 (스킬 없는 분석의 경우):
        │   법적 쟁점 분석, 리스크 평가, 대응 전략 제안
        │
        └── 출력: specialist 보고서 + 생성된 .docx 파일들

Step 5  Managing Partner — 결과 통합 + QA
        ├── 처리:
        │   ① 각 specialist 결과 수합
        │   ② 사건 ID(case_id) 일관성 검증
        │   ③ 법령·판례 인용 교차 검증 (specialist 간 인용 충돌 확인)
        │   ④ [확인 필요] 항목 집계 및 의뢰인 확인 요청 목록 생성
        │   ⑤ [MCP 미확인] 항목 집계 (v2 M5):
        │      - 임계값 3건 초과 시, 변호사에게 "MCP 장애로 인용
        │        미검증 항목이 N건입니다. 진행하시겠습니까?" 확인
        │   ⑥ CTA 포함 여부 검증 (references/cta-config.md 대조)
        │   ⑦ 품질 점수 산출 (docs/QUALITY_SCORE.md 기준)
        │   ⑧ 에스컬레이션 재확인 (specialist가 발견한 추가 리스크)
        └── 출력: 통합 사건 보고서 초안

Step 6  변호사 검수 게이트 ← 여기가 인간 게이트 (v2 M3 상세화)

        ※ 이 게이트는 채팅 인터페이스 상의 "능동적 게이트"이다. 기존 SKILL.md의
          면책 문구("변호사 검토 필수")는 텍스트 수준 면책일 뿐이며, 이 게이트는
          워크플로우를 실제로 일시 정지하여 변호사 입력을 기다리는 메커니즘이다.

        ├── (a) 채팅 표시 텍스트:
        │   Managing Partner가 통합 보고서 출력 후, 아래 형식의 게이트 메시지를 표시한다:
        │
        │   ╔══════════════════════════════════════════════════════╗
        │   ║  변호사 검수 대기                                     ║
        │   ╠══════════════════════════════════════════════════════╣
        │   ║                                                      ║
        │   ║  위 보고서와 첨부 문서를 검토해 주시기 바랍니다.           ║
        │   ║                                                      ║
        │   ║  [확인 필요] 항목: N건                                 ║
        │   ║  [MCP 미확인] 항목: M건                                ║
        │   ║  생성 문서: 고소장.docx 외 K건                          ║
        │   ║                                                      ║
        │   ║  아래 중 하나를 입력해 주세요:                           ║
        │   ║  - "승인" → 의뢰인에게 최종 보고서를 전달합니다           ║
        │   ║  - "수정: [지시사항]" → 해당 부분을 수정합니다            ║
        │   ║  - "반려: [사유]" → 전체 재검토 후 재작성합니다           ║
        │   ║                                                      ║
        │   ╚══════════════════════════════════════════════════════╝
        │
        ├── (b) 변호사 입력 해석:
        │   - "승인" → Step 7로 진행. 보고서를 의뢰인에게 전달.
        │   - "승인" + 코멘트 (예: "승인. 단 3페이지 날짜 수정") → 코멘트 반영 후 Step 7
        │   - "수정: ..." → 해당 specialist 재실행 또는 Managing Partner가 직접 수정.
        │     수정 완료 후 다시 Step 6 게이트로 복귀.
        │   - "반려: ..." → 위임 단계(Step 3)부터 재실행. 사유를 specialist에 전달.
        │
        ├── (c) 기존 SKILL.md 면책 문구와의 차이:
        │   - 기존 면책: "생성된 고소장 초안은 반드시 담당 변호사의 검토와 수정을
        │     거친 후 제출해야 합니다" — 이는 사용자에 대한 텍스트 안내. 구조적으로
        │     우회를 방지하지 않음.
        │   - 이 게이트: Managing Partner 워크플로우가 "승인" 입력 전까지 Step 7로
        │     진행하지 않음. 프롬프트 수준에서 워크플로우를 일시 정지하는 능동적 메커니즘.
        │
        └── (d) 구조적 우회 불가 보장 — 한계 명시:
            - **보장되는 것**: Managing Partner 에이전트 시스템을 통해 사건을 의뢰한 경우,
              변호사 검수 게이트를 거치지 않고는 최종 보고서가 출력되지 않는다.
              Managing Partner의 프롬프트에 "승인 없이 Step 7 진행 금지" 규칙이 명시된다.
            - **보장되지 않는 것 (soft gate 한계)**:
              · 사용자가 에이전트 시스템을 우회하여 기존 SKILL.md를 직접 트리거하면
                변호사 게이트 없이 문서가 생성된다. 이는 §5.4의 MANDATORY TRIGGERS
                비활성화로 완화하지만, 사용자가 의도적으로 스킬을 호출하면 막을 수 없다.
              · Claude Code의 프롬프트 수준 제약이므로, Claude Code 런타임이
                이 규칙을 기술적으로 강제하는 것은 아니다.
              · 기존 SKILL.md의 면책 문구가 이 한계에 대한 보완 역할을 한다.

Step 7  의뢰인 보고서 출력
        ├── 채팅 요약:
        │   - 사건 개요
        │   - 법적 분석 요약
        │   - 생성된 문서 목록 (computer:// 링크)
        │   - 후속 조치 안내
        │   - CTA (면책 + 변호사 연락처)
        │
        └── .docx 파일 목록:
            - 고소장, 합의서, 내용증명, 사실확인서 등 (해당하는 것만)
            - 각 .docx에 CTA 마지막 페이지 포함 (기존 규칙 유지)
```

### 5.2 병렬 vs 순차 실행

| 상황 | 실행 방식 | 이유 |
|------|-----------|------|
| 단일 분야 + 단일 문서 | 순차 | 불필요한 오버헤드 방지 |
| 고소장 + 양형 예측 (동일 사건) | **병렬** | 독립적 산출물, 사건 정보 공유 |
| 형사 분석 + 민사 내용증명 (복합) | **병렬** | 별개 분야의 독립적 작업 |
| 양형 예측 → 합의서 (의존) | **순차** | 합의서 작성에 양형 결과 필요 |

### 5.3 사건 ID 추적

모든 산출물(보고서, .docx, 채팅 메시지)에 사건 ID(`LYN-YYYY-MMDD-NNN`)를 포함한다. Managing Partner가 사건 접수 시 부여하며, 모든 specialist 호출과 MCP 쿼리 로그에 동일 ID를 태깅한다.

### 5.4 Managing Partner 활성 시 기존 SKILL.md 트리거 비활성화

> **v2 추가 (M2 Resolution A)**

Managing Partner 에이전트 시스템이 활성 상태일 때, 기존 SKILL.md의 MANDATORY TRIGGERS에 의한 직접 스킬 실행은 비활성화된다. 모든 스킬 호출은 Managing Partner를 경유해야 한다.

**메커니즘**: 각 SKILL.md에 frontmatter 필드 `gate: attorney-review`를 추가한다. Managing Partner가 활성 상태에서 이 필드가 있는 스킬이 직접 트리거되면, Managing Partner가 해당 요청을 인터셉트하여 자신의 워크플로우(Step 2~Step 7)로 라우팅한다.

**비활성화 범위**:
- complaint-drafter의 MANDATORY TRIGGERS ("고소장", "형사고소" 등)
- certified-letter의 MANDATORY TRIGGERS ("내용증명", "통지서" 등)
- criminal-procedure-simulator의 MANDATORY TRIGGERS ("형사절차", "수사 절차" 등)
- sentencing-predictor의 MANDATORY TRIGGERS ("양형", "양형 예측" 등)
- criminal-settlement, fact-confirmation은 MANDATORY TRIGGERS가 없으나 동일 규칙 적용

**비활성화되지 않는 경우**: Managing Partner 에이전트 시스템이 비활성 상태(미설치, 비활성화)인 경우, 기존 SKILL.md의 MANDATORY TRIGGERS는 그대로 동작한다. 이는 기존 플러그인 사용자의 경험을 보존한다.

---

## 6. MVP Scope + Roadmap

### Phase 0: 플랫폼 가능성 검증 스파이크 (1-2일)

> **v2 추가 (M1)**

**목적**: `agents/*/AGENT.md` 파일이 플러그인 배포 패키지에 포함되었을 때 Claude Code가 이를 subagent로 인식하는지 검증한다.

**검증 항목**:
1. 플러그인 내 `agents/test-agent/AGENT.md`를 생성하고, `/plugin install`로 설치 후 해당 에이전트가 Claude Code에서 인식되는지 확인
2. 인식되는 경우: AGENT.md의 frontmatter 포맷(name, description, triggers, delegates_to)이 Claude Code가 기대하는 스펙과 일치하는지 확인
3. 인식되지 않는 경우: `.claude/agents/*.md` 사용자 레벨 경로에서 동작하는지 확인

**결정 분기**:
- **성공 (plugin agents 인식됨)** → Phase 1에서 `agents/*/AGENT.md` 구조로 진행. L5 레이어 신설.
- **실패 (plugin agents 미인식)** → Fallback: Managing Partner를 `skills/managing-partner/SKILL.md` (메타스킬)로 구현.
  - L5 레이어 대신, L2에 "메타스킬" 하위 유형 추가
  - layer-rules.md 업데이트 범위: L5 신설 → "L2 메타스킬은 다른 L2 스킬을 호출할 수 있다" 예외 규칙 추가
  - Managing Partner SKILL.md가 다른 스킬을 이름으로 지정하여 호출
  - specialist 에이전트는 Managing Partner 프롬프트 내 분기로 처리 (별도 파일 없음)

**Acceptance Criteria (Phase 0)**:
- [ ] "plugin agents 인식 여부"가 Yes/No로 확정되었다
- [ ] 확정된 메커니즘에 맞춰 Phase 1 구현 계획이 업데이트되었다
- [ ] Fallback 경로를 선택한 경우, SKILL.md 메타스킬의 프롬프트 뼈대가 작성되었다

**Phase 0 산출물**:
- 검증 결과 문서 (`.omc/research/agent-mechanism-spike.md`)
- Phase 1 구현 계획 확정 (이 문서 업데이트)

---

### Phase 1: MVP (2-3주)

**범위**: 대표변호사 AI + 형사팀 1팀 + 기존 6 스킬 통합

**구현 대상**:
1. `agents/managing-partner/AGENT.md` — 사건 접수, 분류, 형사팀 위임, 결과 통합, QA, 변호사 검수 게이트
   (Fallback 시: `skills/managing-partner/SKILL.md`)
2. `agents/intake-triage/AGENT.md` — 자유 텍스트 → 사건 메타데이터 추출
   (Fallback 시: Managing Partner SKILL.md 내 분기)
3. `agents/criminal/AGENT.md` — 형사팀 specialist (고소·고발 / 수사대응 / 양형·합의를 프롬프트 내 분기로 처리)
   (Fallback 시: Managing Partner SKILL.md 내 분기)
4. `agents/shared/quality-gate.md` — QA 규칙 (CTA 검증, [확인 필요] 집계, [MCP 미확인] 집계)
5. 기존 6개 SKILL.md에 최소 변경 적용:
   - frontmatter에 `gate: attorney-review` 필드 추가 (1줄)
   - "생성 후 검증" 섹션에 Managing Partner QA 게이트 연동 1문장 추가
6. **문서 업데이트** (v2 R3):
   - `docs/design-docs/layer-rules.md` 업데이트 — L5 Agent 레이어 추가 (또는 Fallback 시 메타스킬 규칙)
   - `docs/design-docs/core-beliefs.md` 업데이트 — Core Belief #4: "strict 5-layer architecture (L0-L4)" → "6-layer (L0-L5)" (또는 Fallback 시 "5-layer + meta-skill")
   - `ARCHITECTURE.md` 업데이트 — Data Flow 다이어그램에 에이전트 레이어 추가
   - `AGENTS.md` 업데이트 — "4 skills" → "6 skills" 정정, 에이전트 시스템 설명 추가
   - `docs/SECURITY.md` 업데이트 — PII 흐름: AGENT 시스템에서 당사자 정보의 흐름과 MCP 쿼리 시 마스킹 규칙 (P6)
   - `CLAUDE.md` 업데이트 — Layer 요약 테이블에 L5 추가, 에이전트 시스템 개요 추가
   - `.claude-plugin/plugin.json` 업데이트 — 에이전트 시스템 설명 추가, 버전 범프

**Acceptance Criteria (Phase 1 완료 조건)**:
- [ ] "횡령 고소장을 써 달라"고 의뢰하면, Intake가 형사/횡령으로 분류 → Managing Partner가 형사팀에 위임 → complaint-drafter가 고소장 .docx 생성 → Managing Partner가 QA → 변호사 검수 게이트 (§5 Step 6 형식의 게이트 메시지 표시) → 변호사 "승인" 입력 → 의뢰인에게 보고서 + .docx 전달
- [ ] 동일 사건에서 양형 예측을 추가 요청하면, 동일 case_id로 sentencing-predictor 호출 → 결과가 기존 보고서에 통합
- [ ] 사건 ID가 모든 산출물에 일관되게 표시
- [ ] 형사절차 시뮬레이션 요청 시 criminal-procedure-simulator가 정상 호출
- [ ] CTA가 모든 .docx와 채팅 응답에 포함
- [ ] 변호사 검수 게이트에서 "승인" 입력 없이는 최종 보고서가 의뢰인에게 전달되지 않음
- [ ] 기존 SKILL.md의 MANDATORY TRIGGERS가 Managing Partner 활성 시 직접 실행되지 않고 Managing Partner로 라우팅됨
- [ ] 변호사 검수 게이트에서 "수정: [지시]" 입력 시 해당 부분이 수정되고 다시 게이트로 복귀됨

**Out of Scope (Phase 1)**:
- 민사/가사/행정/지재/노동/회사·M&A 전문 에이전트
- K-Data MCP 통합
- 사건 DB / 히스토리 저장
- 의뢰인 포털 / 웹 UI
- 소장, 준비서면, 항소이유서 등 민사·행정 고유 문서 템플릿

### Phase 2: 민사·가사 확장 (1-2개월)

**범위**: 민사팀 + 가사팀 에이전트 추가, K-Data MCP 연동

**구현 대상**:
1. `agents/civil/AGENT.md` — 민사팀 (certified-letter + fact-confirmation 스킬 활용)
2. `agents/family/AGENT.md` — 가사팀 (fact-confirmation 활용)
3. `.mcp.json`에 K-Data MCP 추가 (또는 선택적 의존으로 처리)
4. 민사 고유 문서 스킬 1-2개 신규 개발 (소장 초안, 답변서 초안 등)
5. Managing Partner 프롬프트 업데이트 — 민사/가사 위임 로직 추가

**Acceptance Criteria**:
- [ ] "임대차보증금 반환 내용증명"을 의뢰하면 민사팀 → certified-letter 스킬 호출 → .docx 생성
- [ ] "이혼 소송 관련 사실확인서"를 의뢰하면 가사팀 → fact-confirmation 스킬 호출
- [ ] K-Data MCP가 연결된 환경에서 기업 공시 데이터 조회 가능
- [ ] K-Data MCP 미연결 시 graceful degradation (기능 없음 안내, 시스템 정상 동작)

**Out of Scope**: 행정/지재/노동/회사 전문 에이전트, 사건 DB

**전제조건**:
- K-Data MCP를 `.mcp.json`에 추가할지, claude.ai 커넥터 전용으로 남길지 결정
  - `.mcp.json` 추가 시: 마켓플레이스 설치 사용자도 K-Data 활용 가능
  - 커넥터 전용: Cowork/claude.ai 사용자만 활용, 플러그인 설치자는 불가

### Phase 3: 전문 분야 확장 (3개월+)

**범위**: 행정·지재·노동·회사 에이전트, 사건 DB, 의뢰인 포털

**구현 대상**:
1. `agents/administrative/AGENT.md` — 행정팀
2. `agents/ip/AGENT.md` — 지재팀
3. `agents/labor/AGENT.md` — 노동팀
4. `agents/corporate/AGENT.md` — 회사·M&A팀 (K-Data MCP 의존)
5. 분야별 문서 스킬 추가 (행정심판청구서, 특허분쟁 답변서, 퇴직금 청구서 등)
6. 사건 히스토리 DB (L4 보강 — JSON 또는 SQLite)
7. 의뢰인 포털 (별도 프로젝트로 분리 가능)

**Acceptance Criteria**:
- [ ] 7개 법률 분야에 대해 Intake가 정확히 라우팅
- [ ] 복합 분야 사건(형사+민사+노동)에서 3팀 병렬 위임 동작
- [ ] 사건 ID로 과거 의뢰 이력 조회 가능

**Out of Scope**: 해외법률, 국제중재, 세무·관세 전문 영역

---

## 7. Pre-mortem (4 시나리오) — Deliberate Mode

### 7-a. 변호사법 위반 / 비변호사 법률사무 리스크

**트리거**: AI 시스템이 "법률 자문"을 제공하고 있다는 신고·민원이 대한변호사협회에 접수됨. "AI 법무법인"이라는 명칭이 변호사법 §23을 위반한다는 지적.

**임팩트**: 이영남 변호사에 대한 변호사법 위반 수사 개시 가능. 마켓플레이스 리스팅 중단 요청. 프로젝트 전체 중단 리스크.

**완화 (Mitigation)**:
1. **명칭**: 외부 배포명에서 "법무법인" 완전 배제 → "Lyn 법률 어시스턴트 팀" 사용
2. **면책**: 모든 산출물에 "AI 초안, 법률자문 아님" 면책조항 필수 (기존 CTA 체계 유지)
3. **변호사 게이트**: 최종 산출물은 반드시 변호사 검수 후 전달 — 워크플로우에서 게이트 건너뛰기 불가 (P2, §5 Step 6)
4. **기능 범위 명시**: "문서 초안 작성 보조" + "법령·판례 검색" = 도구 제공. "법률 의견" / "소송 전략 권고" = 금지
5. **이용약관**: 플러그인 설치 시 "법률자문이 아님" 동의 필수

**조기 경고 신호**:
- 의뢰인이 변호사 검수 없이 문서를 직접 제출하는 사례 보고
- 대한변호사협회 AI 법률 서비스 가이드라인 발표
- 유사 서비스에 대한 법적 조치 뉴스

### 7-b. AI Hallucination on Case Citations — 존재하지 않는 판례 인용

**트리거**: Specialist 에이전트가 MCP로 검색한 판례와 자체 생성한 판례를 혼합하여 보고서에 포함. 변호사가 검수하지 않고(또는 놓치고) 법원에 제출. 상대측이 허위 판례 인용을 지적.

**임팩트**: 해당 소송에서 신뢰도 상실. 변호사 징계 리스크. 고객 이탈. 프로젝트 신뢰도 파괴.

**완화 (Mitigation)**:
1. **MCP-only 인용 규칙**: `search_cases` → `get_case_detail`로 확인된 판례만 인용 가능. 에이전트 프롬프트에 "MCP 검색 결과에 없는 판례는 절대 인용하지 않는다" 명시 (기존 스킬에 이미 적용된 규칙).
   **예외 (v2 M6)**: sentencing-predictor의 양형기준표 데이터는 WebFetch로 대법원 양형위원회 사이트에서 직접 조회하므로 MCP 검증 경로를 거치지 않는다. 이 데이터는 "판례 인용"이 아닌 "양형기준 참조"이므로 MCP-only 인용 규칙의 적용 대상이 아니다. WebFetch 실패 시에는 기본 형량 범위만 제시하고 사용자에게 양형위원회 사이트 수동 확인을 안내한다.
2. **판례 출처 태깅**: 보고서 내 모든 판례에 `[MCP 검증됨]` 또는 `[미검증 — 확인 필요]` 태그 부착
3. **QA 게이트에서 교차 검증**: Managing Partner의 QA 단계에서 인용된 사건번호를 `get_case_detail`로 재검증
4. **변호사 검수 체크리스트**: "모든 판례 사건번호가 실존하는지" 항목을 변호사 검수 체크리스트에 포함

**조기 경고 신호**:
- QA 단계에서 `[미검증]` 태그가 붙은 판례가 빈번하게 발견
- MCP 서버 다운타임 중에 에이전트가 자체 판례를 생성하는 패턴
- 변호사 검수에서 판례 수정률이 20% 초과

### 7-c. 에이전트 오케스트레이션 비용·지연 폭주

**트리거**: 복합 사건(예: 형사+민사+가사)에서 Managing Partner가 3개 specialist를 동시 위임. 각 specialist가 MCP 호출 3-5회 + 스킬 호출 1-2회. 총 토큰 사용량이 단일 사건당 수십만 토큰에 달하고, 응답 시간이 10분 이상.

**임팩트**: 사용자 경험 저하 (긴 대기). Claude API 비용 급증. rate limit 도달 시 중간에 실패.

**완화 (Mitigation)**:
1. **단계적 실행**: Phase 1에서는 형사팀 1팀만 → 비용 제한. 복합 사건 병렬 위임은 Phase 2+에서 점진적 도입
2. **MCP 호출 예산**: specialist당 MCP 호출 최대 5회. 초과 시 "추가 리서치가 필요합니다. 계속할까요?" 의뢰인 확인
3. **캐싱**: 동일 법조항/판례를 다른 specialist가 중복 조회하지 않도록 Managing Partner가 공통 리서치 결과를 사전 배포

> **v2 변경 (M2)**: "경량 모드" 항목 삭제. Resolution A 채택으로 모든 스킬 호출은 Managing Partner를 경유한다. 비용 절감은 MCP 호출 예산과 캐싱으로 달성한다.

**조기 경고 신호**:
- 사건당 평균 토큰 사용량이 10만 토큰 초과
- 평균 응답 시간 5분 초과
- rate limit 에러 빈도 증가

### 7-d. MCP 다운타임 Cascade

> **v2 추가 (M5)**

**트리거**: korean-law MCP 서버(`https://korean-law.up.railway.app/mcp`)가 다운됨. Railway 호스팅이므로 cold start, 배포 실패, 외부 API(국가법령정보센터) 장애 등으로 다운타임 발생 가능.

**임팩트**: Specialist의 모든 법령·판례 인용이 `[MCP 미확인]`으로 태깅됨 → 보고서가 빈 껍데기 → 변호사 검수 부담이 폭증하여 검수 자체가 불가능해질 수 있음 → 사실상 서비스 중단.

**완화 (Mitigation)**:
1. **즉시 사용자 통보**: Specialist가 MCP 호출 실패(timeout 또는 5xx)를 감지하면 사용자에게 "MCP 서버 일시 장애로 법령·판례 검색이 불가합니다. 복구 후 재시도를 권장합니다."를 통보 (§5 Step 4 ⑥).
2. **[MCP 미확인] 태깅**: 문서 생성 자체는 진행하되, MCP로 검증하지 못한 인용 부분을 명시적으로 `[MCP 미확인]`으로 태깅. 기존 `[확인 필요]`와 구별하여 장애 원인을 명확히 함.
3. **QA 임계값 게이트**: Managing Partner QA(§5 Step 5 ⑤)에서 `[MCP 미확인]` 건수가 3건을 초과하면 변호사에게 "MCP 장애로 인용 미검증 항목이 N건입니다. 진행하시겠습니까?" 확인을 요청. 이를 통해 품질이 보장되지 않는 보고서의 무조건 전달을 방지.
4. **재시도 안내**: MCP 복구 후 동일 case_id로 재실행하면 `[MCP 미확인]` 항목이 검증된 인용으로 대체됨.

**조기 경고 신호**:
- MCP 호출 실패율 메트릭이 10% 초과 (§8.4 Observability 참조)
- Railway 대시보드에서 서버 상태 이상 감지
- 국가법령정보센터 Open API 공지사항에 점검·장애 안내

---

## 8. Expanded Test Plan — Deliberate Mode

### 8.1 Unit Tests (각 에이전트 단위)

| 테스트 | 입력 | 기대 결과 | 검증 방법 | 실행 방법 |
|--------|------|-----------|-----------|-----------|
| Intake 분류: 형사 | "횡령으로 고소하고 싶습니다" | domain: "criminal", sub_type: "횡령" | 메타데이터 JSON 필드 검증 | Manual |
| Intake 분류: 민사 | "보증금을 안 돌려줍니다" | domain: "civil", sub_type: "임대차" | 메타데이터 JSON 필드 검증 | Manual |
| Intake 분류: 가사 | "이혼·재산분할을 원합니다" | domain: "family" | 메타데이터 JSON 필드 검증 | Manual |
| Intake 분류: 복합 | "횡령 고소 + 손해배상 소송" | domains: ["criminal", "civil"] | 복수 분야 반환 | Manual |
| Intake 긴급도 | "공소시효가 2개월 남았습니다" | urgency: "urgent", escalation_flags 포함 | 에스컬레이션 플래그 검증 | Manual |
| Managing Partner 위임 | domain: "criminal" | 형사팀에 위임 지시 | 위임 대상 팀 검증 | Manual |
| 형사팀 스킬 라우팅 | sub_type: "고소" | complaint-drafter 스킬명 호출 | 호출된 스킬 이름 검증 | Manual |
| 형사팀 양형 라우팅 | "집행유예 가능성은?" | sentencing-predictor 스킬명 호출 | 호출된 스킬 이름 검증 | Manual |
| QA CTA 검증 | CTA 미포함 문서 | QA fail 반환 | pass/fail 결과 검증 | Manual |
| QA [확인 필요] 집계 | 3개 미비 항목 문서 | 미비 항목 3개 보고 | 집계 수 검증 | Manual |
| QA [MCP 미확인] 집계 | MCP 장애 상태 문서 | [MCP 미확인] 항목 수 보고 | 집계 수 검증 | Manual |

> **v2 변경 (M4)**: "실행 방법" 컬럼 추가. Phase 1에서는 모든 unit test가 manual checklist이다.

### 8.2 Integration Tests (에이전트 체인)

| 테스트 | 시나리오 | 검증 포인트 | 실행 방법 |
|--------|----------|-------------|-----------|
| 접수→분류→위임 | "사기 고소장" 의뢰 | Intake→MP→형사팀 체인 정상 동작 | Manual |
| 사건 ID 일관성 | 고소장 + 양형예측 동시 의뢰 | 두 산출물의 case_id 동일 | Manual |
| MCP→스킬 체인 | 형사팀이 search_laws 후 complaint-drafter 호출 | 법령 검색 결과가 고소장에 반영 | Manual |
| 에스컬레이션 전파 | 피해액 6억 의뢰 | Intake escalation → MP가 변호사에게 즉시 알림 | Manual |
| QA→재실행 | QA에서 CTA 누락 발견 | specialist 재호출 후 CTA 포함 확인 | Manual |
| 복합 사건 위임 | "횡령 고소 + 보증금 내용증명" | 형사팀 + 민사팀 병렬 위임, 각각 산출물 생성 | Manual (Phase 2) |
| 변호사 게이트 "승인" | 보고서 생성 후 "승인" 입력 | Step 7 의뢰인 보고서 출력 정상 진행 | Manual |
| 변호사 게이트 "수정" | 보고서 생성 후 "수정: 날짜 변경" 입력 | 수정 후 다시 게이트로 복귀 | Manual |
| MCP 다운타임 | MCP 서버 미응답 상태에서 고소장 의뢰 | [MCP 미확인] 태깅 + 사용자 통보 + QA 임계값 게이트 | Manual |

### 8.3 E2E Gold Cases (10개 시나리오)

| # | 시나리오 | 기대 산출물 | 품질 기준 |
|---|---------|-------------|-----------|
| 1 | 업무상횡령 고소 (피해액 3억) | 고소장 .docx + 절차 안내 | 가중처벌 규정(§356) 포함, 증거번호 일관 |
| 2 | 사기죄 양형예측 (초범, 피해액 1억, 일부 변제) | 양형 보고서 + 시나리오별 예측 | 양형기준표 3유형 적용, 집행유예 확률 범위 제시 |
| 3 | 폭행 합의서 (합의금 500만원, 처벌불원) | 합의서 .docx | 특약 조항 적절, 당사자 정보 일관 |
| 4 | 임대차보증금 반환 내용증명 (Phase 2) | 내용증명 .docx | 주택임대차보호법 인용, 기한 설정 |
| 5 | 이혼 관련 사실확인서 (Phase 2) | 사실확인서 .docx | 가사 사건번호 형식, 객관적 사실만 |
| 6 | 복합: 횡령 고소 + 손해배상 내용증명 | 고소장 + 내용증명 .docx 2건 | 동일 case_id, 사실관계 일관 |
| 7 | 명예훼손 고소 (정보통신망법) | 고소장 .docx | 정보통신망법 §70 적용, 공연성 분석 |
| 8 | 양형예측 → 합의서 순차 의뢰 | 양형 보고서 → 합의서 .docx | 양형 결과가 합의 전략에 반영 |
| 9 | 공소시효 임박 사건 (2개월) | 긴급 배너 + 고소장 | 문서 상단 긴급 경고, 에스컬레이션 |
| 10 | 피해액 6억 특경법 사건 | 고소장 + 변호사 즉시 알림 | 특정경제범죄법 §3 적용, 에스컬레이션 |

각 Gold Case에 대해:
- **정확성**: 법조항·판례 인용이 MCP 검색 결과와 일치하는지
- **완결성**: 필수 구성요소(CTA, 서명란, 증거번호 등) 누락 없는지
- **일관성**: 당사자 정보, case_id가 문서 전체에서 일관되는지
- **적절성**: 에스컬레이션, 가중처벌 규정이 누락 없이 적용되었는지

### 8.4 Copy-Paste 테스트 프롬프트 (Phase 1 검증용)

> **v2 추가 (M4)**

Phase 1 MVP는 manual 검증을 전제한다. 아래 3개 프롬프트를 Claude Code 채팅에 그대로 붙여넣어 테스트한다.

#### 테스트 프롬프트 1: 업무상횡령 고소장

```
사건을 의뢰하겠습니다.

저는 주식회사 한빛테크의 대표이사 김철수입니다. 저희 회사 경리부장 박영희(1985년 3월 15일생, 
서울시 강남구 역삼동 123-45 거주)가 2024년 1월부터 2025년 6월까지 약 1년 6개월간 
회사 법인카드를 개인 용도로 사용하고, 거래처 대금을 자신의 개인 계좌로 수령하는 방법으로 
총 3억 2천만원을 횡령한 것을 발견했습니다.

증거로는 법인카드 사용내역(개인 쇼핑몰 결제 건 47건), 거래처 박철민 대표의 녹취록
(박영희가 개인 계좌로 입금을 요청한 통화 내용), 회사 장부와 실제 입출금 내역의 차이를 
정리한 엑셀 파일이 있습니다.

고소장을 작성해 주세요.
```

**Pass 기준**:
- [ ] Intake가 `domain: "criminal"`, `sub_type: "횡령"` 또는 `"업무상횡령"`으로 분류
- [ ] Managing Partner가 형사팀에 위임
- [ ] complaint-drafter 스킬이 호출되어 .docx 고소장 생성
- [ ] 생성된 고소장에 `형법 제356조`(업무상횡령) 또는 `형법 제355조`(횡령) 조문 인용 포함
- [ ] 피해액 3.2억이므로 특정경제범죄법 §3 미적용 (5억 미만) — 만약 적용했다면 Fail
- [ ] 증거번호("증 제1호증", "증 제2호증" 등)가 입증방법 표와 본문에서 일관
- [ ] 변호사 검수 게이트 메시지가 표시됨 ("변호사 검수 대기" 형식)
- [ ] CTA가 .docx와 채팅 응답 모두에 포함
- [ ] `[확인 필요]` 항목으로 고소인 주민등록번호, 법인 등기번호 등 미제공 정보가 표시

**Fail 기준**:
- Intake가 형사 이외의 분야로 분류
- complaint-drafter가 호출되지 않거나 .docx가 생성되지 않음
- 변호사 검수 게이트 없이 바로 최종 보고서 출력
- CTA 누락
- 존재하지 않는 판례 사건번호 인용 (MCP 검색 결과에 없는 것)

#### 테스트 프롬프트 2: 사기죄 양형 예측

```
사건을 의뢰합니다.

제가 투자 사기 혐의로 재판을 받게 되었습니다. 상황은 이렇습니다:
- 죄명: 사기(형법 제347조)
- 피해액: 총 1억 2천만원 (피해자 3명)
- 저는 초범이고, 현재 불구속 상태입니다
- 피해자 1명에게는 4천만원을 변제했고(피해자 김OO), 나머지 2명과는 합의 시도 중입니다
- 전부 자백했습니다
- 투자금 유용이지 보이스피싱은 아닙니다

양형이 어떻게 될지 예측해 주시고, 집행유예를 받으려면 어떻게 해야 하는지 알려주세요.
```

**Pass 기준**:
- [ ] Intake가 `domain: "criminal"`, `sub_type: "사기"` 또는 유사 분류
- [ ] Managing Partner가 형사팀(양형·합의)에 위임
- [ ] sentencing-predictor 스킬이 호출됨
- [ ] 출력에 양형기준표 구간(감경/기본/가중)과 형량 범위가 포함
- [ ] 피해액 1.2억 → 사기 3유형(5천만~5억) 적용
- [ ] 초범, 자백, 일부 변제가 양형인자로 분석에 반영
- [ ] 집행유예 가능성에 대한 분석이 포함
- [ ] 변호사 검수 게이트 메시지 표시
- [ ] CTA 포함

**Fail 기준**:
- sentencing-predictor가 호출되지 않음
- 양형기준표 참조 없이 일반론만 제시
- 변호사 검수 게이트 없이 바로 최종 출력

#### 테스트 프롬프트 3: 형사절차 안내 + 고소장 복합 의뢰

```
상담 요청입니다.

이웃집에 사는 정민수(40대 남성)가 2025년 11월 아파트 주차장에서 저를 폭행했습니다.
얼굴을 2회 가격당해서 전치 3주 진단을 받았고, 진단서가 있습니다. 목격자도 1명 있습니다.
CCTV 영상은 관리사무소에 보존 요청해둔 상태입니다.

두 가지를 도와주세요:
1. 앞으로 형사 절차가 어떻게 진행되는지 알려주세요 (경찰 신고부터 재판까지)
2. 폭행 고소장도 작성해 주세요
```

**Pass 기준**:
- [ ] Intake가 `domain: "criminal"`, `sub_type: "폭행"` 또는 `"상해"`로 분류
- [ ] Managing Partner가 형사팀에 2개 요청을 위임: (a) 절차 안내 (b) 고소장 작성
- [ ] criminal-procedure-simulator 스킬이 호출되어 형사절차 안내 생성
- [ ] complaint-drafter 스킬이 호출되어 폭행 고소장 .docx 생성
- [ ] 두 산출물이 동일 case_id를 공유
- [ ] 고소장에 `형법 제260조`(폭행) 또는 `형법 제257조`(상해) 인용
- [ ] 증거번호에 진단서, CCTV 영상, 목격자 진술이 포함
- [ ] 변호사 검수 게이트 메시지가 한 번 표시됨 (두 산출물 통합 후)
- [ ] CTA 포함

**Fail 기준**:
- 두 요청 중 하나라도 누락
- 스킬이 직접 트리거되어 Managing Partner를 우회
- 변호사 검수 게이트 없이 바로 최종 출력
- 동일 사건인데 case_id가 다름

### 8.5 Test Execution 가이드

> **v2 추가 (M4)**

**Phase 1 테스트 방법론**: Manual Checklist

Phase 1의 모든 테스트는 수동으로 실행한다. 테스터(이영남 변호사 또는 개발자)가 Claude Code 채팅에 프롬프트를 입력하고, 출력을 육안으로 Pass/Fail 기준에 대조한다.

**실행 절차**:
1. Claude Code에 lyn-lawfirm 플러그인을 설치한다 (`/plugin install lyn-lawfirm`)
2. §8.4의 테스트 프롬프트를 그대로 붙여넣는다
3. 각 Pass 기준의 체크박스를 하나씩 확인한다
4. 하나라도 Fail이면 해당 테스트는 Fail로 기록한다
5. Fail 항목에 대해 원인을 분석하고 AGENT.md 프롬프트를 수정한다
6. 수정 후 동일 프롬프트로 재테스트한다

**테스트 결과 기록**: `.omc/research/test-results-phase1.md`에 날짜, 프롬프트 번호, Pass/Fail 여부, Fail 시 원인과 수정 사항을 기록한다.

**향후 자동화 (Phase 2+)**: 테스트 프롬프트를 스크립트로 실행하고, 출력에서 특정 문자열(법조항 번호, `[확인 필요]` 패턴, CTA 포함 여부)을 정규식으로 검증하는 자동화 스크립트 도입을 검토한다. 예: `node scripts/test-runner.js --prompt "테스트1.txt" --expect "형법 제356조" --expect-file "*.docx"`.

### 8.6 Observability (관측 가능성)

> **v2 변경 (R2)**: 각 메트릭에 실현 가능성 태그 추가.

| 메트릭 | 측정 방법 | 경고 임계값 | 실현 가능성 |
|--------|-----------|-------------|-------------|
| 사건당 총 토큰 사용량 | Managing Partner가 집계하여 보고서 말미에 기재 | > 100K tokens | `[Phase 3+ 수집 가능]` — 현재 Claude Code 플러그인에서 토큰 카운트를 프로그래매틱하게 수집하는 API 없음. 보고서 말미 수동 기재는 가능하나 정확도 낮음 |
| 사건당 MCP 호출 횟수 | specialist별 MCP 호출 카운트 | > 15 calls/case | `[현재 수집 가능]` — Managing Partner 프롬프트에서 specialist 보고 시 MCP 호출 횟수를 포함하도록 지시 가능 |
| 사건당 총 응답 시간 | 접수~보고서 출력 경과 시간 | > 5 minutes | `[현재 수집 가능]` — Managing Partner가 사건 접수 시각과 보고서 출력 시각을 기록하여 차이 계산 |
| [확인 필요] 항목 수 | QA 게이트에서 집계 | > 10 items | `[현재 수집 가능]` — QA 게이트 프롬프트에서 집계하도록 지시 |
| [MCP 미확인] 항목 수 | QA 게이트에서 집계 | > 3 items | `[현재 수집 가능]` — QA 게이트 프롬프트에서 집계하도록 지시 |
| 변호사 수정률 (첨삭률) | 변호사 검수 전후 diff | > 30% 수정 | `[수집 불가 — 외부 도구 필요]` — 프롬프트 수준에서 diff를 자동 계산할 수 없음. 변호사가 수동으로 "수정 비율 체감"을 피드백하는 정성 데이터만 가능 |
| CTA 누락률 | QA 게이트에서 검증 | > 0% (절대 금지) | `[현재 수집 가능]` — QA 게이트 프롬프트에서 CTA 포함 여부를 필수 검증 |
| MCP 호출 실패율 | 실패 응답 카운트 / 총 호출 | > 10% | `[현재 수집 가능]` — specialist가 MCP 호출 결과(성공/실패)를 Managing Partner에 보고 |
| 스킬 호출 성공률 | 정상 .docx 생성 / 총 호출 | < 95% | `[현재 수집 가능]` — specialist가 스킬 호출 결과(.docx 생성 여부)를 Managing Partner에 보고 |

**Phase 1 모니터링 대상**: `[현재 수집 가능]` 태그가 붙은 7개 메트릭만 모니터링한다.
**Phase 3+ 모니터링 대상**: 토큰 사용량(사건 DB 저장 후 자동 집계), 변호사 수정률(전용 도구 도입 시).

**추적 방식**: Managing Partner의 통합 보고서 말미에 "시스템 메트릭" 섹션을 추가하여, 해당 사건 처리에 사용된 리소스를 투명하게 보고한다. 이 데이터는 Phase 3에서 사건 DB에 저장하여 추세 분석에 활용한다.

---

## 9. ADR (Architecture Decision Record)

### Decision

**Claude Code Native Subagents + MCP 구조를 채택한다.** 기존 플러그인 아키텍처(`skills/*/SKILL.md` + `.mcp.json`) 위에 에이전트 레이어를 추가하여, Managing Partner → Specialist → Skills/MCP 체인을 구성한다. **Phase 0 스파이크에서 에이전트 메커니즘을 검증한 후 구체적 구현 방식(plugin `agents/*/AGENT.md` vs Fallback `skills/managing-partner/SKILL.md`)을 확정한다.**

### Drivers

1. **D1 변호사법 위반 리스크**: 기존 CTA/면책 체계를 보존하고, 변호사 검수 게이트를 자연스럽게 삽입할 수 있는 구조가 필요
2. **D2 단일 변호사 운영**: 별도 서버 없이 `git push main`으로 배포 가능한 구조 필수
3. **D3 한국어 법률 도메인 정밀도**: 기존 6개 SKILL.md를 변경 없이 호출 가능해야 함

### Alternatives Considered

| Option | 탈락 사유 | 관련 Driver |
|--------|-----------|-------------|
| B-D. OpenClaw / Hermes / Claude OS | 검증 가능한 1차 소스 없음. 플러그인 호환성 미확인. 동일 사유로 일괄 탈락 | D2, D3 |
| E. FastAPI+LangGraph | 별도 서버 운영 부담, SKILL.md 재구현 필요, 1인 운영 불가 | D2, D3 |

### Why Chosen (핵심 근거 3개)

1. **배포 경로 보존**: `/plugin install lyn-lawfirm`으로 설치하면 에이전트 시스템이 자동으로 포함됨. 사용자에게 추가 설치 과정 없음
2. **레이어 호환성**: 에이전트 레이어가 기존 레이어 규칙을 자연스럽게 확장. L0-L4 변경 최소화
3. **프롬프트 기반 아키텍처 일관성**: SKILL.md처럼 AGENT.md도 프롬프트 파일. 코드 작성 최소화. Core Belief #5(Prompt-First Architecture) 준수

### Consequences

**좋은 점**:
- 기존 사용자 경험 유지 (플러그인 설치 방식 동일)
- 기존 스킬 핵심 로직 변경 불필요 (frontmatter 1줄 + 검증 섹션 1문장만 추가)
- 새 에이전트 추가 = AGENT.md 파일 1개 추가 (빠른 확장)
- 인프라 비용 제로 (서버 불필요)

**나쁜 점**:
- Claude Code plugin agent 시스템의 동작 여부가 Phase 0까지 불확실 → Fallback 경로 준비 필요
- 에이전트 간 상태 공유가 프롬프트 컨텍스트에 의존 → 긴 사건에서 컨텍스트 윈도우 압박 가능
- L5↔L5 직접 호출 금지 규칙이 프롬프트 수준 제약이므로, 강제 불가 (soft enforcement)
- 변호사 검수 게이트도 soft gate — 프롬프트 수준 워크플로우 정지이며, 기술적 강제는 아님 (§5 Step 6-d)

**차후 검토 트리거**:
- Claude Code가 plugin agent 시스템을 공식 발표하면 AGENT.md 포맷을 해당 스펙에 맞춰 업데이트
- OpenClaw/Hermes/Claude OS 중 검증 가능한 플랫폼이 등장하면 재평가
- 사건당 토큰 사용량이 지속적으로 100K 초과 시 외부 오케스트레이션 검토

### Follow-ups (다음 의사결정 포인트)

1. **Phase 0 결과**: plugin agents 인식 여부 확정 → 구현 경로 결정
2. **AGENT.md 포맷 확정**: Claude Code plugin agent 스펙이 공식화되면 AGENT.md의 frontmatter 필드를 확정
3. **K-Data MCP 배포 방식**: `.mcp.json` 통합 vs claude.ai 커넥터 전용 — Phase 2 시작 전 결정
4. **외부 명칭 최종 확정**: "Lyn 법률 어시스턴트 팀" vs 다른 안 — 마켓플레이스 업데이트 전 이영남 변호사와 확정
5. **사건 DB 구조**: Phase 3 시작 전 JSON vs SQLite vs 외부 서비스 결정
6. **민사 문서 스킬 우선순위**: 소장/답변서/준비서면 중 Phase 2에서 어떤 것을 먼저 개발할지

---

## 10. Round 2 Response Matrix

> Architect/Critic Round 1 피드백 → Plan 변경 매핑

| ID | Finding (요약) | 출처 | 조치 (어느 섹션을 어떻게 변경) |
|----|----------------|------|---------------------------------|
| M1 | Phase 0 스파이크 추가: plugin agents 동작 검증 + Fallback 정의 + "또는" 제거 | Architect | §3 Option A에서 "또는" 제거, Phase 0 스파이크 결과 기반으로 메커니즘 확정 명시. §6에 Phase 0 섹션 신설 (검증 항목, 결정 분기, AC, Fallback 경로). §9 ADR Decision에 Phase 0 검증 언급 추가 |
| M2 | P2/P3 모순 해결: Resolution A 채택 (경량 모드 제거, P2 엄격성 유지) | Architect | §1 P2에 MANDATORY TRIGGERS 비활성화 + 면책 vs 능동적 게이트 구분 추가. §1 P3에 최소 변경 허용 범위(frontmatter 1줄 + 검증 섹션 1문장) 명시. §5.4 신설 (MANDATORY TRIGGERS 비활성화 메커니즘). §7-c에서 "경량 모드" 삭제. §6 Phase 1 AC에 트리거 비활성화 검증 항목 추가 |
| M3 | 변호사 검수 게이트 UX 메커니즘 명시: 표시 텍스트, 입력 해석, 면책 차이, soft gate 한계 | Architect | §5 Step 6을 전면 확장: (a) 채팅 표시 텍스트 박스, (b) "승인"/"수정"/"반려" 입력 해석 규칙, (c) 기존 면책 vs 능동적 게이트 차이 설명, (d) 보장/미보장 사항 명시 |
| M4 | 테스트 하네스/실행 방법 명시: manual 선언, copy-paste 프롬프트 3건, pass/fail 기준 | Critic | §8.1/8.2에 "실행 방법" 컬럼 추가 (전부 Manual). §8.4 신설 (3개 copy-paste 테스트 프롬프트 + 상세 pass/fail 기준). §8.5 신설 (Test Execution 가이드: 실행 절차, 결과 기록, 향후 자동화 방향) |
| M5 | Pre-mortem 7-d 추가: MCP 다운타임 cascade | Architect | §7-d 신설 (트리거, 임팩트, 4개 완화 방안, 조기 경고 신호). §5 Step 4 ⑥에 MCP 호출 실패 감지 + 사용자 통보 + [MCP 미확인] 태깅 추가. §5 Step 5 ⑤에 [MCP 미확인] 임계값 게이트(3건 초과 시 변호사 확인) 추가. §8.1에 QA [MCP 미확인] 집계 테스트 추가 |
| M6 | sentencing-predictor의 WebFetch 의존성 명시 | Architect | §4.1 양형·합의 담당에 WebFetch 참고 주석 추가 (양형기준표는 MCP 경로 아님, WebFetch 실패 시 기본 형량만 제시). §7-b mitigation 1번에 양형기준표 예외 명시 |
| R1 | B/C/D 통합: 동일 탈락 사유로 1개 섹션에 묶기 | Critic | §3의 Option B, C, D를 "Option B-D: 외부 에이전트 프레임워크 — 검증 불가로 일괄 탈락"으로 통합. 개별 후보명은 표로 유지 |
| R2 | Observability 메트릭 실현 가능성 태깅 | Critic | §8.6(구 §8.4) 각 메트릭에 `[현재 수집 가능]` / `[Phase 3+ 수집 가능]` / `[수집 불가 — 외부 도구 필요]` 태그 부착. Phase 1 모니터링 대상을 현재 수집 가능한 7개로 한정 |
| R3 | 문서 업데이트 범위 명시 | Critic | §6 Phase 1 구현 대상에 문서 업데이트 목록 추가: core-beliefs.md, ARCHITECTURE.md, AGENTS.md, SECURITY.md, CLAUDE.md, plugin.json. 각 파일의 변경 내용 명시 |
| R4 | PII 보호 규칙 명시 | Architect | §1에 P6 (PII 보호) 신설. §4.3에 AGENT.md 공통 규칙으로 PII 마스킹, 사건 메타데이터 외부 전송 금지 명시. SECURITY.md 업데이트를 §6 R3 문서 목록에 포함 |
| R5 | SKILL.md 트리거 메커니즘 통일: 스킬명 직접 지정 | Critic | §1 P4의 L5 → L2 규칙을 "스킬명 직접 지정"으로 변경. §4.1 전체 구조에서 모든 스킬 호출에 "(스킬명 직접 지정)" 주석 추가. §4.3 AGENT.md 공통 규칙에 호출 방식 명시. §5 Step 4 문서 생성 단계에서 "스킬명 직접 지정으로 호출" 명시 |

---

## Appendix A: RALPLAN-DR Summary (Consensus Alignment)

**Principles** (6):
1. P1: 법률 정확성 > 자동화 속도
2. P2: 변호사 검수 게이트 필수 (경량 모드 없음, Resolution A)
3. P3: 기존 6개 스킬 핵심 로직 무변경 (최소 변경 허용 범위 명시)
4. P4: L0-L4 유지 + L5 Agent 레이어 신설 (스킬명 직접 지정 호출)
5. P5: Never Fabricate (계승)
6. P6: PII 보호 (MCP 쿼리 마스킹)

**Decision Drivers** (3):
1. D1: 변호사법 위반 리스크
2. D2: 단일 변호사 운영 현실
3. D3: 한국어 법률 도메인 정밀도

**Viable Options**: 5개 검토 (B/C/D 통합 표현), 1개 채택 (Option A + Phase 0 스파이크), 4개 명시적 탈락
- B/C/D: 검증 불가 (1차 소스 없음) — 일괄 탈락
- E: D2 위반 (운영 부담 과다)

**Mode**: DELIBERATE (법률 도메인 고위험)
- Pre-mortem: 4개 시나리오 (§7) — v2에서 7-d MCP 다운타임 추가
- Expanded test plan: Unit/Integration/E2E/Observability + Copy-paste 프롬프트 3건 (§8)
- ADR: 완료 (§9)
