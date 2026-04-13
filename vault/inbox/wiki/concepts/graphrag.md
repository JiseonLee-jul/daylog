# GraphRAG

## Definition

LLM으로 원문에서 엔티티·관계 트리플을 추출해 지식 그래프를 구축하고, 쿼리 시 그래프 탐색으로 관련 청크를 찾아 LLM 답변을 생성하는 RAG 패러다임. 벡터 검색이 놓치는 멀티홉 추론과 구조적 연관성을 잡기 위해 도입됨.

## Common Architecture Pattern — Graph as Index, Text as Content

모든 GraphRAG 계열(Microsoft GraphRAG, LightRAG, RAPTOR, KET-RAG, HippoRAG)이 공유하는 공통 패턴:

1. **그래프 탐색** — 관련성 판단, 멀티홉 추론
2. **원본 청크 fetch** — 그래프 노드가 가리키는 원문
3. **LLM 답변 생성** — 원본 청크를 컨텍스트로

**핵심:** 그래프의 entity·relation은 LLM에 직접 들어가지 않는다. "어느 청크를 봐야 할지"를 결정하는 라우팅 정보로만 쓰인다.

## Why Graph as Index (Not Content)

- LLM은 자연어 원문을 트리플보다 잘 다룸 (뉘앙스·조건·부정 보존)
- 그래프 추출은 손실 압축 — 원문의 미묘한 정보가 트리플에 담기지 않음
- 감사 가능성 — 정확한 출처 추적 필요

## Variants

- **Microsoft GraphRAG** — Leiden 재귀 클러스터링 + 커뮤니티 요약, global vs local search
- **LightRAG** — 단일 그래프 + 쿼리 시점 dual-level (low-level entity / high-level concept)
- **RAPTOR** — 임베딩 기반 GMM 클러스터링 → LLM 요약 → 재귀 트리
- **KET-RAG** — 풀 KG + 이분 그래프 이중 인덱스, PageRank 기반 중요도 선별
- **HippoRAG** — 해마 영감, Personalized PageRank 기반 다중 홉

## Exception

Pure Symbolic KGQA (Wikidata QA 등)는 트리플만으로 답변한다. 구조화 데이터에 한정되며, 자연어 코퍼스(회의록·법령)에는 적용 불가. LLM 시대의 GraphRAG는 모두 "그래프 + 원본 청크" 하이브리드.

## Contrast: GraphRAG vs LLM Wiki

GraphRAG와 LLM Wiki는 모두 "원시 문서 위에 구조를 구축"하지만 접근이 다르다:
- **GraphRAG**: 엔티티·관계 그래프를 자동 구축 → 질의 시 그래프 탐색 → 원시 청크로 회귀. 그래프는 라우팅 인덱스.
- **LLM Wiki**: 마크다운 위키를 점진적 구축 → 지식이 위키에 컴파일되어 상주. 원시 문서로 회귀할 필요 감소.

GraphRAG는 대규모 코퍼스의 자동화된 구조화에 강하고, LLM Wiki는 인간 큐레이션이 개입하는 소·중규모 개인 지식 관리에 적합.

## Sources

- [260409_graph-pollution-heterogeneous-corpus](../summaries/260409_graph-pollution-heterogeneous-corpus.md)
- [gist_github_com_karpathy-llm-wiki](../summaries/gist_github_com_karpathy-llm-wiki.md)

## Related Concepts

- [lightrag](lightrag.md)
- [raptor](raptor.md)
- [ket-rag](ket-rag.md)
- [graph-pollution](graph-pollution.md)
- [hybrid-rag](hybrid-rag.md)
- [llm-wiki](llm-wiki.md) — 대비되는 지식 구조화 접근
- [knowledge-compilation](knowledge-compilation.md) — 사전 컴파일 vs 질의 시 검색
