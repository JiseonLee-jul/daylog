# RAPTOR

## Definition

**Recursive Abstractive Processing for Tree-Organized Retrieval** (Sarthi et al., ICLR 2024, arXiv:2401.18059). 청크를 의미 기반으로 클러스터링하고 LLM으로 요약하기를 재귀적으로 반복해 다층 요약 트리를 구축하는 RAG 인덱싱 방법.

## Indexing Process

1. 청크 분할 (보통 100토큰)
2. 청크 임베딩 (SBERT/ada 등)
3. **UMAP 차원 축소** — global + local 2단계, 고차원 거리 왜곡 방지
4. **GMM 클러스터링**
   - K-means 아닌 이유: soft assignment (한 청크가 여러 클러스터에 소속 가능)
   - BIC로 최적 클러스터 수 자동 결정
5. 클러스터별 LLM 요약 → Level 1 노드 생성
6. 재귀 (루트까지 또는 지정 깊이)
7. 모든 레벨 노드를 **평면 테이블로 벡터 DB 저장**: `node_id | level | text | embedding | children_ids | source_chunk_ids`

## Retrieval Modes

| 모드 | 특징 | 논문 권장 |
|---|---|---|
| Tree Traversal | 루트→하위 가지치기, 저비용, 상위 오판 시 손실 | ✗ |
| Collapsed Tree | 모든 레벨 노드 평면화 후 유사도 검색, 쿼리 적응적 | ✅ |

Collapsed Tree는 쿼리 추상도에 맞는 레벨의 노드를 자동 선택 — 추상 질의는 상위 요약, 구체 질의는 하위 원본.

## Core Insight — Tree as Generator, Not Traverser

**"RAPTOR의 계층 구조는 탐색 구조가 아니라, 다양한 추상도의 요약 노드를 생성하기 위한 장치다."**

Collapsed Tree 검색이 트리 순회를 안 한다고 해서 계층이 무의미한 것이 아니다. 트리 생성 과정(GMM 클러스터링 → LLM 요약 → 재귀)이 검색 공간에 추상도 다양성을 주입하는 것이 핵심. 계층 생성이 없다면 Collapsed Tree는 평범한 벡터 검색과 동일해진다.

## Pollution Mitigation Mechanism

- 요약 과정에서 LLM이 잡담·언쟁을 자연스럽게 생략 (물리적 노이즈 제거)
- 상위 레벨이 하위 노이즈를 희석
- 의미 기반 클러스터링으로 이기종 문서가 주제 단위로 재조직되어 자동 균형화

## Limitations

- 원본 문서 구조(법령 조문 계층) 완전 해체
- GMM 클러스터링 결과 불안정성
- 모든 레벨 요약에 LLM 비용 필요

## Use with Original Document Storage

RAPTOR 노드에 `source_chunk_ids` 포인터를 두면 원본 저장소와 함께 쓸 수 있다. LangChain MultiVectorRetriever 패턴과 동일 — 검색은 요약으로 빠르게, 응답은 원본으로 정확하게.

## Related Concepts

- [graphrag](graphrag.md)
- [ket-rag](ket-rag.md)
- [graph-pollution](graph-pollution.md)
- [chunking-strategy](chunking-strategy.md)
