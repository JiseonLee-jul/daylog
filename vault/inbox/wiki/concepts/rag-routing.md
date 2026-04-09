# RAG Routing Architecture

## Definition

문서 종류별로 물리적으로 다른 인덱스에 저장하고 쿼리 시점에 적절한 인덱스로 라우팅하는 아키텍처 패턴. Graph Pollution 해결 4개 축 중 축 D에 해당하며, "아예 다른 방에 보관"이 철학.

## Execution Methods

### D-1: Metadata Hard Filter

수집 시 `doc_type`, `domain`, `date` 태그 부여. 쿼리는 SQL WHERE처럼 필터로 좁힘. 단일 인덱스 유지 가능.
- 대표: Azure AI Search field-level filtering, Pinecone/Weaviate metadata filter

### D-2: LLM Selector

각 인덱스에 자연어 설명 → LLM이 쿼리를 보고 인덱스 선택.
- 대표: LlamaIndex `RouterQueryEngine` (LLMSingleSelector)

### D-3: Query Decomposition

복합 쿼리를 LLM이 서브 쿼리로 분해 → 각 서브를 다른 인덱스에 분산 → 결과 합성.
- 대표: LlamaIndex `SubQuestionQueryEngine`
- 크로스-인덱스 질의 가능, LLM 호출 N+1회

### D-4: Agentic RAG

각 인덱스·DB·API를 Tool로 등록 → 에이전트가 런타임에 필요한 만큼 호출·재호출. Planner-Orchestrator-Executor 3단 계층(A-RAG).
- 가장 유연, 가장 비쌈, 디버깅 어려움

### D-5: Federated / Tiered

도메인·팀별 독립 인덱스, 상위 라우터가 페더레이션. 또는 핫/웜/콜드 계층. 엔터프라이즈 거버넌스·접근 제어 대응.
- 대표: Vectara(RAG Sprawl 해결), Glean, Azure AI Search

## Other Related Patterns

- **LangChain MultiVectorRetriever** — 한 문서에 요약·가상질문·키워드 복수 벡터
- **LangChain ParentDocumentRetriever** — small-to-retrieve, large-to-generate
- **Hybrid retrieval + per-type weighting** — BM25/dense/graph 병렬 + 타입별 가중치 RRF 병합

## Pollution Mitigation Role

오염이 한 인덱스 안에 격리되므로 다른 인덱스 품질에 영향 없음. 쿼리가 관련 인덱스로만 라우팅되면 무관한 오염과의 접촉 자체가 없음. **물리적 격리가 가장 강력한 방어**.

## Trade-offs

- 크로스-타입 질의("법령에 근거해 회의 결정이 유효한가?")에서 여러 인덱스 조합 부담
- 인덱스 N배 저장 공간
- 운영 복잡도 증가 (스키마 거버넌스)
- 1M+ 토큰 context 확산으로 일부 복잡 라우팅의 필요성 재검토 중

## Related Concepts

- [graph-pollution](graph-pollution.md)
- [hybrid-rag](hybrid-rag.md)
- [heterogeneous-corpus](heterogeneous-corpus.md)
- [lightrag](lightrag.md)
