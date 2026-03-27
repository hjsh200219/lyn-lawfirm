# lyn-lawfirm

한국 법률 검색, 법률 문서 작성 및 형사 분석 Claude Code 플러그인 - 법령/판례/법률용어 검색 + 고소장, 내용증명, 형사합의서, 사실확인서 초안 작성 + 형사절차 시뮬레이션, 양형 예측

Korean law search, legal document drafting & criminal analysis plugin for Claude Code. Search laws, cases, legal terms via MCP + draft complaints, certified letters, settlement agreements, fact confirmations + criminal procedure simulation, sentencing prediction.

## 주요 기능 | Features

- **법령 검색 | Law Search**: 현행 법률, 대통령령, 부령 등을 법령명 또는 본문으로 검색 / Search current Korean laws by name or full text
- **판례 검색 | Case Search**: 대법원, 하급심 판결문 검색 / Search court decisions and case law
- **법률용어 검색 | Legal Term Search**: 법률용어 정의 조회 / Look up legal terminology definitions
- **법령해석 검색 | Interpretation Search**: 법령해석 및 의견 검색 / Search legal interpretations and opinions
- **행정심판 검색 | Administrative Appeals**: 행정심판 재결 검색 / Search administrative appeal decisions
- **헌법재판 검색 | Constitutional Decisions**: 헌법재판소 결정문 검색 / Search Constitutional Court decisions
- **법령 상세 | Law Detail**: 개별 법령의 조문 전문 조회 / Get full text of specific laws with articles
- **판례 상세 | Case Detail**: 판결문 전문 조회 / Get full court decision text
- **조약 검색 | Treaty Search**: 국제 조약 검색 / Search international treaties
- **자치법규 검색 | Ordinance Search**: 지방자치단체 조례/규칙 검색 / Search local government ordinances

### 법률 스킬 | Legal Skills

#### 문서 작성 | Document Drafting

- **고소장 작성 | Complaint Drafter**: 상담녹취록/증거자료를 분석하여 형사 고소장 초안을 .docx로 작성 / Draft criminal complaints from consultation transcripts and evidence
- **내용증명 작성 | Certified Letter**: 법률 분쟁 상황에 맞는 내용증명 초안을 .docx로 작성 / Draft certified letters for legal disputes
- **형사 합의서 작성 | Criminal Settlement**: 형사 사건 합의서 초안을 .docx로 작성 / Draft criminal case settlement agreements
- **사실확인서 작성 | Fact Confirmation**: 법원 제출용 사실확인서 초안을 .docx로 작성 / Draft fact confirmation documents for court submission

#### 형사 분석 | Criminal Analysis

- **형사절차 시뮬레이터 | Criminal Procedure Simulator**: 사용자의 상황(피의자/피고인/피해자/참고인)을 파악하여 수사→기소→재판→판결→집행의 전 과정을 안내하고, 인터랙티브 React 위젯으로 시각화 / Simulate the entire criminal procedure flow based on user's situation and visualize it with an interactive React widget
- **양형 예측 | Sentencing Predictor**: 대법원 양형위원회 양형기준표와 실제 판례를 조합하여 예상 선고형과 집행유예 가능성을 IRAC 분석으로 제공 / Predict sentencing outcomes using Supreme Court sentencing guidelines and case law with IRAC analysis

## 설치 방법 | Installation

### Claude Code 플러그인으로 설치 | As a Claude Code Plugin

마켓플레이스를 추가하고 플러그인을 설치합니다:

```shell
/plugin marketplace add hjsh200219/lyn-lawfirm
/plugin install lyn-lawfirm@lyn-lawfirm
```

### MCP 서버 직접 추가 | Manual MCP Setup

또는 원격 MCP 서버를 직접 추가할 수도 있습니다:

```shell
claude mcp add korean-law --transport http https://korean-law.up.railway.app/mcp
```

## 사용 예시 | Usage Examples

설치 후 Claude에게 다음과 같이 요청할 수 있습니다:

- "민법 제750조를 검색해줘" (Search Civil Code Article 750)
- "부당해고 관련 판례를 찾아줘" (Find court cases about unfair dismissal)
- "개인정보보호법 전문을 보여줘" (Show the full text of the Personal Information Protection Act)
- "선의취득의 법률용어 정의를 알려줘" (Define the legal term 'bona fide acquisition')
- "Search for laws related to intellectual property"
- "Find constitutional court decisions about freedom of expression"
- "이 녹취록을 바탕으로 고소장을 작성해줘" (Draft a criminal complaint from this transcript)
- "임대차 분쟁 내용증명을 작성해줘" (Draft a certified letter for a lease dispute)
- "형사 합의서를 작성해줘" (Draft a criminal settlement agreement)
- "사실확인서를 작성해줘" (Draft a fact confirmation document)
- "경찰 조사를 받았는데 앞으로 어떻게 되나요?" (I was questioned by police, what happens next?)
- "사기죄로 기소됐는데 형사절차를 알려줘" (Show me the criminal procedure for a fraud charge)
- "사기 피해액 1억, 초범인데 양형이 어떻게 될까요?" (Predict sentencing for fraud with 100M KRW damage, first offense)
- "집행유예 가능성이 있나요?" (Is a suspended sentence possible?)

## MCP 서버 | MCP Server

이 플러그인은 아래 주소의 원격 MCP 서버에 연결됩니다:

This plugin connects to a remote MCP server hosted at:

```
https://korean-law.up.railway.app/mcp
```

국가법령정보센터 Open API를 래핑하여 한국 법률 데이터에 구조화된 접근을 제공합니다.

The server wraps the Korean Law Information Center Open API and provides structured access to Korean legal data.

## 유의사항 | Disclaimer

본 플러그인의 법률 문서 작성 스킬은 AI가 생성한 초안을 제공합니다. AI는 실수할 수 있으며, 본 스킬 사용으로 인한 피해에 대해서는 책임지지 않습니다. 최종 전문가 검수를 원하시면 이영남 변호사와 상의하세요.

This plugin provides AI-generated legal document drafts. AI can make mistakes, and we are not responsible for any damages arising from the use of this plugin. For professional review, please consult Attorney Young-Nam Lee.

- 📞 010-8652-4348
- 📧 ibiubu70@gmail.com
- 🌐 [www.thechain.lawyer](https://www.thechain.lawyer)

## 라이선스 | License

MIT
