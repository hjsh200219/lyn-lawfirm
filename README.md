# lyn-lawfirm

Korean law search plugin for Claude Code. Connects to the Korean Law Information Center (국가법령정보센터) API via MCP.

## Features

- **Law Search** (법령 검색): Search current Korean laws by name or full text
- **Case Search** (판례 검색): Search court decisions and case law
- **Legal Term Search** (법률용어 검색): Look up legal terminology definitions
- **Interpretation Search** (법령해석 검색): Search legal interpretations and opinions
- **Administrative Appeals** (행정심판 검색): Search administrative appeal decisions
- **Constitutional Decisions** (헌법재판 검색): Search Constitutional Court decisions
- **Law Detail** (법령 상세): Get full text of specific laws with articles
- **Case Detail** (판례 상세): Get full court decision text
- **Treaty Search** (조약 검색): Search international treaties
- **Ordinance Search** (자치법규 검색): Search local government ordinances

## Installation

### As a Claude Code Plugin

Add the marketplace and install:

```shell
/plugin marketplace add hoshin/lyn-lawfirm
/plugin install lyn-lawfirm@lyn-lawfirm
```

### Manual MCP Setup

Or add the remote MCP server directly:

```shell
claude mcp add korean-law --transport http https://korean-law.up.railway.app/mcp
```

## Usage Examples

After installation, you can ask Claude:

- "민법 제750조를 검색해줘" (Search Civil Code Article 750)
- "부당해고 관련 판례를 찾아줘" (Find court cases about unfair dismissal)
- "개인정보보호법 전문을 보여줘" (Show the full text of the Personal Information Protection Act)
- "선의취득의 법률용어 정의를 알려줘" (Define the legal term 'bona fide acquisition')

## MCP Server

This plugin connects to a remote MCP server hosted at:

```
https://korean-law.up.railway.app/mcp
```

The server wraps the Korean Law Information Center Open API and provides structured access to Korean legal data.

## License

MIT
