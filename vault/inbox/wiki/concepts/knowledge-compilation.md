# Knowledge Compilation

## Definition

원시 소스로부터 지식을 한 번 추출·구조화·종합하고, 이후에는 최신 상태로 유지하는 방식. 매 질의마다 원시 문서에서 지식을 재도출하는 RAG의 "질의 시점 검색(retrieval-at-query-time)" 방식과 대비된다.

## Compile-Once vs Re-Derive-Every-Time

| 측면 | Knowledge Compilation | 기존 RAG |
|------|----------------------|----------|
| 지식 처리 시점 | 소스 수집 시 (사전 컴파일) | 질의 시 (실시간 검색) |
| 축적 | 복리로 쌓임 | 없음 — 매번 처음부터 |
| 교차 참조 | 사전 구축 완료 | 매번 LLM이 조각 맞추기 |
| 모순 탐지 | 수집 시 즉시 표시 | 질의에 따라 우연히 발견 |
| 종합 품질 | 모든 소스 반영 누적 | 검색된 청크에 제한 |
| 인프라 | 마크다운 파일 (임베딩 불필요) | 벡터 DB, 임베딩 파이프라인 |

## Key Insight

5개 문서를 종합해야 하는 미묘한 질문에서 차이가 극명해진다:
- RAG: 매번 관련 조각을 찾아 하나로 엮어야 함
- Compilation: 종합이 이미 위키에 존재, 즉시 참조 가능

## Analogy

Karpathy의 비유: **Obsidian = IDE, LLM = 프로그래머, Wiki = 코드베이스**. 소스 코드(원시 문서)를 컴파일하여 실행 가능한 바이너리(위키)를 만드는 것과 유사. 인터프리터(RAG)가 매번 소스를 해석하는 것과 대비.

## Limitations

- 컴파일 품질이 LLM 능력에 의존 — 오류가 위키에 영구 반영될 위험
- Lint(주기적 건강 검진)로 보완 필요
- 원시 소스를 진실의 원천(source of truth)으로 보존해야 함

## Sources

- [gist_github_com_karpathy-llm-wiki](../summaries/gist_github_com_karpathy-llm-wiki.md)

## Related Concepts

- [llm-wiki](llm-wiki.md) — Knowledge Compilation을 실현하는 패턴
- [personal-knowledge-base](personal-knowledge-base.md) — 컴파일의 대상
- [graphrag](graphrag.md) — 그래프 구축도 일종의 사전 컴파일이나, 질의 시 원시 청크로 회귀
- [hybrid-rag](hybrid-rag.md) — 다양한 인덱스 구조의 하이브리드 접근
