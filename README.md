# lyn-lawfirm

한국 법률 검색 Claude Code 플러그인 - 국가법령정보센터 API를 통해 법령, 판례, 법률용어 등을 검색합니다.

Korean law search plugin for Claude Code. Connects to the Korean Law Information Center (국가법령정보센터) API via MCP.

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

## MCP 서버 | MCP Server

이 플러그인은 아래 주소의 원격 MCP 서버에 연결됩니다:

This plugin connects to a remote MCP server hosted at:

```
https://korean-law.up.railway.app/mcp
```

국가법령정보센터 Open API를 래핑하여 한국 법률 데이터에 구조화된 접근을 제공합니다.

The server wraps the Korean Law Information Center Open API and provides structured access to Korean legal data.

## 라이선스 | License

MIT
