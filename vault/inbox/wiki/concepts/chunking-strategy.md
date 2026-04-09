# Chunking Strategy

## Definition

RAG 인덱싱 파이프라인에서 문서를 검색 가능한 단위로 분할하는 전략. Graph Pollution 해결의 4개 축 중 축 A에 해당하며, "자르는 방식을 문서마다 다르게"가 핵심 철학.

## Four Execution Patterns

### A-1: Rule-based (사전 정의 룩업)

확장자·MIME·메타데이터로 파서 분기. `CHUNK_CONFIG[doc_type]` 방식. 결정론적, 디버깅 쉬움, 새 타입마다 파서 추가 필요.
- 대표: LlamaIndex `SimpleFileNodeParser`, Unstructured.io, LangChain `MarkdownHeaderTextSplitter`

### A-2: Structure Parser (구조 자동 감지)

문서 내부 헤딩·리스트·표·조문 마커를 파싱해 문서가 스스로 말하는 경계를 따름. 법령의 조·항·호, Markdown 헤딩, HTML DOM 등.
- 대표: Unstructured.io (가장 범용), LlamaIndex `HierarchicalNodeParser`

### A-3: Semantic (의미 경계)

문장 임베딩 유사도 급변점을 경계로 삼음. 문서 타입 불가지론.
- 대표: LangChain `SemanticChunker`, LlamaIndex `SemanticSplitterNodeParser`

### A-4: LLM Prompt-based

LLM에게 "독립 사실 단위로 분해하라" 등 지시. 문서당 LLM 호출 N회로 비용 폭증, 단 품질 최상.
- 대표: Dense X Retrieval (Proposition Indexing, EMNLP 2024), Anthropic Contextual Retrieval (2024, 실패율 67%↓ 보고)

## Other Notable Techniques

- **Late Chunking** (Jina AI, 2024) — 문서 전체를 long-context 모델로 먼저 인코딩 후 사후 풀링. 회의록 대명사 해소에 강함.
- **Parent-Child / Small-to-Big** — 작은 청크로 검색, 큰 청크로 응답. LangChain ParentDocumentRetriever.
- **RAPTOR** — 청크 분할은 단순히 100토큰, 그 위에 의미 기반 재귀 요약 트리 구축 (사실상 B축 영역).

## Pollution Mitigation Role

경계가 문서 의미 단위와 일치하면 LLM이 엔티티 추출 시 "맥락 없는 파편"을 다루지 않아 노이즈 엔티티 생성 확률 감소. 그러나 **경계만 정리할 뿐 내용 필터링은 아니므로**, 잡담 자체가 있는 문서에서는 효과 제한적. 축 C(Saliency)와 결합이 권장됨.

## Related Concepts

- [graph-pollution](graph-pollution.md)
- [heterogeneous-corpus](heterogeneous-corpus.md)
- [saliency-denoising](saliency-denoising.md)
- [raptor](raptor.md)
