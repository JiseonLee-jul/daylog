# Graph Pollution ↔ GraphRAG

## Relationship

**Graph Pollution은 GraphRAG 아키텍처가 이기종 코퍼스를 만날 때 필연적으로 부딪히는 문제다.** GraphRAG의 "그래프는 색인, 답변은 원본 청크" 패턴 덕분에 오염의 영향 범위를 이해할 수 있다 — 오염은 검색 정밀도에 치명적이지만 답변 자체는 원본 청크 보존 시 일부 회복 가능.

## Mitigation in GraphRAG Variants

GraphRAG 계열은 각자 다른 방식으로 오염에 대응한다:

- **Microsoft GraphRAG** — Leiden 커뮤니티 요약이 상위 레벨에서 노이즈 희석, 단 추출 입도 문제(benchmark, methodology 같은 일반 명사 엔티티화) 잔존
- **LightRAG** — 인덱스 레벨 격리 없음, dual-level 쿼리 분기만
- **RAPTOR** — 의미 기반 클러스터링 + LLM 요약 과정에서 노이즈 자연 생략
- **KET-RAG** — 물리적 이중 인덱스로 오염 소스를 구조적으로 격리
- **HippoRAG** — PPR 기반, 고밀도 엔티티 편향 위험

## Design Implication

LLM 비용은 그래프 구축에서 **"답변 생성에 대한 투자가 아니라 검색 품질에 대한 투자"**다. 따라서 오염 방어는 "어느 청크에 그래프 추출 비용을 쓸 가치가 있는가"의 자원 분배 문제가 된다 — KET-RAG의 통찰.

## See Also

- [graph-pollution](../concepts/graph-pollution.md)
- [graphrag](../concepts/graphrag.md)
