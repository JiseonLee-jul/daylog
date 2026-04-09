# Hybrid RAG ↔ KET-RAG

## Relationship

**KET-RAG는 Hybrid RAG의 세 번째 층위(Index Structure Hybrid)의 가장 명확한 구현 사례다.** 업계에서 "하이브리드 RAG"는 여러 층위로 쓰이지만, KET-RAG는 "동일 데이터를 서로 다른 자료구조로 중복 인덱싱"하는 Layer 3에 해당하며 논문 제목에도 "Multi-Granular Indexing Framework"로 명시되어 있다.

## Why KET-RAG is "Genuinely" Hybrid

다른 GraphRAG(Microsoft GraphRAG, LightRAG, RAPTOR)는 단일 그래프 또는 단일 트리 구조 안에서 계층화를 시도하지만, KET-RAG는 **풀 KG와 이분 그래프라는 구조가 다른 두 인덱스를 물리적으로 병존**시킨다. 이 비대칭 구조가 이기종 코퍼스의 정보 밀도 편차에 직접 대응한다.

## Implication for Heterogeneous Corpus

하이브리드는 선택이 아니라 필연이다. 정보 밀도가 본질적으로 다르다면 자료구조도 달라야 한다는 원칙을 KET-RAG가 극단까지 밀어붙인 결과. "모든 문서가 풀 KG의 표현력을 필요로 하진 않는다"는 통찰이 설계 철학의 핵심.

## See Also

- [hybrid-rag](../concepts/hybrid-rag.md)
- [ket-rag](../concepts/ket-rag.md)
