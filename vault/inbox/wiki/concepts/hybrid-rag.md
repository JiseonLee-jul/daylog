# Hybrid RAG

## Definition

서로 다른 검색 방식·데이터 소스·인덱스 구조를 조합해 단일 RAG 시스템의 한계를 보완하는 접근. "하이브리드"는 업계에서 세 가지 층위로 쓰인다.

## Three Layers of Hybridization

### Layer 1: Retrieval Hybrid

BM25(희소 검색) + Dense Vector(밀집 검색) 병합. RRF(Reciprocal Rank Fusion) 또는 가중 점수 합산. "Hybrid search"라고도 부름. Elasticsearch + 벡터 DB 조합이 대표.

### Layer 2: Knowledge Source Hybrid

구조화 데이터(SQL, KG) + 비구조화 데이터(텍스트)를 함께 사용. Agent가 상황별로 선택·병합. HybGRAG, Microsoft "structured + unstructured RAG".

### Layer 3: Index Structure Hybrid

**동일 데이터를 다른 자료구조로 중복 인덱싱**하고 쿼리 성격에 따라 다른 구조를 탐색. KET-RAG의 풀 KG + 이분 그래프가 가장 명확한 사례. "Multi-granular hybrid indexing"이라고도 부름.

## Why Hybrid is Necessary for Heterogeneous Corpus

단일 인덱스·단일 전략으로는 Graph Pollution을 근본적으로 해결할 수 없다. 정보 밀도 편차가 본질적인 이상 자료구조도 달라야 하며, 효과적 해결책은 모두 본질적으로 하이브리드다. 하이브리드는 선택이 아니라 필연.

## Representative Implementations

- **HybGRAG** — BM25 + dense + Knowledge Graph 3중 검색
- **KET-RAG** — 풀 KG + 이분 그래프 (Layer 3)
- **LangChain MultiVectorRetriever** — 한 문서에 요약·가상질문·키워드 복수 벡터
- **LlamaIndex RouterQueryEngine** — 복수 인덱스 + LLM selector

## Related Concepts

- [ket-rag](ket-rag.md)
- [graphrag](graphrag.md)
- [rag-routing](rag-routing.md)
- [graph-pollution](graph-pollution.md)
