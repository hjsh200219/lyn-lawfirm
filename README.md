# lyn-lawfirm → lawfirm 으로 이전했습니다

`lyn-lawfirm` 플러그인은 [hjsh200219/lawfirm](https://github.com/hjsh200219/lawfirm) 으로 합쳐졌습니다. 이 저장소는 이전 안내만 남기고 더 이상 갱신하지 않습니다.

## 이미 설치하신 분

1. Claude Code 에서 `/plugin` → Marketplaces → `lyn-lawfirm` → **Update marketplace**
2. 「Renamed to lawfirm」 안내가 뜨면 `/plugin install lawfirm@lyn-lawfirm` 한 번 실행

옛 k-data MCP 법령 검색은 없어지고, 법제처 Open API 직접 조회(본인 OC 키 — 처음 쓸 때 안내)로 바뀝니다. 법률 문서 스킬 6종(내용증명·고소장·형사합의서·사실확인서·형사절차 시뮬레이터·양형 예측)은 그대로 들어 있습니다.

## 새로 설치하실 분

```
/plugin marketplace add hjsh200219/lawfirm
/plugin install lawfirm@lawfirm
```

자세한 내용은 [lawfirm README](https://github.com/hjsh200219/lawfirm#readme) 를 보십시오.
